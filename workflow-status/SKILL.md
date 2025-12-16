---
name: workflow-status
description: "Display project workflow progress by reading handoff documents in .docs/ directory. This skill should be used when users want to check their workflow status, see what phase they're in, or when other workflow skills need to verify prerequisites. Provides reusable prerequisite-checking templates for integration with other workflow skills."
---

# workflow-status

<purpose>
Orchestrator skill that loads workflow manifest just-in-time and presents workflow progress. Reads handoff documents from .docs/, evaluates conditions, and recommends next steps based on the manifest's transition logic.

Key responsibilities:
1. Load and validate workflow manifest (`.claude/workflow-manifest.yaml`)
2. Scan `.docs/` for handoff documents to determine completion state
3. Evaluate decision gates and conditions
4. Present status in the appropriate output mode (default/full/trace)
5. Recommend next skill based on manifest routing
</purpose>

<output>
- Formatted status display showing workflow progress
- Current phase identification
- Completed skills with their outputs
- Next skill recommendation with rationale
- Decision gate prompts when applicable
- Reusable templates for other skills' prerequisite checks
</output>

---

<manifest-loading>

<location>
Primary: `.claude/workflow-manifest.yaml` (project-level)
Fallback: Use hardcoded defaults if manifest not found (backward compatibility)
</location>

<loading-process>
1. Attempt to read `.claude/workflow-manifest.yaml`
2. If found:
   a. Parse YAML content
   b. Validate schema (see schema-validation section)
   c. Extract workflow configuration
3. If not found or invalid:
   a. Log: "Manifest not found - using built-in workflow definition"
   b. Use hardcoded fallback (preserves backward compatibility)
4. Cache manifest for duration of skill invocation
</loading-process>

<manifest-structure>
Expected top-level fields:
```yaml
schema_version: "3.0"        # Required
workflow: project-development # Required
version: 1.0.0               # Required
handoff_directory: ".docs/"  # Required

phases: {}                   # Required - phase definitions
skills: {}                   # Required - skill definitions
decision_gates: []           # Optional - conditional routing
variants: {}                 # Optional - named workflow paths
termination_points: []       # Optional - explicit endpoints
```
</manifest-structure>

<fallback-defaults>
When manifest is missing, use this hardcoded workflow:

Entry: project-brief-writer
Flow: project-brief-writer → tech-stack-advisor → deployment-advisor → project-spinup
Optional: solution-architect (after brief), test-orchestrator (after tech-stack)
Termination: project-spinup (localhost), deploy-guide (manual), ci-cd-implement (full)
</fallback-defaults>

</manifest-loading>

---

<schema-validation>

<validation-on-load>
When manifest is loaded, validate before use:
</validation-on-load>

<required-validations>
These cause errors (halt with message):

| Check | Error Condition | Message |
|-------|-----------------|---------|
| Structure | Missing `workflow`, `version`, or `skills` | "Missing required field: {field}" |
| Skill references | `next` references non-existent skill | "Skill '{name}' references unknown skill '{ref}'" |
| Gate references | Gate route references non-existent skill | "Decision gate references unknown skill '{ref}'" |
| Phase references | Skill's `phase` not in `phases` | "Skill '{name}' references unknown phase '{phase}'" |
| Entry point | No skill has `entry_point: true` | "No entry point defined" |
| Termination | No skill has `termination: true` or `next: null` | "No termination point reachable" |
| Cycles | Graph contains cycles | "Cycle detected: {path}" |
</required-validations>

<warning-validations>
These show warnings but continue:

| Check | Warning Condition | Message |
|-------|-------------------|---------|
| Redundant routing | Skill has both `next` AND a decision_gate | "Gate overrides 'next' for skill '{name}'" |
| Orphan skills | Skill not reachable from entry point | "Skill '{name}' is not reachable from entry point" |
| Missing outputs | Skill declares no `outputs` | "Skill '{name}' has no outputs defined" |
</warning-validations>

<cycle-detection>
Algorithm: Depth-first traversal from entry point
Track: visited set, current path stack
Detect: If skill already in current path → cycle found
Report: Full cycle path for debugging
</cycle-detection>

