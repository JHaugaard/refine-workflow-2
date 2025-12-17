# Skills Workflow Handoff Documentation

## Overview

The skills workflow consists of 9 skills that produce **handoff documents** in a `.docs/` subdirectory. These documents serve as session bridges, ensuring decisions and context are preserved across multiple Claude sessions.

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    CLAUDE.md (lightweight)                   │
│  Points to: .claude/workflow-manifest.yaml                  │
│  Points to: .docs/ directory                                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              .claude/workflow-manifest.yaml                  │
│  - Defines all skills, phases, dependencies                 │
│  - Contains decision gates and routing logic                │
│  - Loaded just-in-time by workflow-status                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   workflow-status skill                      │
│  - Reads manifest on-demand                                 │
│  - Evaluates handoff document state                         │
│  - Recommends next skill based on current state             │
└─────────────────────────────────────────────────────────────┘
```

### Key Design Principles

**Manifest-Based Orchestration:** Workflow routing lives in `.claude/workflow-manifest.yaml`, not embedded in individual skills. Skills are "pure capabilities" — they focus on their domain and produce handoff documents.

**Just-In-Time Loading:** The manifest is only loaded when `workflow-status` is invoked. Non-workflow interactions incur zero token overhead.

**Progressive Disclosure:** Skills load their full definition only when invoked. The orchestrator provides minimal context by default.

---

## Workflow Phases

```
DISCOVERY PHASE
   project-brief-writer
      Output: .docs/brief.json, .docs/PROJECT-MODE.md

ARCHITECTURE PHASE
   solution-architect (optional)
      Output: .docs/architecture-context.json
   tech-stack-advisor
      Output: .docs/tech-stack-decision.json

PLANNING PHASE
   deployment-advisor
      Output: .docs/deployment-strategy.json

IMPLEMENTATION PHASE
   project-spinup
      Output: Project foundation + CLAUDE.md
      TERMINATION POINT: Localhost/learning projects

[MANUAL DEVELOPMENT - User builds features]

QUALITY PHASE (optional, when ready)
   test-orchestrator
      Output: .docs/test-strategy.md, test scaffolding

