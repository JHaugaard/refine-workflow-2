# Skills Workflow Handoff Documentation

## Overview

The skills workflow consists of 8 skills across 7 phases (Phase 0-6), with each phase creating **handoff documents** in a `.docs/` subdirectory that serve as session bridges for the next skill. This ensures that decisions and context are preserved across multiple Claude sessions.

### Workflow Structure

```
PLANNING PHASE
   Phase 0: project-brief-writer
      Output: .docs/PROJECT-MODE.md, .docs/brief-*.md
   Phase 1: tech-stack-advisor
      Output: .docs/tech-stack-decision.md
   Phase 2: deployment-advisor
      Output: .docs/deployment-strategy.md

SETUP PHASE
   Phase 3: project-spinup
      Output: Project foundation + claude.md
      TERMINATION POINT: Localhost/learning projects

[MANUAL DEVELOPMENT - User builds features]

QUALITY PHASE (optional, when ready)
   Phase 4: test-orchestrator
      Output: .docs/test-strategy.md, test scaffolding

RELEASE PHASE
   Phase 5: deploy-guide
      Output: .docs/deployment-log.md, deployed application
      TERMINATION POINT: Manual deployment only
   Phase 6: ci-cd-implement
      Output: .github/workflows/*, scripts/*
      TERMINATION POINT: Full automation

UTILITY (anytime)
   workflow-status - Check progress, get guidance
```

## Termination Points

The workflow has three natural stopping points based on project needs:

### 1. After project-spinup (Phase 3)
**For:** Localhost-only projects, learning exercises, personal utilities
- Project runs locally on developer machine
- No public deployment needed
- Development complete or ongoing without deployment

### 2. After deploy-guide (Phase 5)
**For:** Projects needing deployment but not CI/CD automation
- Manual deployment workflow established
- No automated pipelines
- Suitable for low-change-frequency projects

### 3. After ci-cd-implement (Phase 6)
**For:** Projects needing full automation
- Automated testing on every push
- Automated deployment to production
- Complete DevOps pipeline

## Complete Handoff Chain

```
Phase 0: project-brief-writer
    |
    Creates: .docs/PROJECT-MODE.md + .docs/brief-[name].md
    |
Phase 1: tech-stack-advisor
    |
    Reads: .docs/PROJECT-MODE.md + .docs/brief-*.md
    Creates: .docs/tech-stack-decision.md
    |
Phase 2: deployment-advisor
    |
    Reads: .docs/PROJECT-MODE.md + .docs/tech-stack-decision.md
    Creates: .docs/deployment-strategy.md
    |
Phase 3: project-spinup
    |
    Reads: All .docs/ documents
    Creates: Project foundation + claude.md
    |
    [TERMINATION POINT: Localhost/learning projects]
    |
    [MANUAL DEVELOPMENT PHASE]
    |
Phase 4: test-orchestrator (optional, when ready)
    |
    Reads: claude.md, project structure
    Creates: .docs/test-strategy.md, test scaffolding
    |
Phase 5: deploy-guide
    |
    Reads: .docs/deployment-strategy.md, project structure
    Creates: .docs/deployment-log.md
    |
    [TERMINATION POINT: Manual deployment only]
    |
Phase 6: ci-cd-implement
    |
    Reads: .docs/deployment-strategy.md, project structure
    Creates: .github/workflows/*, CICD-SECRETS.md
    |
    [TERMINATION POINT: Full automation]
```

---

## Handoff Documents

### 1. PROJECT-MODE.md (Phase 0)

**Created by:** project-brief-writer (automatically, before gathering project information)

**Location:** `.docs/PROJECT-MODE.md`

**Purpose:** Declares workflow commitment and checkpoint strictness for the entire Skills workflow

**Contains:**
- Workflow mode (LEARNING/BALANCED/DELIVERY)
- Decision date and context
- Mode implications for subsequent skills
- Workflow commitments
- Anti-bypass protections
- Success criteria

**Used by:** All subsequent skills (tech-stack-advisor, deployment-advisor, project-spinup)

**Key Feature:** Governs strategic learning (advisory workflow) - determines how much exploration, discussion, and checkpoint validation happens during tech stack and deployment decisions

---