<validation-behavior>
- Errors: Halt and display actionable message, do not proceed
- Warnings: Display but continue with workflow
- Success: Proceed to workflow analysis
</validation-behavior>

</schema-validation>

---

<output-modes>

<mode-selection>
Determine output mode from invocation:
- Default: No flags (minimal context, ~5-10 lines)
- Full: `--full` flag (complete workflow view)
- Trace: `--trace` flag (machine-readable JSON for debugging)
</mode-selection>

<default-mode>
Minimal context optimized for token efficiency:

```
Workflow: {workflow} v{version}

Current Phase: {phase_name} ({phase_number} of {total_phases})

Completed:
  [x] {skill_name} → {output_file}
  [x] {skill_name} → {output_file}

Next Recommended: {next_skill}
  Reason: {why this skill is next}
  Requires: {dependency} ({exists|missing})

{If decision pending:}
Decision Pending: {gate_question}
  Options: {route1}, {route2}
```
</default-mode>

<full-mode>
Complete workflow visualization:

```
Workflow: {workflow} v{version}
Schema: {schema_version}

═══════════════════════════════════════
PHASES
═══════════════════════════════════════

{For each phase:}
## {phase_name}
{phase_description}

Skills:
  {For each skill in phase:}
  - {skill_name}: {status} {if complete: → output_file}
    {description}

═══════════════════════════════════════
DECISION GATES
═══════════════════════════════════════

{For each gate:}
After {skill}: {question}
  Routes: {route1} → {target1}, {route2} → {target2}
  {If condition:} Condition: {condition_summary}

═══════════════════════════════════════
TERMINATION POINTS
═══════════════════════════════════════

{For each termination:}
- After {skill}: {label}
  {If condition:} When: {condition_summary}

═══════════════════════════════════════
CURRENT STATUS
═══════════════════════════════════════

{Same as default mode status section}
```
</full-mode>

<trace-mode>
Machine-readable JSON for debugging:

```json
{
  "manifest": {
    "workflow": "{workflow}",
    "version": "{version}",
    "schema_version": "{schema_version}",
    "loaded_from": "{path or 'fallback'}"
  },
  "validation": {
    "errors": [],
    "warnings": ["{warning1}", "{warning2}"]
  },
  "files_checked": [
    {"path": ".docs/brief.json", "exists": true, "valid_json": true},
    {"path": ".docs/tech-stack-decision.json", "exists": false}
  ],
  "skills_status": {
    "project-brief-writer": {"status": "completed", "output": ".docs/brief.json"},
    "tech-stack-advisor": {"status": "pending", "blocked_by": null}
  },
  "gates_evaluated": [
    {
      "after": "project-brief-writer",
      "condition": null,
      "result": "awaiting_user_input",
      "question": "Does this project need architecture review?"
    }
  ],
  "transition": {
    "current_skill": "project-brief-writer",
    "next_skill": null,
    "reason": "decision_gate_pending",
    "gate": "architecture_review"
  }
}
```
</trace-mode>

</output-modes>

---

<condition-evaluation>

<predicate-structure>
Conditions use structured predicates (not free-form strings):

```yaml
condition:
  source: ".docs/deployment-strategy.json"  # File to read
  field: "hosting.type"                      # JSON path (dot notation)
  op: "equals"                               # Operator
  value: "localhost"                         # Expected value
```

Or file existence shorthand:
```yaml
condition:
  file: ".docs/architecture-context.json"
  op: "exists"
```
</predicate-structure>

<supported-operators>
| Operator | Description | Value Type | Example |
|----------|-------------|------------|---------|
| `equals` | Exact match | string/number/boolean | `{ op: equals, value: "localhost" }` |
| `not_equals` | Negation | string/number/boolean | `{ op: not_equals, value: "production" }` |
| `in` | Value in list | array | `{ op: in, value: ["vps", "cloud"] }` |
| `exists` | Field/file exists | (none) | `{ op: exists }` |
| `not_exists` | Field/file missing | (none) | `{ op: not_exists }` |
</supported-operators>

<evaluation-process>
1. Check if condition has `file` key (file existence check)
   - If yes: Check if file exists, return boolean
