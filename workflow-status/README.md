# workflow-status Skill

## Overview

Display project workflow progress by reading handoff documents in the .docs/ directory. Presents a clear picture of completed phases, pending phases, and actionable next steps. Also provides reusable prerequisite-checking templates for other workflow skills to integrate.

**Use when:** Checking workflow status, returning to a project after time away, or verifying prerequisites before running a workflow skill.

**Output:** Formatted status display with phase progress, next step recommendation, and list of found handoff documents.

---

## How It Works

When invoked, this skill will:

1. **Detect context** - Read project directory name and scan .docs/ for handoff documents
2. **Analyze progress** - Determine completion status of each workflow phase
3. **Determine next step** - Identify actionable recommendation based on current progress
4. **Display status** - Present formatted output with all findings

---

## Workflow Integration

This skill serves two roles:

1. **Standalone utility** - Users invoke directly to check status
2. **Prerequisite provider** - Other skills use reusable templates for their Step 0

### Reusable Templates

The skill includes prerequisite-check templates that other workflow skills (tech-stack-advisor, deployment-advisor, project-spinup) can reference to auto-check status and report conversationally before proceeding.

---

## Reference Files

- **HANDOFF-DOCUMENTATION.md** - Complete documentation of the handoff system, revision guidance, and troubleshooting

---

## Version History

### v1.0 (2025-11-19)
**Initial Release**

- Workflow status detection and display
- Phase progress analysis from handoff documents
- Next step recommendations
- Reusable prerequisite-check templates for other skills
- HANDOFF-DOCUMENTATION.md reference

---

**Version:** 1.0
**Last Updated:** 2025-11-19
