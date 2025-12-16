# tech-stack-advisor Skill

## Overview

Analyze project requirements and recommend appropriate technology stacks with detailed rationale. Provides primary recommendation, alternatives, and ruled-out options with explanations.

**Use when:** After brainstorming project goals and features, before invoking project-spinup skill, when comparing multiple tech stack options, or when unsure which framework/language fits best.

**Don't use when:** For trivial projects, when tech stack is mandated, for quick prototypes where stack doesn't matter, or when you've already decided and just need validation.

**Output:** tech-stack-decision.md file containing complete analysis with primary recommendation, alternatives, ruled-out options, learning opportunities, and cost analysis.

---

## How It Works

When invoked, this skill will:

1. **Check PROJECT-MODE.md** - Determine checkpoint strictness level
2. **Gather project information** - Requirements, features, complexity, timeline
3. **Check brief quality** - Detect over-specification that bypasses learning
4. **Consider user context** - Experience and learning goals (deployment-neutral)
5. **Evaluate backend tools** - Supabase vs PocketBase decision framework
6. **Analyze Enterprise vs Hacker spectrum** - Surface philosophy trade-offs
7. **Generate recommendations** - Primary, alternatives, ruled-out options with spectrum labels
8. **Create handoff document** - tech-stack-decision.md for deployment-advisor
9. **Run checkpoint** - Validate understanding based on MODE

---

## Skills Workflow Integration

This skill is **Phase 1 of 3** in the learning-focused project workflow:

```
project-brief-writer (Phase 0)
    ↓
tech-stack-advisor ← YOU ARE HERE
    ↓
deployment-advisor (Phase 2)
    ↓
project-spinup (Phase 3)
```

---

## Checkpoint System

### LEARNING Mode
After recommendations, answer 3 focused comprehension questions:
1. Why does the primary recommendation fit this project's core need?
2. What is the single most important trade-off if you chose Alternative 1 instead?
3. What is the biggest new responsibility or learning challenge this stack introduces?

### BALANCED Mode
Simple self-assessment checklist - confirm to proceed.

### DELIVERY Mode
Quick acknowledgment: "Ready to proceed? [Yes/No]"

---

## Deployment Neutrality

This skill focuses purely on technology choices (languages, frameworks, databases) without considering hosting or infrastructure. This separation ensures unbiased recommendations.

**This skill handles:** Languages, frameworks, libraries, databases, architecture patterns

**Deployment-advisor handles:** Hosting platforms, server specs, infrastructure costs, deployment strategies

**User-stated constraints:** If you explicitly mention deployment preferences (e.g., "must run on shared PHP hosting"), these are documented and passed forward but do NOT bias the tech stack recommendation itself.

---

## Enterprise vs Hacker Spectrum

Every recommendation includes an analysis of where options fall on the Enterprise-to-Hacker spectrum:

| Position | Characteristics | Best For |
|----------|-----------------|----------|
| **Enterprise** | Strong typing, corporate backing, explicit patterns, team scalability | Long-term projects, regulated industries, growing teams |
| **Balanced** | Optional typing, modern frameworks, good defaults with escape hatches | Projects that start small but might grow |
| **Hacker** | Dynamic typing, minimal boilerplate, convention over configuration, rapid iteration | Solo developers, MVPs, learning, personal tools |

This is a philosophy choice, not a quality judgment. The skill surfaces this tension so you can consciously decide where you want to land.

---

## Backend Tool Selection

### Supabase (Preferred Default)
Recommend when: Advanced PostgreSQL features, auth + database + storage + realtime all needed, vector embeddings (pgvector), complex queries, future scaling anticipated.

### PocketBase (Lightweight Alternative)
Recommend when: Authentication is primary need, simple CRUD sufficient, embedded SQLite appropriate, single-binary simplicity valued, small scope.

---

## Common Patterns

- **Content-Heavy Site:** Next.js + Markdown/CMS
- **SaaS Application:** Next.js + Supabase
- **API-First:** FastAPI + PostgreSQL
- **Real-Time Collaboration:** Next.js + Supabase Realtime
- **Learning CRUD Project:** PHP + MySQL + Simple MVC
- **Documentation Site:** Wiki.js (Self-Hosted)

---

## Decision Framework

Score each option 1-5 on:

- **Project Fit:** Feature support, performance potential, scalability path, community health
- **Learning Value:** Transferable skills, industry relevance, conceptual clarity
- **Development Experience:** Speed to MVP, tooling quality, debugging ease, documentation
- **Stack Philosophy:** Where it falls on Enterprise vs Hacker spectrum (noted, not scored)

---

## Advisory Mode

This is a CONSULTANT role, not a BUILDER role:
- Will NOT write production code
- Will NOT generate project scaffolding
- Will NOT create implementation files
- CAN write reference documents when explicitly requested for learning

---

## Related Skills

- **project-brief-writer** - Creates PROJECT-MODE.md that determines checkpoint strictness (prerequisite)
- **deployment-advisor** - Continues advisory workflow with hosting recommendations (next step)
- **project-spinup** - Scaffolds project foundation based on tech stack decisions (final step)

---

## Version History

### v1.4 (2025-11-30)
- **Deployment Neutrality:** Strengthened guardrails to keep tech stack recommendations free from hosting/infrastructure bias
- **Enterprise vs Hacker Framework:** Added spectrum analysis to surface philosophy trade-offs in recommendations
- **User-Stated Constraints:** Added handoff section to capture explicit user deployment preferences without biasing recommendations
- Removed infrastructure-related scoring criteria and output fields
- Updated output template with Enterprise vs Hacker analysis table

### v1.3 (2025-01-17)
- Reduced LEARNING mode checkpoint questions from 5 to 3
- 40% reduction in checkpoint burden while preserving pedagogical value

### v1.2 (2025-11-17)
- Updated infrastructure list with VPS8 specs, Caddy, PocketBase
- Added Backend Tool Selection Framework (Supabase vs PocketBase)
- Added Ancillary Infrastructure Tools section (n8n, Ollama, Wiki.js)

### v1.1 (2025-11-11)
- Added 3-level checkpoint system
- Added brief quality detection
- Integrated self-hosted infrastructure evaluation framework

### v1.0 (2025-11-04)
Initial release with tech stack recommendation framework.

---

**Version:** 1.4
**Last Updated:** 2025-11-30
