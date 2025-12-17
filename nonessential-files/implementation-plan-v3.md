# Implementation Plan v3: Manifest-Based Workflow Orchestration

## Evolution from v2

**v2 validated:** Multi-model analysis (Gemini, GPT-5.1) confirmed the core design aligns with Anthropic's context engineering principles (avg score 4.3/5).

**v3 additions:** This revision addresses 5 critical refinements identified during review:

1. **Transition semantics** — Clarify `next` vs `decision_gates` relationship
2. **Output contract** — Define `workflow-status` response format for progressive disclosure
3. **Structured conditions** — Replace free-form condition strings with testable predicates
4. **Schema validation** — Add validation rules for manifest integrity
5. **`optional` semantics** — Clarify as metadata hint, not control flow

---

## Goal

Convert 9 workflow skills to manifest-based orchestration using Anthropic's progressive disclosure pattern:
- Skills become "pure capabilities" (no workflow awareness)
- Manifest lives in `.claude/workflow-manifest.yaml` (not CLAUDE.md)
- CLAUDE.md contains only a lightweight pointer
- Orchestrator (`workflow-status`) loads manifest just-in-time
- Zero token overhead on non-workflow interactions

---

## Design Decisions (v3 Refinements)

### Decision 1: Transition Semantics

**Problem:** v2 had two overlapping ways to express transitions (`next` on skills and `decision_gates`), creating ambiguity.

**Resolution:**

| Construct | Purpose | Behavior |
|-----------|---------|----------|
| `skills.<skill>.next` | Default fallthrough | Used when no decision gate applies |
| `decision_gates` | Conditional override | When present for a skill, **overrides** that skill's `next` |

**Rules:**
- If a `decision_gate` exists with `after: X`, skill X's `next` is ignored
- Validation warns if both `next` and a gate exist for the same skill
- `next: null` means workflow pauses for user direction (not termination)
- `termination: true` on a skill means workflow ends after that skill

**Example:**
```yaml
skills:
  project-brief-writer:
    next: tech-stack-advisor  # DEFAULT: ignored because gate exists below

decision_gates:
  - after: project-brief-writer  # OVERRIDE: this controls routing
    ask: "Does this project need architecture review?"
    routes:
      yes: solution-architect
      no: tech-stack-advisor
```

---

### Decision 2: `workflow-status` Output Contract

**Problem:** If orchestrator dumps the full manifest, progressive disclosure fails.

**Resolution:** Define three output modes:

#### Default Mode (minimal context)
```
Workflow: project-development v1.0.0

Current Phase: Architecture (2 of 6)

Completed:
  [x] project-brief-writer -> .docs/brief.json

Next Recommended: tech-stack-advisor
  Reason: Brief complete, ready for stack recommendation
  Requires: .docs/brief.json (exists)

Decision Pending: None
```

#### `--full` Mode (complete view)
Shows all phases, all skills, all decision points, full dependency graph.

#### `--trace` Mode (debugging)
Machine-readable JSON showing:
- Files checked and their status
- Gates evaluated and results
- Transition logic applied

**Implementation:** Add `<output-modes>` section to `workflow-status/SKILL.md` specifying these formats.

---

### Decision 3: Structured Condition Predicates

**Problem:** Free-form strings like `"deployment_strategy.hosting.type == 'localhost'"` require a custom parser and are error-prone.

**Resolution:** Replace with structured predicates:

```yaml
# OLD (v2) - ambiguous, hard to validate
condition: "deployment_strategy.hosting.type == 'localhost'"

# NEW (v3) - structured, testable
condition:
  source: ".docs/deployment-strategy.json"
  field: "hosting.type"
  op: "equals"
  value: "localhost"
```

**Supported operators:**

| Operator | Description | Example |
|----------|-------------|---------|
| `equals` | Exact match | `{ op: equals, value: "localhost" }` |
| `not_equals` | Negation | `{ op: not_equals, value: "production" }` |
| `in` | Value in list | `{ op: in, value: ["vps", "cloud"] }` |
| `exists` | Field/file exists | `{ op: exists }` (no value needed) |
| `not_exists` | Field/file missing | `{ op: not_exists }` |