### 2. Project Brief File (Phase 0)

**Created by:** project-brief-writer

**Location:** `.docs/brief-[project-name].md`

**Purpose:** Problem-focused project requirements without technical implementation details (prevents Over-Specification Problem)

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
- Next steps

**Used by:** tech-stack-advisor (for requirements analysis)

**Key Feature:** Technology-agnostic description focusing on WHAT to build, not HOW. Over-specification detection ensures implementation details are quarantined. Deployment intent helps identify early localhost termination.

---

### 3. tech-stack-decision.md (Phase 1)

**Created by:** tech-stack-advisor (AFTER collaborative refinement and user convergence)

**Location:** `.docs/tech-stack-decision.md`

**Purpose:** Complete technology stack recommendation with detailed rationale serving as session bridge

**Contains:**
- Primary tech stack recommendation with rationale
  - Why it fits the project
  - Why it fits learning goals
  - Why it fits available infrastructure
- Complete tech stack breakdown
  - Frontend, backend, database, auth, styling, file storage, deployment, testing
- Learning opportunities (frontend, backend, architecture, transferable skills)
- Cost analysis (setup + monthly costs)
- Alternative options (2-3 alternatives with trade-offs)
- Ruled-out options (with explanations)
- Next steps

**Used by:** deployment-advisor (to understand tech requirements), project-spinup (for scaffolding)

**Format:** Uses structured markdown with clear sections matching the output template in tech-stack-advisor/SKILL.md

**Key Feature:** Created AFTER collaborative discussion, not before. Includes infrastructure evaluation (Supabase vs PocketBase decision framework, ancillary tools like n8n/Ollama/Wiki.js)

---

### 4. deployment-strategy.md (Phase 2)

**Created by:** deployment-advisor (AFTER collaborative refinement and user convergence)

**Location:** `.docs/deployment-strategy.md`

**Purpose:** Complete hosting and deployment strategy with concrete workflows serving as session bridge

**Contains:**
- Primary hosting recommendation
  - Why it fits the project
  - Why it fits the infrastructure
- Hosting details (provider, server type, container approach, database, file storage, CDN, SSL, DNS)
- Deployment workflow
  - Initial setup (one-time steps)
  - Regular deployment (for updates)
  - Rollback procedure
- Cost breakdown (setup + monthly + scaling costs)
- Scaling path (multiple phases with triggers)
- Alternative hosting options (with trade-offs)
- Monitoring & maintenance (setup, schedule, tools)
- Backup strategy (database, files, configuration, verification, disaster recovery)
- Security considerations
- Next steps

**Used by:** project-spinup (for deployment integration in claude.md and README), deploy-guide (for actual deployment), ci-cd-implement (for pipeline generation)

**Format:** Uses structured markdown with clear sections matching the output template in deployment-advisor/SKILL.md

**Key Feature:** Created AFTER collaborative discussion, not before. Evaluates 5 deployment options (Localhost, Hostinger Shared, Cloudflare Pages, Fly.io, VPS with Docker) with honest infrastructure trade-off analysis. Localhost option enables early termination for learning projects.

---

### 5. claude.md + Project Foundation (Phase 3)

**Created by:** project-spinup

**Location:** `[project-dir]/claude.md`

**Purpose:** Final planning handoff document serving as complete project context for all future Claude Code development sessions