2. Check if condition has `source` key (field value check)
   - Read source file as JSON
   - Navigate to field using dot notation (e.g., "hosting.type" → obj.hosting.type)
   - Apply operator to field value
3. Return evaluation result (true/false)
4. On error (file missing, invalid JSON, field not found):
   - For `exists`/`not_exists`: Return appropriate boolean
   - For value operators: Return false (condition not met)
</evaluation-process>

<nested-field-access>
For fields like "hosting.type":
1. Split by "."
2. Traverse object: obj["hosting"]["type"]
3. Handle missing intermediate keys gracefully (return undefined)
</nested-field-access>

</condition-evaluation>

---

<transition-resolution>

<core-rule>
Decision gates OVERRIDE a skill's `next` field when present.
</core-rule>

<resolution-algorithm>
Given a completed skill, determine next step:

1. Find all decision_gates where `after` matches skill name
2. For each gate (in order):
   a. If gate has `condition`: evaluate it
      - If condition is false: skip this gate
   b. If gate has `auto_terminate: true`:
      - Return: workflow terminates here
   c. If gate has `ask`:
      - Return: decision pending, present question to user
   d. If gate has `routes` without `ask`:
      - Use first route (or evaluate route conditions if present)
3. If no gate applies:
   a. Use skill's `next` field
   b. If `next: null`: workflow pauses for user direction
   c. If `termination: true` on skill: workflow complete
4. Return resolved next skill (or termination/pause state)
</resolution-algorithm>

<gate-priority>
When multiple gates exist for same skill:
- Gates are evaluated in order defined in manifest
- First matching gate (condition passes) wins
- Allows priority-based conditional routing
</gate-priority>

<user-decision-handling>
When a gate requires user input (`ask` field):
1. Present question to user
2. Show route options
3. Wait for user selection
4. Route to selected skill

Do NOT auto-advance past decision gates.
</user-decision-handling>

<termination-states>
Workflow can terminate via:
1. `auto_terminate: true` on a gate (conditional termination)
2. `termination: true` on a skill (skill is endpoint)
3. `next: null` with no applicable gate (pause, not termination)
</termination-states>

</transition-resolution>

---

<workflow>

<phase id="0" name="load-and-validate">
<action>Load manifest and validate schema.</action>

<process>
1. Attempt to read `.claude/workflow-manifest.yaml`
2. If found: Parse and validate per schema-validation rules
3. If not found: Use fallback defaults, note in output
4. If validation errors: Display errors and halt
5. If validation warnings: Display warnings and continue
</process>
</phase>

<phase id="1" name="scan-handoffs">
<action>Scan handoff directory for completed outputs.</action>

<process>
1. Read `handoff_directory` from manifest (default: `.docs/`)
2. For each skill in manifest:
   a. Check if all `outputs` exist
   b. If all outputs exist: Mark skill as completed
   c. If some outputs exist: Mark as in_progress
   d. If no outputs exist: Mark as pending
3. Build completion map: { skill_name: status }
</process>

<file-validation>
For JSON outputs: Optionally validate JSON structure
For other files: Check existence only
</file-validation>
</phase>

<phase id="2" name="determine-current-position">
<action>Find current position in workflow.</action>

<process>
1. Start from entry_point skill
2. Follow completion chain:
   - If skill completed: Resolve next via transition-resolution
   - If skill pending: This is current position
   - If decision gate pending: Current position with pending decision
3. Track path taken through workflow
</process>

<current-phase-detection>
The "current phase" is determined by:
1. Find the first uncompleted skill in the active path
2. Look up that skill's `phase` field
3. Map to phase definition for display name
</current-phase-detection>
</phase>

<phase id="3" name="evaluate-next-step">
<action>Determine recommended next action.</action>

<process>
1. Get most recently completed skill
2. Apply transition-resolution rules
3. If gate pending: Prepare question for user
4. If skill determined: Check its prerequisites
5. Generate recommendation with rationale
</process>

<rationale-generation>
Explain why this skill is recommended:
- "Previous phase complete, ready for {skill}"
- "Required output {file} is present"
- "Decision gate resolved to {skill}"
- "Workflow at termination point: {label}"
</rationale-generation>
</phase>