**File existence shorthand:**
```yaml
# Check if file exists (no JSON parsing needed)
condition:
  file: ".docs/architecture-context.json"
  op: "exists"
```

---

### Decision 4: Schema Validation Rules

**Problem:** Invalid manifests cause runtime failures.

**Resolution:** `workflow-status` validates manifest on load:

#### Required Validations

| Check | Error if... |
|-------|-------------|
| Structure | Missing required fields (`workflow`, `version`, `skills`) |
| Skill references | `next` or gate references non-existent skill |
| Phase references | Skill's `phase` not defined in `phases` |
| Acyclicity | Graph contains cycles (A -> B -> A) |
| Entry point | No skill marked `entry_point: true` |
| Termination | No reachable termination point |

#### Warning Validations

| Check | Warn if... |
|-------|------------|
| Redundant routing | Skill has both `next` and a `decision_gate` |
| Orphan skills | Skill not reachable from entry point |
| Missing outputs | Skill declares no `outputs` |

**Behavior on validation failure:**
- Errors: Halt and display actionable message
- Warnings: Display but continue
- Fallback: If manifest missing/unreadable, use hardcoded defaults (backward compatibility)

---

### Decision 5: `optional` Semantics

**Problem:** `optional: true` overlaps with decision gates and variants.

**Resolution:** `optional` is **metadata only**, not control flow:

```yaml
skills:
  solution-architect:
    optional: true  # METADATA: hints to UI/docs that this can be skipped
    # Actual skip logic is in decision_gates or variants
```

**Usage:**
- UI can display "(optional)" badge
- Documentation generation uses it
- Does NOT affect routing — gates and variants control that
- If you want a skill skippable, add a decision gate

---

## Revised Manifest Schema (v3)

