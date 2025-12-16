# Implementation Plan v2: Manifest-Based Workflow Orchestration

## Evolution from v1

**Key insight from Anthropic's context engineering guidance:** Rather than embedding the manifest in CLAUDE.md (constant token tax), use **just-in-time retrieval** - store manifest externally, load only when needed.

**v1 approach:** Full manifest in CLAUDE.md (~150 lines on every turn)
**v2 approach:** One-liner reference in CLAUDE.md, manifest in `.claude/workflow-manifest.yaml`, loaded on-demand by orchestrator

---

## Goal

Convert 9 workflow skills to manifest-based orchestration using Anthropic's progressive disclosure pattern:
- Skills become "pure capabilities" (no workflow awareness)
- Manifest lives in `.claude/workflow-manifest.yaml` (not CLAUDE.md)
- CLAUDE.md contains only a lightweight pointer
- Orchestrator (`workflow-status`) loads manifest just-in-time
- Zero token overhead on non-workflow interactions

---

## Context Efficiency Principles Applied

| Anthropic Principle | Our Implementation |
|--------------------|-------------------|
| Just-in-time retrieval | Manifest file loaded only when orchestrator runs |
| Progressive disclosure | Skill names in manifest → full SKILL.md loaded only on invocation |
| Lightweight identifiers | CLAUDE.md has one-liner, not full manifest |
| Structured note-taking | `.docs/` handoffs persist decisions outside context |
| Sub-agent architecture | Each skill is focused capability; orchestrator coordinates |

---

## Phase A: Foundation

### A1. Create External Manifest File
**File:** `.claude/workflow-manifest.yaml`

Create the manifest as a standalone YAML file:

```yaml
# Workflow Manifest - Loaded on-demand by workflow-status orchestrator
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
    next: tech-stack-advisor
    entry_point: true

  solution-architect:
    phase: architecture
    description: "Resolve architectural ambiguity for complex projects"
    optional: true
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
    next:
      default: deploy-guide
      conditions:
        - if: "deployment_strategy.hosting.type == 'localhost'"
          then: null
          termination: true
    optional_next:
      - test-orchestrator

  test-orchestrator:
    phase: quality
    description: "Set up testing infrastructure and strategy"
    optional: true
    standalone: true
    requires:
      - ".docs/tech-stack-decision.json"
    outputs:
      - ".docs/test-strategy.md"
    next: deploy-guide

  deploy-guide:
    phase: release
    description: "Guide through actual deployment steps"
    requires:
      - ".docs/deployment-strategy.json"
    outputs:
      - ".docs/deployment-log.json"
    next: null
    termination: true
    optional_next:
      - ci-cd-implement

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

termination_points:
  - after: project-spinup
    label: "Localhost/Native - Development Ready"
    condition: "deployment_strategy.hosting.type == 'localhost'"
  - after: deploy-guide
    label: "Manual Deployment Complete"
    default: true
  - after: ci-cd-implement
    label: "Full CI/CD Automation"

decision_gates:
  - after: project-brief-writer
    ask: "Does this project need architecture review?"
    options:
      yes: solution-architect
      no: tech-stack-advisor
  - after: project-spinup
    condition: "deployment_strategy.hosting.type != 'localhost'"
    ask: "Project foundation ready. Deploy now or develop first?"
    options:
      deploy: deploy-guide
      develop: null
  - after: deploy-guide
    ask: "Set up CI/CD automation?"
    options:
      yes: ci-cd-implement
      no: null

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
```

### A2. Add Lightweight Pointer to CLAUDE.md Template
**For new projects using this workflow:**

```markdown
## Workflow

This project uses the skill-foundry workflow system.

**Manifest:** `.claude/workflow-manifest.yaml`
**Status:** Invoke `workflow-status` skill to check progress
**Handoffs:** `.docs/` directory
```

That's it. ~5 lines instead of ~150.

### A3. Transform workflow-status into Orchestrator
**File:** `workflow-status/SKILL.md`

Modify to:
1. Check for `.claude/workflow-manifest.yaml` in project directory
2. If found: Parse YAML and use for phase mapping, routing, termination detection
3. If not found: Fall back to hardcoded defaults (backward compatibility)
4. Keep existing `.docs/` scanning logic (works well)
5. Present status using manifest's phase structure

Key additions:
- `<manifest-loading>` section with just-in-time file read
- Manifest-driven `<phase-mapping>` replacing hardcoded table
- Condition evaluation for `next` routing
- Backward compatibility when no manifest exists

---

## Phase B: Pilot Skill Refactoring

### B1. Refactor project-brief-writer
**File:** `project-brief-writer/SKILL.md`

**Remove:**
- Line 348: `"handoff_to": [...]` from JSON schema
- Lines 377-399: Phase 7 "Workflow Status" display
- Lines 531-557: `<integration-notes>` section
- Line 422: "Guide user to next skill" guardrail

**Keep:** All core capability (question gathering, brief generation, mode selection)

**Test checkpoint:** Verify skill works standalone and orchestrator correctly identifies it as complete when `.docs/brief.json` exists.

