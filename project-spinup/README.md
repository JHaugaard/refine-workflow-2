# project-spinup Skill

## Overview

Generate complete project foundations with personalized CLAUDE.md, Docker setup, and project structure. Supports Guided Setup (step-by-step learning) or Quick Start (full scaffolding).

**Use when:** After tech-stack-advisor and deployment-advisor have completed, to initialize new projects with your workflow, infrastructure, and best practices.

**Output:** Complete project foundation including CLAUDE.md, docker-compose.yml, directory structure, configuration files, and either guided learning prompts or full scaffolding.

---

## What's New (v2.0)

- **Environment Registry Integration**: Loads infrastructure context from `~/.claude/environment.json`
- **JSON Handoffs**: Reads upstream JSON handoffs and produces structured `project-foundation.json`
- **Approval Gate**: Confirms generation plan before creating files
- **Hard Boundaries**: Clear scope restrictions - executor role, not advisor

---

## How It Works

When invoked, this skill will:

1. **Phase 0 - Load Environment**: Read `~/.claude/environment.json` for infrastructure context (services, endpoints, established choices)
2. **Phase 1 - Load Handoffs**: Parse JSON handoffs from upstream skills (brief.json, tech-stack-decision.json, deployment-strategy.json)
3. **Phase 2 - Spinup Approach**: Ask about Guided Setup vs Quick Start preference
4. **Phase 3 - Approval Gate**: Present generation plan for user confirmation before creating files
5. **Phase 4 - Generate Files**: Create CLAUDE.md, Docker config, directory structure
6. **Phase 5 - Apply Approach**: Add guided prompts or full scaffolding
7. **Phase 6 - Create Handoff**: Save `.docs/project-foundation.json`
8. **Phase 7 - Summary**: Provide next steps and workflow status

---

## Skills Workflow Integration

This skill is **Phase 3 of 6** in the Skills workflow:

```
project-brief-writer (Phase 0) -> produces brief.json
    |
tech-stack-advisor (Phase 1) -> produces tech-stack-decision.json
    |
deployment-advisor (Phase 2) -> produces deployment-strategy.json
    |
project-spinup (Phase 3) <- YOU ARE HERE -> produces project-foundation.json
    |                       <- TERMINATION POINT (localhost)
    |
test-orchestrator (Phase 4) - optional
    |
deploy-guide (Phase 5) <- TERMINATION POINT (manual deploy)
    |
ci-cd-implement (Phase 6) <- TERMINATION POINT (full automation)
```

---

## Environment Registry Access

This skill accesses specific paths from `~/.claude/environment.json`:

| Path | Purpose |
|------|---------|
| `infrastructure.services` | Available services (Supabase, PocketBase, Redis) for docker-compose |
| `credentials.api_endpoints` | API URLs for .env.example templates |
| `established_choices` | Locked decisions (containerization, proxy, etc.) |

**Out of scope:** deployment_options, skill_guidance, infrastructure.vps (these belong to other skills)

If environment.json is missing, the skill gracefully degrades to hardcoded defaults.

---

## JSON Handoffs

### Input (from upstream skills)

- `.docs/PROJECT-MODE.md` - Workflow mode (LEARNING/DELIVERY/BALANCED)
- `.docs/brief.json` - Project brief from project-brief-writer
- `.docs/tech-stack-decision.json` - Technology decisions from tech-stack-advisor
- `.docs/deployment-strategy.json` - Deployment strategy from deployment-advisor

### Output

- `.docs/project-foundation.json` - Structured handoff for downstream skills

---

## Spinup Approach vs PROJECT-MODE

**Important distinction:**

- **PROJECT-MODE** (LEARNING/BALANCED/DELIVERY): Governs advisory skills - exploration, discussion, checkpoints. This is **strategic learning**.

- **spinup_approach** (Guided/Quick Start): Governs scaffolding style - step-by-step or all-at-once. This is **tactical implementation**.

**Key insight:** You can be in LEARNING mode but use Quick Start if you're already familiar with the tech stack. Or be in DELIVERY mode but use Guided Setup for a new framework.

---

## What Gets Generated

### Always Created (Both Approaches)

- CLAUDE.md - Comprehensive project context for AI assistant
- docker-compose.yml - Local development environment
- Directory structure - src/, tests/, docs/
- .gitignore - Tech-stack-appropriate
- README.md - Setup and development instructions
- .env.example - Required environment variables
- .docs/project-foundation.json - Structured handoff

### Guided Setup Adds

- Detailed "Next Steps" section with copy-paste prompts
- Learning objectives and estimated times
- Verification commands

### Quick Start Adds

- Complete source file structure
- Tech-stack-specific configurations
- Starter code with comments
- Sample tests
- All middleware/utilities

---

## Hard Boundaries

This skill operates as an **EXECUTOR**, not an advisor:

**WILL NOT:**

- Make new technology decisions (tech-stack-advisor's scope)
- Change deployment strategy (deployment-advisor's scope)
- Suggest alternative platforms or hosting options

**WILL:**

- Implement LOCKED decisions from upstream handoffs
- Use environment registry for infrastructure context
- Ask user before proceeding differently from upstream decisions

---

## Supported Tech Stacks

See [EXAMPLES.md](EXAMPLES.md) for detailed templates:

- **Next.js + Supabase** - App Router, TypeScript, Tailwind
- **PHP + MySQL** - MVC structure, PDO, PHPUnit
- **FastAPI + PostgreSQL** - SQLAlchemy, Alembic, pytest
- **Laravel + SQLite/PostgreSQL** - FrankenPHP, Eloquent

---

## Related Skills

- **project-brief-writer** - Creates brief.json (Phase 0)
- **tech-stack-advisor** - Creates tech-stack-decision.json (Phase 1)
- **deployment-advisor** - Creates deployment-strategy.json (Phase 2)
- **test-orchestrator** - Sets up testing infrastructure (Phase 4)
- **deploy-guide** - Executes deployment (Phase 5)
- **ci-cd-implement** - Creates CI/CD pipelines (Phase 6)
- **workflow-status** - Displays workflow progress

---

## Additional Files

- [SKILL.md](SKILL.md) - Full skill definition
- [EXAMPLES.md](EXAMPLES.md) - Detailed examples for each tech stack
- [QUICK_REFERENCE.md](QUICK_REFERENCE.md) - Quick reference guide

---

## Version History

### v2.0 (2025-12-10)

- Added Phase 0 environment registry loading
- Implemented JSON handoff input/output
- Added Approval Gate (Planning Mindset)
- Added hard-boundaries block
- Updated workflow status to Phase 3 of 6
- Externalized templates to EXAMPLES.md and QUICK_REFERENCE.md

### v1.3 (2025-11-17)

- Updated infrastructure list with VPS8 specs, Caddy, PocketBase
- Expanded deployment options reference

### v1.2 (2025-11-12)

- Added support for reading handoff documents
- Integrated complete context loading

### v1.1 (2025-11-12)

- Added workflow state visibility
- Separated spinup approach from PROJECT-MODE
- Implemented MODE-informed suggestions

### v1.0 (Initial)

- Initial release with CLAUDE.md generation and dual-mode scaffolding

---

**Version:** 2.0
**Last Updated:** 2025-12-10