```yaml
# Workflow Manifest v3 - Loaded on-demand by workflow-status orchestrator
schema_version: "3.0"  # Manifest format version (separate from workflow version)
workflow: project-development
version: 1.0.0
handoff_directory: ".docs/"

phases:
  discovery:
    name: "Discovery"
    description: "Understanding the problem space"
  architecture:
    name: "Architecture"
    description: "Technical decisions and design"
  planning:
    name: "Planning"
    description: "Deployment and infrastructure strategy"
  implementation:
    name: "Implementation"
    description: "Project generation and setup"
  quality:
    name: "Quality"
    description: "Testing infrastructure"
  release:
    name: "Release"
    description: "Deployment and automation"

skills:
  project-brief-writer:
    phase: discovery
    description: "Transform rough ideas into structured project briefs"
    requires: []
    outputs:
      - ".docs/brief.json"
      - ".docs/PROJECT-MODE.md"
    next: tech-stack-advisor  # Default (overridden by gate below)
    entry_point: true

  solution-architect:
    phase: architecture
    description: "Resolve architectural ambiguity for complex projects"
    optional: true  # Metadata hint only
    requires:
      - ".docs/brief.json"
    outputs:
      - ".docs/architecture-context.json"
    next: tech-stack-advisor

  tech-stack-advisor:
    phase: architecture
    description: "Recommend technology stack with rationale"
    requires:
      - ".docs/brief.json"
    optional_inputs:
      - ".docs/architecture-context.json"
    outputs:
      - ".docs/tech-stack-decision.json"
    next: deployment-advisor

  deployment-advisor:
    phase: planning
    description: "Recommend hosting and deployment strategy"
    requires:
      - ".docs/tech-stack-decision.json"
    outputs:
      - ".docs/deployment-strategy.json"
    next: project-spinup

  project-spinup:
    phase: implementation
    description: "Generate project foundation with CLAUDE.md and structure"
    requires:
      - ".docs/tech-stack-decision.json"
      - ".docs/deployment-strategy.json"
    outputs:
      - ".docs/project-foundation.json"
      - "CLAUDE.md"
    next: deploy-guide  # Default (overridden by gate below for localhost)

  test-orchestrator:
    phase: quality
    description: "Set up testing infrastructure and strategy"
    optional: true
    standalone: true  # Can be invoked anytime after tech-stack
    requires:
      - ".docs/tech-stack-decision.json"
    outputs:
      - ".docs/test-strategy.md"
    next: null  # Returns to previous flow

  deploy-guide:
    phase: release
    description: "Guide through actual deployment steps"
    requires:
      - ".docs/deployment-strategy.json"
    outputs:
      - ".docs/deployment-log.json"
    next: null
    termination: true

  ci-cd-implement:
    phase: release
    description: "Implement CI/CD pipelines"
    optional: true
    requires:
      - ".docs/deployment-strategy.json"
    outputs:
      - ".github/workflows/ci.yml"
      - ".github/workflows/deploy.yml"
      - "CICD-SECRETS.md"
    next: null
    termination: true

# Decision gates - OVERRIDE skill's `next` when present
decision_gates:
  - after: project-brief-writer
    ask: "Does this project need architecture review?"
    routes:
      yes: solution-architect
      no: tech-stack-advisor

  - after: project-spinup
    # Only show this gate if NOT localhost
    condition:
      source: ".docs/deployment-strategy.json"
      field: "hosting.type"
      op: "not_equals"
      value: "localhost"
    ask: "Project foundation ready. Deploy now or develop first?"
    routes:
      deploy: deploy-guide
      develop: null  # Pause workflow

  - after: project-spinup
    # Localhost projects terminate here
    condition:
      source: ".docs/deployment-strategy.json"
      field: "hosting.type"
      op: "equals"
      value: "localhost"
    auto_terminate: true
    label: "Localhost/Native - Development Ready"

  - after: deploy-guide
    ask: "Set up CI/CD automation?"
    routes:
      yes: ci-cd-implement
      no: null  # End workflow

# Named workflow variants
variants:
  minimal:
    description: "Fast path for simple localhost projects"
    skills: [project-brief-writer, tech-stack-advisor, deployment-advisor, project-spinup]
  standard:
    description: "Standard path through deployment"
    skills: [project-brief-writer, tech-stack-advisor, deployment-advisor, project-spinup, deploy-guide]
  full:
    description: "Complete path with all optional phases"
    skills: all

# Explicit termination points for documentation/validation
termination_points:
  - after: project-spinup
    label: "Localhost/Native - Development Ready"
    condition:
      source: ".docs/deployment-strategy.json"
      field: "hosting.type"
      op: "equals"
      value: "localhost"
  - after: deploy-guide
    label: "Manual Deployment Complete"
    default: true
  - after: ci-cd-implement
    label: "Full CI/CD Automation"
```

---

## Phase A: Foundation (Updated)

### A0. Document Design Decisions (NEW)
**Output:** This plan serves as the design doc. Key decisions documented above.

### A1. Create External Manifest File
**File:** `.claude/workflow-manifest.yaml`

Create manifest using v3 schema above with:
- `schema_version` field
- Structured condition predicates
- Clear `next` vs gate separation
- `optional` as metadata only

### A2. Add Lightweight Pointer to CLAUDE.md Template
**For new projects using this workflow:**

```markdown
## Workflow

This project uses the skill-foundry workflow system.

**Manifest:** `.claude/workflow-manifest.yaml`
**Status:** Invoke `workflow-status` skill to check progress
**Handoffs:** `.docs/` directory
```

### A3. Transform workflow-status into Orchestrator
**File:** `workflow-status/SKILL.md`

Implement:
1. **Manifest loading** — JIT read of `.claude/workflow-manifest.yaml`
2. **Schema validation** — Check structure, references, cycles per Decision 4
3. **Output modes** — Default (minimal), `--full`, `--trace` per Decision 2
4. **Condition evaluation** — Parse structured predicates per Decision 3
5. **Transition logic** — Gates override `next` per Decision 1
6. **Backward compatibility** — Fallback to hardcoded if no manifest

Key sections to add/modify:
- `<manifest-loading>` — JIT file read with validation
- `<output-modes>` — Three response formats
- `<condition-evaluation>` — Predicate parser
- `<transition-resolution>` — Gate-overrides-next logic