RELEASE PHASE
   deploy-guide
      Output: .docs/deployment-log.json
      TERMINATION POINT: Manual deployment only
   ci-cd-implement
      Output: .github/workflows/*, CICD-SECRETS.md
      TERMINATION POINT: Full automation

UTILITY (anytime)
   workflow-status - Check progress, get guidance
```

---

## Termination Points

The workflow has three natural stopping points based on project needs:

### 1. After project-spinup
**For:** Localhost-only projects, learning exercises, personal utilities, native apps (iOS/macOS/Tauri)
- Project runs locally on developer machine
- No public deployment needed
- Development complete or ongoing without deployment

### 2. After deploy-guide
**For:** Projects needing deployment but not CI/CD automation
- Manual deployment workflow established
- No automated pipelines
- Suitable for low-change-frequency projects

### 3. After ci-cd-implement
**For:** Projects needing full automation
- Automated testing on every push
- Automated deployment to production
- Complete DevOps pipeline

---

## Handoff Document Chain

```
project-brief-writer
    │
    Creates: .docs/PROJECT-MODE.md + .docs/brief.json
    │
solution-architect (if invoked)
    │
    Reads: .docs/brief.json
    Creates: .docs/architecture-context.json
    │
tech-stack-advisor
    │
    Reads: .docs/PROJECT-MODE.md + .docs/brief.json
    Optional: .docs/architecture-context.json
    Creates: .docs/tech-stack-decision.json
    │
deployment-advisor
    │
    Reads: .docs/PROJECT-MODE.md + .docs/tech-stack-decision.json
    Creates: .docs/deployment-strategy.json
    │
project-spinup
    │
    Reads: All .docs/ documents
    Creates: Project foundation + CLAUDE.md
    │
    [TERMINATION POINT: Localhost/learning projects]
    │
    [MANUAL DEVELOPMENT PHASE]
    │
test-orchestrator (optional, when ready)
    │
    Reads: CLAUDE.md, project structure
    Creates: .docs/test-strategy.md, test scaffolding
    │
deploy-guide
    │
    Reads: .docs/deployment-strategy.json, project structure
    Creates: .docs/deployment-log.json
    │
    [TERMINATION POINT: Manual deployment only]
    │
ci-cd-implement
    │
    Reads: .docs/deployment-strategy.json, project structure
    Creates: .github/workflows/*, CICD-SECRETS.md
    │
    [TERMINATION POINT: Full automation]
```

---

## Handoff Documents

### 1. PROJECT-MODE.md

**Created by:** project-brief-writer (automatically, before gathering project information)

**Location:** `.docs/PROJECT-MODE.md`

**Purpose:** Declares workflow commitment and checkpoint strictness

**Contains:**
- Workflow mode (LEARNING/BALANCED/DELIVERY)
- Decision date and context
- Mode implications for subsequent skills
- Workflow commitments
- Success criteria

**Key Feature:** Governs strategic learning — determines how much exploration, discussion, and checkpoint validation happens during tech stack and deployment decisions.

---

### 2. Project Brief (brief.json)

**Created by:** project-brief-writer

**Location:** `.docs/brief.json`

**Purpose:** Problem-focused project requirements without technical implementation details

**Contains:**
- Project overview (narrative)
- Problem statement
- Project goals (primary + secondary)
- Functional requirements
- Success criteria
- Use cases & scenarios
- Edge cases to handle
- Constraints & assumptions
- Out of scope items
- Learning goals (optional)
- Initial tech thoughts (quarantined in separate section)
- Deployment intent (localhost-only, public, or TBD)

**Key Feature:** Technology-agnostic description focusing on WHAT to build, not HOW.

---

### 3. Architecture Context (architecture-context.json)

**Created by:** solution-architect

**Location:** `.docs/architecture-context.json`

**Purpose:** Resolve architectural ambiguity before tech stack selection

**Contains:**
- Scale profile (personal, small-team, growing, enterprise)
- Platform decision (web, native, desktop, CLI)
- Integration requirements
- Constraints (budget, compliance, data residency)
- Decisions array with rationale

**Key Feature:** Optional skill for complex projects needing architectural clarity before technology selection.

---

### 4. Tech Stack Decision (tech-stack-decision.json)

**Created by:** tech-stack-advisor (AFTER collaborative refinement and user convergence)

**Location:** `.docs/tech-stack-decision.json`

**Purpose:** Complete technology stack recommendation with detailed rationale

**Contains:**
- Primary tech stack recommendation with rationale
- Complete stack breakdown (frontend, backend, database, auth, styling, storage, testing)
- Platform decision (if non-web)
- Learning opportunities
- Cost analysis
- Alternative options with trade-offs
- Ruled-out options with explanations

**Key Feature:** Created AFTER collaborative discussion, not before. Reflects actual user decision.

---

### 5. Deployment Strategy (deployment-strategy.json)

**Created by:** deployment-advisor (AFTER collaborative refinement and user convergence)

**Location:** `.docs/deployment-strategy.json`

**Purpose:** Complete hosting and deployment strategy with concrete workflows

**Contains:**
- Primary hosting recommendation with rationale
- Hosting details (provider, server type, container approach, database, storage, CDN, SSL)
- Deployment workflow (initial setup, regular deployment, rollback)
- Development environment decision
- Cost breakdown and scaling path
- Alternative hosting options
- Monitoring & maintenance schedule
- Backup strategy
- Security considerations

**Key Feature:** Localhost option enables early termination for learning projects. Created AFTER collaborative discussion.

---

### 6. Project Foundation (CLAUDE.md)

**Created by:** project-spinup

**Location:** `[project-dir]/CLAUDE.md`

**Purpose:** Complete project context for all future Claude Code development sessions

**Contains:**
- Developer profile
- Project overview (description, tech stack, architecture decisions)
- Development environment setup
- Infrastructure & hosting (from deployment-strategy.json)
- Development workflow (git, commits, testing, deployment)
- Code conventions
- Common commands
- Project-specific notes
- Deployment workflow and checklist
- Resources & references
- Troubleshooting

**Also creates:**
- docker-compose.yml (local development environment)
- Directory structure (src/, tests/, docs/)
- .gitignore (tech-stack-appropriate)
- README.md (setup and development instructions)
- .env.example (required environment variables)

**Key Feature:** Loads ALL handoff documents to preserve complete context chain. Natural termination point for localhost/learning projects.

---

### 7. Test Strategy (test-strategy.md)

**Created by:** test-orchestrator

**Location:** `.docs/test-strategy.md`

**Purpose:** Document test strategy decisions and serve as reference

**Contains:**
- Testing philosophy and approach
- Test types implemented (unit, integration, e2e)
- Test file organization
- Coverage goals
- Test commands and scripts
- CI integration notes
- Guidance for writing new tests

**Key Feature:** Standalone skill that can be invoked anytime after tech-stack selection.

---

### 8. Deployment Log (deployment-log.json)

**Created by:** deploy-guide

**Location:** `.docs/deployment-log.json`

**Purpose:** Record of deployment actions taken and ongoing reference

**Contains:**
- Deployment date and target
- Pre-deployment checklist completed
- Deployment steps executed
- Post-deployment verification
- Environment URLs
- Credentials/secrets notes (references, not values)
- Rollback procedure
- Lessons learned

**Key Feature:** Living document updated with each deployment. Termination point for projects without CI/CD.

---

### 9. CI/CD Outputs

**Created by:** ci-cd-implement

**Location:** `.github/workflows/`, `scripts/`, `CICD-SECRETS.md`

**Purpose:** Automated pipelines for testing and deployment

**Contains:**
- `.github/workflows/ci.yml` - Continuous Integration (test, lint, build)
- `.github/workflows/deploy.yml` - Continuous Deployment
- `scripts/deploy.sh` - Manual deployment script
- `scripts/rollback.sh` - Rollback script
- `CICD-SECRETS.md` - Secrets documentation

**Key Feature:** Final workflow skill. Analyzes project structure and deployment-strategy.json to generate appropriate pipelines.

---

## Flexible Entry Pattern

Skills can be invoked standalone without all prerequisites. The flexible entry pattern handles missing context:

### Standard Flow (prerequisites exist)
```
"I can see you've completed [previous phase]. Ready to [this phase action]?"
→ Proceed with skill workflow
```

### Flexible Entry (prerequisites missing)
```
"I don't see [expected handoff docs]. No problem - let me gather what I need.
[Ask targeted questions to get equivalent information]
Got it. Let's proceed with [this phase action]."
→ Proceed with skill workflow
```

**Key principle:** Skills gather missing info conversationally, never block.

### Example: Invoking deploy-guide without deployment-strategy.json

```
User: "Use deploy-guide skill"

Claude: "I don't see .docs/deployment-strategy.json. No problem!

To help you deploy, I need to know:
1. Where are you deploying? (VPS, Cloudflare Pages, fly.io, shared hosting)
2. What's your tech stack?
3. Do you have any existing infrastructure?

[User provides answers]

Got it. Let's deploy your app to [target]."
→ Proceeds with deployment guidance
```

---

## Session Bridge Pattern

Each handoff document is designed to be **session-independent**, meaning:

1. **Self-contained**: Contains all context needed to understand the decision
2. **Rationale-rich**: Explains WHY decisions were made, not just WHAT was decided
3. **Action-ready**: Provides concrete next steps and workflows
4. **Cross-session**: Can be loaded in a fresh Claude session months later
5. **Created after convergence**: Documents are written AFTER collaborative refinement with the user

### When Are Handoff Documents Created?

**Critical timing rule:** Handoff files are created **after the user has discussed and confirmed the primary recommendation** in each skill phase.

**The workflow for advisory skills:**
1. Skill presents recommendations in console (primary + alternatives + ruled-out)
2. User asks questions, explores alternatives, discusses trade-offs
3. User confirms choice or converges on final decision
4. **THEN** skill creates handoff document with complete analysis
5. Skill confirms document location

**Why this timing matters:**
- Ensures document reflects actual decision, not preliminary recommendation
- Captures any modifications made during discussion
- Prevents premature file creation that might need rewriting
- Guarantees accuracy of handoff

---

## File Organization

All handoff documents go in `.docs/` subdirectory:

```
project-workspace/
├── .claude/                            # Claude configuration
│   └── workflow-manifest.yaml          # Workflow definition (if using)
├── .docs/                              # All handoff documents
│   ├── PROJECT-MODE.md                 # Workflow mode
│   ├── brief.json                      # Project brief
│   ├── architecture-context.json       # Architecture decisions (optional)
│   ├── tech-stack-decision.json        # Tech stack recommendation
│   ├── deployment-strategy.json        # Deployment strategy
│   ├── test-strategy.md                # Test strategy (optional)
│   └── deployment-log.json             # Deployment record
├── CLAUDE.md                           # Project context (project-spinup output)
├── docker-compose.yml
├── README.md
├── .env.example
├── .gitignore
├── src/
├── tests/
├── docs/
├── .github/                            # CI/CD output
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
├── scripts/                            # CI/CD output
│   ├── deploy.sh
│   └── rollback.sh
└── CICD-SECRETS.md                     # CI/CD output
```

**Note:** The `.docs/` directory keeps handoff documents organized and separate from project source code.

---

## Validation Checklist

To verify your handoff chain is complete at each phase:

**After project-brief-writer:**
- [ ] .docs/PROJECT-MODE.md exists and declares mode
- [ ] .docs/brief.json exists and is problem-focused
- [ ] Brief includes deployment intent

**After solution-architect (if used):**
- [ ] .docs/architecture-context.json exists
- [ ] Includes scale profile and platform decision
- [ ] Decisions array has rationale

**After tech-stack-advisor:**
- [ ] .docs/tech-stack-decision.json exists with complete recommendation
- [ ] Includes primary recommendation, alternatives, and ruled-out options
- [ ] Created AFTER collaborative discussion

**After deployment-advisor:**
- [ ] .docs/deployment-strategy.json exists with complete strategy
- [ ] Includes deployment workflow, cost breakdown, scaling path
- [ ] Created AFTER collaborative discussion

**After project-spinup:**
- [ ] Project directory exists with CLAUDE.md
- [ ] docker-compose.yml configured for local development
- [ ] README.md has setup instructions
- [ ] .env.example lists required environment variables
- [ ] **TERMINATION CHECK:** If localhost-only project, workflow complete here

**After test-orchestrator (if used):**
- [ ] .docs/test-strategy.md exists with testing approach
- [ ] Test scaffolding created
- [ ] Test commands documented

**After deploy-guide:**
- [ ] Application deployed to target environment
- [ ] .docs/deployment-log.json created
- [ ] Post-deployment verification complete
- [ ] **TERMINATION CHECK:** If no CI/CD needed, workflow complete here

**After ci-cd-implement:**
- [ ] .github/workflows/ contains CI and/or CD workflows
- [ ] CICD-SECRETS.md documents required secrets
- [ ] Pipeline tested with initial push
- [ ] **WORKFLOW COMPLETE**

---

## Revising Decisions

If you need to change a decision made in a previous phase:

### Revising Tech Stack

**Procedure:**
1. Delete `.docs/tech-stack-decision.json` and `.docs/deployment-strategy.json` (if it exists)
2. Re-invoke tech-stack-advisor skill
3. Make new tech stack decision
4. Proceed with deployment-advisor

**Why delete deployment-strategy.json?** Deployment strategy depends on tech stack. Deleting ensures consistency.

### Revising Deployment Strategy

**Procedure:**
1. Delete `.docs/deployment-strategy.json`
2. Re-invoke deployment-advisor skill
3. Make new deployment decision
4. Proceed with project-spinup

### Revising Project Mode

**Procedure:**
1. Edit `.docs/PROJECT-MODE.md` directly
2. Change the mode line
3. Update the mode details section
4. Continue with next skill

### Starting Over

**Procedure:**
1. Delete entire `.docs/` directory
2. Re-invoke project-brief-writer skill
3. Proceed through workflow again

---

## Key Concepts

### Manifest-Based Orchestration

Workflow routing is defined in `.claude/workflow-manifest.yaml`, not embedded in skills. Benefits:
- Skills are simpler — focused on their domain only
- Routing logic is centralized and auditable
- Changes to workflow don't require editing multiple skills
- Just-in-time loading reduces token overhead

### Strategic Learning vs Tactical Learning

**Strategic Learning (PROJECT-MODE.md):**
- Governs advisory skills
- Determines checkpoint strictness and exploration depth
- About understanding trade-offs, evaluating options
- LEARNING mode = detailed checkpoints, DELIVERY mode = minimal checkpoints

**Tactical Learning (spinup_approach in project-spinup):**
- Governs implementation scaffolding style
- Determines whether to build incrementally (Guided Setup) or all at once (Quick Start)
- About familiarity with the chosen tech stack
- Independent of PROJECT-MODE

### Flexible Entry

Skills don't require strict prerequisite enforcement. If handoff documents are missing, skills gather equivalent information conversationally. This enables:
- Starting at any phase when context is clear
- Using skills standalone outside the workflow
- Recovering from incomplete workflows

### Termination Points

The workflow has three natural stopping points:
1. **After project-spinup** - For localhost/learning/native projects
2. **After deploy-guide** - For manually deployed projects
3. **After ci-cd-implement** - For fully automated projects

Users choose their stopping point based on project needs.

### Collaborative Refinement

Handoff documents are created AFTER collaborative discussion with the user. This ensures:
- User convergence on recommendations
- Opportunity to explore alternatives
- Questions answered before documenting
- Accurate representation of final decision

---

## Version History

### v4.0 (2025-12-16)
**Manifest-based orchestration refactor**

- Updated to manifest-based architecture (`.claude/workflow-manifest.yaml`)
- Documented just-in-time loading pattern
- Skills no longer contain routing logic
- Added architecture diagram
- Updated file format references (JSON for structured handoffs)
- Simplified handoff document descriptions
- Added solution-architect to documentation
- Aligned with v3 manifest schema

### v3.0 (2025-11-22)
**Major restructure for refined workflow**

- Updated to 7 phases (0-6) with 8 skills
- Added three termination points
- Changed handoff document location to `.docs/` subdirectory
- Added new handoff documents: test-strategy.md, deployment-log.md
- Added flexible entry pattern documentation
- Added new skills: test-orchestrator, deploy-guide

### v2.1 (2025-01-17)
**v1.0-ready refinements based on comprehensive review**

- Reduced checkpoint questions from 5 to 3 in LEARNING mode
- Added "When Are Handoff Documents Created?" section
- Added comprehensive "Revising Decisions" section

### v2.0 (2025-01-17)
**Major update aligned with current SKILL.md implementations**

- Updated to reflect Phase 0-3 structure
- Documented handoff document creation timing
- Added key concepts section

### v1.0 (2025-11-12)
**Initial handoff documentation system**

- Established handoff chain pattern
- Created session bridge pattern for workflow continuity
