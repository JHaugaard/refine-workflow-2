# Skill Manifest: A CLAUDE.md-Based Workflow Orchestration Pattern

## Overview

This document describes a pattern for organizing, orchestrating, and deploying related Claude Code skills using a declarative manifest embedded in CLAUDE.md. The goal is to create a **single source of truth** for workflow logic, enabling lighter-weight skills and more flexible composition.

## The Problem

Currently, Claude Code skills are standalone units that each carry their own:
- Prerequisite checking logic
- Handoff instructions ("run X next")
- Awareness of where they fit in a larger workflow

This creates several issues:
1. **Duplication** - Multiple skills repeat similar orchestration logic
2. **Fragility** - Changing a workflow means editing multiple skill files
3. **Tight coupling** - Skills "know" about each other, reducing reusability
4. **No single view** - Understanding the full workflow requires reading all skills

## The Solution: Manifest in CLAUDE.md

Embed a declarative workflow manifest in CLAUDE.md that:
- Defines which skills exist and their relationships
- Specifies inputs/outputs and dependencies
- Handles branching logic and conditional flows
- Serves as living documentation of the workflow

Skills become **pure capability units** focused solely on their task, while the manifest handles all orchestration concerns.

---

## Core Concepts

### 1. Skills as Pure Capabilities

With a manifest handling orchestration, skills shed their workflow awareness:

**Before (orchestration-heavy skill):**
```markdown
# tech-stack-advisor

## Prerequisites
- Ensure .docs/project-brief.md exists
- If not, run project-brief-writer first

## After Completion
- Produces .docs/tech-stack-decision.md
- Next: run deployment-advisor

## Purpose
[actual skill content...]
```

**After (pure capability):**
```markdown
# tech-stack-advisor

## Purpose
Analyze project requirements and recommend appropriate technology stacks with detailed rationale.

## What This Skill Does
[focused skill content - no orchestration logic]
```

### 2. The Manifest Structure

The manifest lives in CLAUDE.md and follows this structure:

```yaml
## Skill Workflow Manifest

```yaml
workflow: [workflow-name]
version: [semver]
description: [what this workflow accomplishes]

# Output directory for handoff documents
handoff_directory: ".docs/"

# Phase definitions (optional, for grouping)
phases:
  discovery: "Understanding the problem space"
  architecture: "Technical decisions and design"
  planning: "Deployment and infrastructure strategy"
  implementation: "Project generation and setup"

# Skill definitions
skills:
  skill-name:
    phase: [phase-id]
    description: [what this skill does]
    requires: [list of input files or prior skills]
    outputs: [list of files this skill produces]
    next: [next skill(s) or conditional block]
    optional: [true/false - can be skipped]
    terminal: [true/false - ends the workflow]

# Decision gates requiring user input
decision_gates:
  - after: [skill-name]
    ask: [question to present to user]

# Workflow variants (optional)
variants:
  minimal: [list of skills for fast path]
  full: [list of skills for comprehensive path]
```
```

### 3. Dependency and Flow Control

**Simple linear flow:**
```yaml
skills:
  step-one:
    outputs: [".docs/output-a.md"]
    next: step-two

  step-two:
    requires: [".docs/output-a.md"]
    next: step-three
```

**Conditional branching:**
```yaml
skills:
  analyzer:
    outputs: [".docs/analysis.md"]
    next:
      default: standard-path
      conditions:
        - if: "analysis.complexity == 'high'"
          then: detailed-review
        - if: "analysis.type == 'legacy'"
          then: legacy-handler
```

**Parallel paths:**
```yaml
skills:
  planning-complete:
    next: [frontend-setup, backend-setup, infra-setup]  # all run in parallel
```

**Optional skills:**
```yaml
skills:
  ci-cd-implement:
    optional: true
    skip_conditions:
      - "deployment.target == 'local-only'"
```

---

## Implementation Pattern

### Manifest-Aware Orchestrator

A single skill (or enhancement to `workflow-status`) reads the manifest and provides:

1. **State detection** - Scans handoff_directory for completed outputs
2. **Next step determination** - Consults manifest for valid transitions
3. **Prerequisite validation** - Checks required files exist before invoking skills
4. **Progress visualization** - Shows workflow state based on manifest structure

### Example Orchestrator Behavior

```
User: "What's my workflow status?"

Orchestrator reads manifest, checks .docs/, responds:

  Workflow: project-development (v1.0)

  [x] Phase 1: Discovery
      [x] project-brief-writer -> .docs/project-brief.md

  [ ] Phase 2: Architecture
      [ ] tech-stack-advisor (ready - prerequisites met)
      [ ] solution-architect (optional, skipped)

  [ ] Phase 3: Planning
      [ ] deployment-advisor (blocked - needs tech-stack-decision.md)

  [ ] Phase 4: Implementation
      [ ] project-spinup (blocked)

  Next recommended: tech-stack-advisor
```