**Contains:**
- Developer profile (John's context, experience level, development approach)
- Project overview (description, tech stack, architecture decisions)
- Development environment (setup, tools, workflow)
- Infrastructure & hosting (from deployment-strategy.md, self-hosted tools integration)
- Development workflow (git branching strategy, conventional commits, testing, deployment)
- Code conventions (structure, naming, style)
- Common commands (dev, docker, database)
- Project-specific notes (auth, schema, APIs, environment variables)
- Deployment (workflow from deployment-strategy.md, checklist, rollback)
- Resources & references
- Troubleshooting
- Next steps (Guided Setup prompts OR Quick Start instructions based on spinup_approach)

**Used by:** All future Claude Code sessions working on the project

**Also creates:**
- docker-compose.yml (local development environment)
- Directory structure (src/, tests/, docs/)
- .gitignore (tech-stack-appropriate)
- README.md (setup and development instructions)
- .env.example (required environment variables)
- Git initialization guidance
- *If Quick Start:* Complete project scaffolding with starter code

**Key Feature:** Separates spinup approach (Guided/Quick Start - tactical implementation familiarity) from PROJECT-MODE (strategic learning). Loads ALL handoff documents to preserve complete context chain. This is a natural termination point for localhost/learning projects.

---

### 6. test-strategy.md (Phase 4)

**Created by:** test-orchestrator

**Location:** `.docs/test-strategy.md`

**Purpose:** Document test strategy decisions and serve as reference for ongoing testing

**Contains:**
- Testing philosophy and approach
- Test types implemented (unit, integration, e2e)
- Test file organization
- Coverage goals
- Test commands and scripts
- CI integration notes
- Guidance for writing new tests

**Used by:** Development sessions (reference), ci-cd-implement (for CI pipeline)

**Key Feature:** Serves primarily as setup + guidance skill. Created when project is ready for testing infrastructure, not required before development.

---

### 7. deployment-log.md (Phase 5)

**Created by:** deploy-guide

**Location:** `.docs/deployment-log.md`

**Purpose:** Record of deployment actions taken and ongoing deployment reference

**Contains:**
- Deployment date and target
- Pre-deployment checklist completed
- Deployment steps executed
- Post-deployment verification
- Environment URLs
- Credentials/secrets notes (references, not values)
- Rollback procedure
- Lessons learned
- Next deployment notes

**Used by:** Future deployments (reference), ci-cd-implement (for automation)

**Key Feature:** Living document updated with each deployment. Serves as both record and runbook. This is a termination point for projects that don't need CI/CD automation.

---

### 8. CI/CD Outputs (Phase 6)

**Created by:** ci-cd-implement

**Location:** `.github/workflows/`, `scripts/`, `CICD-SECRETS.md`

**Purpose:** Automated pipelines for testing and deployment

**Contains:**
- `.github/workflows/ci.yml` - Continuous Integration (test, lint, build)
- `.github/workflows/deploy.yml` - Continuous Deployment
- `scripts/deploy.sh` - Manual deployment script
- `scripts/rollback.sh` - Rollback script
- `CICD-SECRETS.md` - Secrets documentation

**Used by:** GitHub Actions (automated execution)

**Key Feature:** Final workflow skill. Analyzes project structure and deployment-strategy.md to generate appropriate pipelines. This is the final termination point.

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

**Key principle:** Skills nudge for missing info conversationally, never block.

### Example: Invoking deploy-guide without deployment-strategy.md

```
User: "Use deploy-guide skill"

Claude: "I don't see .docs/deployment-strategy.md. No problem!

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
5. **Created after convergence**: Documents are written AFTER collaborative refinement with the user, not before

### When Are Handoff Documents Created?

**Critical timing rule:** Handoff files are created **after the user has discussed and confirmed the primary recommendation** in each skill phase.

**The workflow for advisory skills (Phase 0-2):**
1. Skill presents recommendations in console (primary + alternatives + ruled-out)
2. User asks questions, explores alternatives, discusses trade-offs
3. User confirms choice or converges on final decision
4. **THEN** skill creates handoff document with complete analysis
5. Skill confirms document location and next steps

**Why this timing matters:**
- Ensures document reflects actual decision, not preliminary recommendation
- Captures any modifications made during discussion
- Prevents premature file creation that might need rewriting
- Guarantees accuracy of handoff to next skill

---

## File Organization

All handoff documents go in `.docs/` subdirectory:

```
project-workspace/
├── .docs/                              # All handoff documents
│   ├── PROJECT-MODE.md                 # Phase 0 output
│   ├── brief-my-project.md             # Phase 0 output
│   ├── tech-stack-decision.md          # Phase 1 output
│   ├── deployment-strategy.md          # Phase 2 output
│   ├── test-strategy.md                # Phase 4 output
│   └── deployment-log.md               # Phase 5 output
├── my-project/                         # Phase 3 output (if separate directory)
│   ├── claude.md                       # Project context
│   ├── docker-compose.yml
│   ├── README.md
│   ├── .env.example
│   ├── .gitignore
│   ├── src/
│   ├── tests/
│   └── docs/
├── .github/                            # Phase 6 output
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
├── scripts/                            # Phase 6 output
│   ├── deploy.sh
│   └── rollback.sh
└── CICD-SECRETS.md                     # Phase 6 output
```

**Note:** The `.docs/` directory keeps handoff documents organized and separate from project source code. This directory can be excluded from version control if desired.

---

## Validation Checklist

To verify your handoff chain is complete at each phase:

**After Phase 0 (project-brief-writer):**
- [ ] .docs/PROJECT-MODE.md exists and declares mode (LEARNING/BALANCED/DELIVERY)
- [ ] .docs/brief-[name].md exists and is problem-focused (technology-agnostic)
- [ ] Brief includes deployment intent (localhost/public/TBD)
- [ ] Brief includes quarantined tech thoughts section (if user mentioned technologies)

**After Phase 1 (tech-stack-advisor):**
- [ ] .docs/tech-stack-decision.md exists with complete recommendation
- [ ] Includes primary recommendation, alternatives, and ruled-out options
- [ ] Infrastructure evaluation present (Supabase vs PocketBase, ancillary tools)
- [ ] Created AFTER collaborative discussion (not before)

**After Phase 2 (deployment-advisor):**
- [ ] .docs/deployment-strategy.md exists with complete strategy
- [ ] Includes deployment workflow, cost breakdown, scaling path
- [ ] Evaluates deployment options appropriate for project
- [ ] Created AFTER collaborative discussion (not before)

**After Phase 3 (project-spinup):**
- [ ] Project directory exists with claude.md and foundation files
- [ ] docker-compose.yml configured for local development
- [ ] README.md has setup instructions
- [ ] .env.example lists all required environment variables
- [ ] Git initialization guidance provided
- [ ] Next steps clear (Guided Setup prompts OR Quick Start ready to run)
- [ ] **TERMINATION CHECK:** If localhost-only project, workflow complete here

**After Phase 4 (test-orchestrator) - Optional:**
- [ ] .docs/test-strategy.md exists with testing approach
- [ ] Test scaffolding created (test files, configs)
- [ ] Test commands documented

**After Phase 5 (deploy-guide):**
- [ ] Application deployed to target environment
- [ ] .docs/deployment-log.md created with deployment record
- [ ] Post-deployment verification complete
- [ ] **TERMINATION CHECK:** If no CI/CD needed, workflow complete here

**After Phase 6 (ci-cd-implement):**
- [ ] .github/workflows/ contains CI and/or CD workflows
- [ ] CICD-SECRETS.md documents required secrets
- [ ] Pipeline tested with initial push
- [ ] **WORKFLOW COMPLETE**

---

## Revising Decisions

If you need to change a decision made in a previous phase, follow these safe revision procedures:

### Revising Tech Stack (After Phase 1)

**Scenario:** You've completed tech-stack-advisor but want to explore a different stack.

**Procedure:**
1. Delete `.docs/tech-stack-decision.md` and `.docs/deployment-strategy.md` (if it exists)
2. Re-invoke tech-stack-advisor skill
3. Make new tech stack decision
4. Proceed with deployment-advisor (it will read new tech stack decision)

**Why delete deployment-strategy.md?** Deployment strategy depends on tech stack. Deleting ensures consistency.

---

### Revising Deployment Strategy (After Phase 2)

**Scenario:** You've completed deployment-advisor but want to use a different hosting approach.

**Procedure:**
1. Delete `.docs/deployment-strategy.md`
2. Re-invoke deployment-advisor skill
3. Make new deployment decision
4. Proceed with project-spinup (it will read new deployment strategy)

**No need to delete tech-stack-decision.md** - tech stack choice is still valid.

---

### Revising Project Mode (Any Phase)

**Scenario:** You started in LEARNING mode but want to switch to BALANCED or DELIVERY.

**Procedure:**
1. Edit `.docs/PROJECT-MODE.md` directly
2. Change the mode line: `**Mode:** BALANCED` (or DELIVERY)
3. Update the "Mode Details" section to match
4. Save the file
5. Continue with next skill (will read updated mode)

**Alternative:** Delete `.docs/PROJECT-MODE.md` and re-run project-brief-writer (will recreate with new mode choice).

---

### Starting Over Completely

**Scenario:** You want to completely restart the workflow for this project.

**Procedure:**
1. Delete entire `.docs/` directory
2. Re-invoke project-brief-writer skill
3. Proceed through all phases again

**Note:** Project foundation directory (if created) is independent - you can keep or delete it as needed.

---

## Key Concepts

### Strategic Learning vs Tactical Learning

The workflow separates two types of learning:

**Strategic Learning (PROJECT-MODE.md):**
- Governs advisory skills (tech-stack-advisor, deployment-advisor)
- Determines checkpoint strictness and exploration depth
- About understanding trade-offs, evaluating options, making informed decisions
- LEARNING mode = detailed checkpoints, BALANCED mode = moderate checkpoints, DELIVERY mode = minimal checkpoints

**Tactical Learning (spinup_approach in project-spinup):**
- Governs implementation scaffolding style
- Determines whether to build incrementally (Guided Setup) or all at once (Quick Start)
- About familiarity with the chosen tech stack
- Independent of PROJECT-MODE - can be in LEARNING mode but use Quick Start if familiar with stack

### Flexible Entry

Skills don't require strict prerequisite enforcement. If handoff documents are missing, skills gather equivalent information conversationally. This enables:
- Starting at any phase when context is clear
- Using skills standalone outside the workflow
- Recovering from incomplete workflows

### Termination Points

The workflow has three natural stopping points:
1. **After project-spinup** - For localhost/learning projects
2. **After deploy-guide** - For manually deployed projects
3. **After ci-cd-implement** - For fully automated projects

Users choose their stopping point based on project needs, not workflow prescription.

### Collaborative Refinement

Handoff documents are created AFTER collaborative discussion with the user, not before. This ensures:
- User convergence on recommendations
- Opportunity to explore alternatives
- Questions answered before documenting
- Accurate representation of final decision

### Infrastructure as Context, Not Constraint

Skills evaluate self-hosted infrastructure (Supabase, PocketBase, n8n, Ollama, Wiki.js on VPS) as context in trade-off analysis, not as a constraint:
- Marginal cost is $0 if infrastructure already running
- But skills honestly recommend cloud alternatives when genuinely better
- Framework: Discover, Evaluate, Analyze, Recommend, Explain, Empower

---

## Version History

### v3.0 (2025-11-22)
**Major restructure for refined workflow**

- Updated to 7 phases (0-6) with 8 skills
- Added three termination points (after spinup, deploy-guide, ci-cd-implement)
- Changed handoff document location to `.docs/` subdirectory
- Added new handoff documents: test-strategy.md, deployment-log.md
- Added flexible entry pattern documentation
- Added new skills: test-orchestrator (Phase 4), deploy-guide (Phase 5)
- Updated ci-cd-implement positioning (now Phase 6)
- Added deployment intent to project brief
- Updated file organization to reflect `.docs/` structure

### v2.1 (2025-01-17)
**v1.0-ready refinements based on comprehensive review**

- Reduced checkpoint questions from 5 to 3 in LEARNING mode
- Added "When Are Handoff Documents Created?" section
- Added comprehensive "Revising Decisions" section

### v2.0 (2025-01-17)
**Major update aligned with current SKILL.md implementations**

- Updated to reflect Phase 0-3 structure (4 phases total)
- Documented handoff document creation timing
- Added key concepts section

### v1.0 (2025-11-12)
**Initial handoff documentation system**

- Established handoff chain pattern
- Created session bridge pattern for workflow continuity

---

## Related Documentation

- **skills/project-brief-writer/SKILL.md** - How PROJECT-MODE.md is created
- **skills/tech-stack-advisor/SKILL.md** - How tech-stack-decision.md is created
- **skills/deployment-advisor/SKILL.md** - How deployment-strategy.md is created
- **skills/project-spinup/SKILL.md** - How all handoffs are consumed to create claude.md
- **skills/test-orchestrator/SKILL.md** - How test-strategy.md is created
- **skills/deploy-guide/SKILL.md** - How deployment-log.md is created
- **skills/ci-cd-implement/SKILL.md** - How CI/CD pipelines are generated
- **skills/workflow-status/SKILL.md** - How to check workflow progress