---

## Phase B-F: Unchanged from v2

Skill refactoring phases remain the same — remove orchestration logic from each skill. See v2 plan for details.

---

## Execution Sequence (Updated)

```
A0 (design doc) → A1 (manifest) → A2 (pointer) → A3 (orchestrator) → B1 → [checkpoint] → C1-C3 → D1-D2 → E1-E2 → F1-F3
```

**Critical checkpoint after A3:** Verify:
- Manifest validation catches errors (test with malformed YAML)
- Default output mode shows minimal context
- `--full` mode shows complete workflow
- Condition predicates evaluate correctly
- Gate-overrides-next logic works
- Backward compatibility (no manifest = fallback)

**Critical checkpoint after B1:** Verify:
- Skill works standalone
- Orchestrator detects completion from `.docs/brief.json`
- Next step recommendation correct

---

## Validation Test Cases

### Manifest Validation Tests

| Test | Input | Expected |
|------|-------|----------|
| Valid manifest | Well-formed YAML | Loads successfully |
| Missing `workflow` | YAML without workflow field | Error: "Missing required field: workflow" |
| Invalid skill ref | `next: nonexistent-skill` | Error: "Skill 'nonexistent-skill' not found" |
| Cycle detection | A -> B -> A | Error: "Cycle detected: A -> B -> A" |
| Redundant routing | Skill has `next` AND gate | Warning: "Gate overrides next for skill X" |

### Condition Predicate Tests

| Test | Condition | File Content | Expected |
|------|-----------|--------------|----------|
| Equals match | `{ field: "type", op: "equals", value: "localhost" }` | `{ "type": "localhost" }` | true |
| Equals no match | `{ field: "type", op: "equals", value: "localhost" }` | `{ "type": "vps" }` | false |
| File exists | `{ file: ".docs/brief.json", op: "exists" }` | File present | true |
| File missing | `{ file: ".docs/brief.json", op: "exists" }` | File absent | false |
| In list | `{ field: "type", op: "in", value: ["vps", "cloud"] }` | `{ "type": "vps" }` | true |
| Nested field | `{ field: "hosting.type", op: "equals", value: "localhost" }` | `{ "hosting": { "type": "localhost" } }` | true |

### Output Mode Tests

| Mode | Expected Content |
|------|------------------|
| Default | Current phase, completed steps, next skill, reason |
| `--full` | All phases, all skills, all gates, dependency graph |
| `--trace` | JSON with files checked, gates evaluated, transitions applied |

---

## Key Files (Updated)

| File | Change Type |
|------|-------------|
| `.claude/workflow-manifest.yaml` | **NEW** - v3 schema with structured conditions |
| `workflow-status/SKILL.md` | Transform to orchestrator with validation + output modes |
| `project-brief-writer/SKILL.md` | Remove orchestration |
| `solution-architect/SKILL.md` | Remove orchestration |
| `tech-stack-advisor/SKILL.md` | Remove orchestration |
| `deployment-advisor/SKILL.md` | Remove orchestration |
| `project-spinup/SKILL.md` | Remove orchestration (complex) |
| `test-orchestrator/SKILL.md` | Remove orchestration |
| `deploy-guide/SKILL.md` | Remove orchestration |
| `ci-cd-implement/SKILL.md` | Remove orchestration |
| `workflow-status/references/HANDOFF-DOCUMENTATION.md` | Update docs |

---

## Risk Mitigation (Updated)

| Risk | Mitigation |
|------|------------|
| Transition ambiguity | **Resolved:** Gates override `next`, documented and validated |
| `workflow-status` bloat | Output modes keep default minimal; split later if needed |
| Condition complexity | **Resolved:** Structured predicates, limited operators |
| Schema drift | Validation on load catches mismatches |
| Backward compatibility | Fallback to hardcoded if no manifest |
| File-state fragility | Deferred to v4; design allows adding state file later |

---

## References

- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) - Anthropic
- [Claude Code: Best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices) - Anthropic
- [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) - Anthropic
- Multi-model review: `review.md` (Gemini + GPT-5.1 analysis)