---

## Benefits

### Single Source of Truth
- Change workflow logic in one place
- No hunting through multiple skill files
- Manifest IS the documentation

### Lighter Skills
- Skills focus purely on their capability
- More reusable across different workflows
- Easier to maintain and test

### Flexible Composition
- Same skills, different manifests for different contexts
- Easy to create workflow variants (minimal, full, enterprise)
- Branching and conditionals handled declaratively

### Better Visibility
- Workflow structure is explicit and readable
- Easy to understand dependencies at a glance
- Progress tracking against a known structure

---

## Example: Project Development Workflow

```yaml
## Skill Workflow Manifest

```yaml
workflow: project-development
version: 1.0.0
description: "End-to-end workflow from idea to deployed project"
handoff_directory: ".docs/"

phases:
  discovery: "Understanding the problem"
  architecture: "Technical decisions"
  planning: "Deployment strategy"
  implementation: "Project generation"
  operations: "CI/CD and deployment"

skills:
  project-brief-writer:
    phase: discovery
    description: "Transform rough ideas into structured project briefs"
    outputs:
      - ".docs/project-brief.md"
    next:
      default: tech-stack-advisor
      conditions:
        - if: "brief.needs_architecture_review"
          then: solution-architect

  solution-architect:
    phase: architecture
    description: "Resolve architectural ambiguity for complex projects"
    requires:
      - ".docs/project-brief.md"
    outputs:
      - ".docs/architecture-context.json"
    next: tech-stack-advisor
    optional: true

  tech-stack-advisor:
    phase: architecture
    description: "Recommend technology stack with rationale"
    requires:
      - ".docs/project-brief.md"
    outputs:
      - ".docs/tech-stack-decision.md"
    next: deployment-advisor

  deployment-advisor:
    phase: planning
    description: "Recommend hosting and deployment strategy"
    requires:
      - ".docs/tech-stack-decision.md"
    outputs:
      - ".docs/deployment-decision.md"
    next: project-spinup

  project-spinup:
    phase: implementation
    description: "Generate project foundation with CLAUDE.md and structure"
    requires:
      - ".docs/tech-stack-decision.md"
      - ".docs/deployment-decision.md"
    outputs:
      - "CLAUDE.md"
      - "docker-compose.yml"
      - "project structure"
    next:
      default: test-orchestrator
      optional: [ci-cd-implement]

  test-orchestrator:
    phase: implementation
    description: "Set up testing infrastructure"
    optional: true
    next: ci-cd-implement

  ci-cd-implement:
    phase: operations
    description: "Implement CI/CD pipelines"
    optional: true
    next: deploy-guide

  deploy-guide:
    phase: operations
    description: "Guide through actual deployment"
    terminal: true

decision_gates:
  - after: project-brief-writer
    ask: "Does this project need detailed architecture review?"
  - after: tech-stack-advisor
    ask: "Proceed with the recommended stack?"
  - after: project-spinup
    ask: "Set up testing infrastructure now?"

variants:
  minimal:
    description: "Fast path for simple projects"
    skills: [project-brief-writer, tech-stack-advisor, project-spinup]
  full:
    description: "Comprehensive path with all optional steps"
    skills: all
```
```

---

## Getting Started

### 1. Create Manifest Template
Start with a CLAUDE.md template that includes the manifest structure for your workflow type.

### 2. Refactor Skills (Gradually)
Remove orchestration logic from skills as you add them to the manifest. Skills should focus on:
- What they do (purpose)
- How they do it (process/steps)
- What good output looks like (quality criteria)

### 3. Implement/Enhance Orchestrator
Create or modify a workflow-status skill to:
- Parse the manifest from CLAUDE.md
- Track state via handoff_directory
- Present next steps and handle branching

### 4. Create Workflow Variants
Build different CLAUDE.md templates for different contexts:
- `templates/mvp-workflow.md` - Minimal viable workflow
- `templates/enterprise-workflow.md` - Full review process
- `templates/learning-project.md` - Extra guidance, slower pace

---

## Open Questions for Future Development

1. **Manifest validation** - Should there be a schema validator for manifests?
2. **Skill discoverability** - How does the orchestrator know which skills are available?
3. **Cross-project workflows** - Can manifests reference skills from different sources?
4. **State persistence** - Beyond file existence, how to track partial completion?
5. **Rollback/restart** - How to handle "start over from phase X"?

---

## Related Concepts

- **Plugins** - Bundle skills for distribution (black box for now, to be explored)
- **Agents** - Programmatic orchestration (more flexible, more complex)
- **MCP Servers** - External tool integration
- **Hooks** - Event-driven automation

---

## Next Steps

1. Create a new project directory for this work
2. Design the manifest schema in detail
3. Prototype the orchestrator skill
4. Refactor 2-3 existing skills to be manifest-aware
5. Test with a real project workflow
