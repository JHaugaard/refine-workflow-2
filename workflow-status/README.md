# workflow-status Skill

## Overview

The workflow orchestrator that reads the manifest and provides intelligent navigation through the skills workflow. Displays progress, recommends next steps, and evaluates decision gates.

**Use when:** Checking workflow status, returning to a project after time away, or getting guidance on what to do next.

**Output:** Formatted status display with phase progress, next step recommendation, and decision gate prompts.

---

## How It Works

When invoked, this skill will:

1. **Load manifest** - Read `.claude/workflow-manifest.yaml` (just-in-time)
2. **Scan handoffs** - Check `.docs/` directory for completed handoff documents
3. **Evaluate state** - Determine which skills have completed based on outputs
4. **Check gates** - Evaluate any decision gates that apply to current position
5. **Recommend next** - Provide actionable next step with reasoning
6. **Display status** - Present formatted output with all findings

---

## Manifest-Based Orchestration

This skill is the central orchestrator for the workflow system. It:

- Loads the workflow manifest just-in-time (zero token overhead when not invoked)
- Evaluates structured condition predicates to determine routing
- Presents decision gates when user input is required
- Validates manifest structure on load

Other skills are "pure capabilities" — they focus on their domain and produce handoff documents. This skill handles all workflow coordination.

---

## Output Modes

### Default Mode (minimal context)

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

### --full Mode (complete view)

Shows all phases, all skills, all decision points, full dependency graph.

### --trace Mode (debugging)

Machine-readable JSON showing files checked, gates evaluated, transitions applied.

---

## Reference Files

- [HANDOFF-DOCUMENTATION.md](references/HANDOFF-DOCUMENTATION.md) - Complete documentation of the handoff system
- [workflow-manifest.yaml](../.claude/workflow-manifest.yaml) - The workflow definition

---

## Version History

### v2.0 (2025-12-16)

Manifest-Based Orchestrator

- Transformed from status display to full orchestrator
- Added just-in-time manifest loading
- Added structured condition evaluation
- Added decision gate handling
- Added output modes (default, --full, --trace)
- Added manifest validation

### v1.0 (2025-11-19)

Initial Release

- Workflow status detection and display
- Phase progress analysis from handoff documents
- Next step recommendations
- Reusable prerequisite-check templates for other skills

---

**Version:** 2.0
**Last Updated:** 2025-12-16