<phase id="4" name="format-output">
<action>Generate output in requested mode.</action>

<process>
1. Check for output mode flags (--full, --trace)
2. Apply appropriate template from output-modes
3. Substitute values from analysis
4. Return formatted output
</process>
</phase>

</workflow>

---

<reusable-templates>

<prerequisite-check-template>
Other workflow skills should include this pattern in their Phase 0:

<flexible-prerequisite-check>
<action>Check for handoff documents and gather missing information conversationally.</action>

<check-process>
1. Scan .docs/ for expected handoff documents
2. If found: Load context and summarize conversationally
3. If missing: Gather equivalent information through questions
4. Proceed with skill workflow regardless
</check-process>

<when-prerequisites-exist>
"I can see you've completed {previous-phase} ({key-detail}).
Ready to {current-phase-action}?"

Then proceed with the skill's main workflow.
</when-prerequisites-exist>

<when-prerequisites-missing>
"I don't see {expected-documents}. No problem - let me gather what I need.

{Targeted questions to get equivalent information}

Got it. Let's proceed with {current-phase-action}."

Then proceed with the skill's main workflow.
</when-prerequisites-missing>

<key-principle>
Skills NEVER block on missing prerequisites. They gather information conversationally and proceed.
</key-principle>
</flexible-prerequisite-check>
</prerequisite-check-template>

<skill-specific-prerequisites>
| Skill | Expected JSON Handoffs | Questions if Missing |
|-------|------------------------|---------------------|
| solution-architect | brief.json | "What are you building? Greenfield or existing codebase? What platform?" |
| tech-stack-advisor | brief.json, architecture-context.json (optional) | "What's your platform? Greenfield or brownfield? Any constraints?" |
| deployment-advisor | tech-stack-decision.json | "What's your tech stack? Frontend, backend, database?" |
| project-spinup | tech-stack-decision.json, deployment-strategy.json | "Where will this deploy? What's your hosting preference?" |
| test-orchestrator | tech-stack-decision.json | "What framework are you using? What needs testing?" |
| deploy-guide | deployment-strategy.json | "Where are you deploying? VPS, cloud, shared hosting?" |
| ci-cd-implement | deployment-strategy.json | "What's your deployment target? What should CI test?" |
</skill-specific-prerequisites>

</reusable-templates>

---

<guardrails>

<must-do>
- Load manifest just-in-time (not cached across invocations)
- Validate manifest schema before use
- Fall back to hardcoded defaults if manifest missing/invalid
- Use structured condition predicates (not free-form strings)
- Respect gate-overrides-next rule
- Show minimal output by default (token efficiency)
- Provide --full and --trace modes for detailed views
- Display validation warnings to user
- Handle file/JSON errors gracefully
- Explain transition logic in recommendations
</must-do>

<must-not-do>
- Dump full manifest in default output mode
- Ignore validation errors (must halt)
- Auto-advance past decision gates requiring user input
- Use deprecated phase numbers (use skill names)
- Hardcode workflow logic (use manifest)
- Expose raw error messages (provide actionable guidance)
- Block on missing handoff files (mark as pending)
</must-not-do>

</guardrails>

---

<integration-notes>

<orchestrator-role>
This skill is the manifest-driven orchestrator for the workflow system:
- Single source of truth for workflow state
- Loads manifest on-demand (not embedded in CLAUDE.md)
- Evaluates conditions and gates at runtime
- Provides consistent status across all skills
</orchestrator-role>

<token-efficiency>
Progressive disclosure pattern:
- CLAUDE.md contains ~5 token pointer
- Manifest loaded only when workflow-status invoked
- Default output mode minimizes response tokens
- Full details available via --full flag
</token-efficiency>

<backward-compatibility>
If manifest is missing (pre-v3 projects):
- Use hardcoded fallback workflow
- Note: "Using built-in workflow (no manifest found)"
- Behavior matches v2 hardcoded logic
</backward-compatibility>

<manifest-location>
Primary: `.claude/workflow-manifest.yaml`
This is a per-project file created by workflow setup or project-spinup.
</manifest-location>

</integration-notes>