---

## Phase C: Planning Skills

### C1. Refactor solution-architect
**File:** `solution-architect/SKILL.md`

Remove: Workflow position, prerequisite Phase 0, `handoff_to`, "LOCKED" language, integration notes

### C2. Refactor tech-stack-advisor
**File:** `tech-stack-advisor/SKILL.md`

Remove: Phase 0 prerequisites, `handoff_to`, workflow-status section, integration notes, "LOCKED" references

**Note:** Already uses progressive disclosure (DECISION-FRAMEWORKS.md, PLATFORM-CONSIDERATIONS.md as external refs)

### C3. Refactor deployment-advisor
**File:** `deployment-advisor/SKILL.md`

Remove: Phase 0/0.5 prerequisites, `termination_point`/`handoff_to` in JSON, workflow sections

**Note:** Already uses progressive disclosure (HOSTING-TEMPLATES.md, RESOURCES/ as external refs)

---

## Phase D: Implementation Skills

### D1. Refactor project-spinup (COMPLEX)
**File:** `project-spinup/SKILL.md`

Remove: Phase 1 handoff loading, Phase 1.5 native routing, `workflow_status`/`handoff_to`, workflow sections

**Key change:** Native platform detection stays as capability, but routing decision moves to manifest condition

### D2. Refactor test-orchestrator
**File:** `test-orchestrator/SKILL.md`

Remove: Phase 0/1 handoff loading, next step guidance, workflow sections

---

## Phase E: Release Skills

### E1. Refactor deploy-guide
**File:** `deploy-guide/SKILL.md`

Remove: Phase 0/1 prerequisites, `workflow_status`/`handoff_to`, workflow sections

**Note:** Already uses progressive disclosure (REFERENCES/ directory)

### E2. Refactor ci-cd-implement
**File:** `ci-cd-implement/SKILL.md`

Remove: Phase 0/1 prerequisites, "LOCKED" language, workflow sections

---

## Phase F: Documentation & Templates

### F1. Update HANDOFF-DOCUMENTATION.md
**File:** `workflow-status/references/HANDOFF-DOCUMENTATION.md`

- Document manifest-based pattern
- Explain just-in-time loading
- Remove skill-to-skill references
- Keep handoff document specifications

### F2. Create Project Template
**New:** Template for new projects using this workflow

```
new-project/
├── .claude/
│   └── workflow-manifest.yaml  # Copy from template
├── .docs/                      # Created by first skill
└── CLAUDE.md                   # With workflow pointer section
```

### F3. Update skill READMEs
Remove workflow integration sections, document standalone capability.

---

## Progressive Disclosure Patterns (Already in Use)

| Skill | External References |
|-------|-------------------|
| tech-stack-advisor | DECISION-FRAMEWORKS.md, PLATFORM-CONSIDERATIONS.md |
| deployment-advisor | HOSTING-TEMPLATES.md, RESOURCES/*.xml |
| deploy-guide | REFERENCES/*.xml |
| workflow-status | references/HANDOFF-DOCUMENTATION.md |

**Recommendation:** Extend this pattern to remaining skills where appropriate. Move examples and edge-case handling to `REFERENCES/` subdirectories.

---

## Execution Sequence

```
A1 → A2 → A3 → B1 → [checkpoint/test] → C1-C3 → D1-D2 → E1-E2 → F1-F3
```

**Critical checkpoint after B1:** Verify:
- Orchestrator loads manifest correctly
- Skill completion detected from `.docs/brief.json`
- Next step recommendation works
- Backward compatibility (no manifest = fallback behavior)

---

## Key Files

| File | Change Type |
|------|-------------|
| `.claude/workflow-manifest.yaml` | **NEW** - External manifest file |
| `workflow-status/SKILL.md` | Transform to manifest-driven orchestrator |
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

## Token Efficiency Comparison

| Scenario | v1 (manifest in CLAUDE.md) | v2 (external manifest) |
|----------|---------------------------|------------------------|
| Non-workflow turn | ~150 tokens (manifest) | ~5 tokens (pointer) |
| workflow-status invocation | ~150 tokens | ~150 tokens (loaded JIT) |
| Skill invocation | ~150 tokens | ~5 tokens |

**Net savings:** ~145 tokens per non-workflow turn. For a typical session with 50 turns where only 5 involve workflow status, that's ~6,500 tokens saved.

---

## Risk Mitigation

1. **Backward compatibility:** Orchestrator falls back if no manifest found
2. **Flexible entry preserved:** Skills gather missing info conversationally
3. **Manifest sync:** Single file, single source of truth
4. **Progressive disclosure:** External refs pattern already proven in 4 skills

---

## References

- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) - Anthropic
- [Progressive Disclosure MCP](https://www.geeky-gadgets.com/progressive-disclosure-mcp/) - Context optimization
- [Claude Memory Deep Dive](https://skywork.ai/blog/claude-memory-a-deep-dive-into-anthropics-persistent-context-solution/) - Fading memory problem
