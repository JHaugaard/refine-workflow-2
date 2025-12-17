# solution-architect Skill

## Overview

Resolve architectural ambiguity before tech stack selection. This skill analyzes project briefs to determine greenfield vs brownfield context, platform constraints, integration requirements, scale expectations, and team context.

**Use when:** Your project brief is ambiguous about platform (web vs native vs CLI), project context (new vs extending existing), or has complex integration requirements that would change tech stack recommendations.

**Don't use when:** Platform and context are obvious from the brief, simple projects with clear scope, or when you've already made architectural decisions.

**Output:** `.docs/architecture-context.json` — structured architectural decisions that constrain downstream tech stack and deployment choices.

---

## How It Works

When invoked, this skill will:

1. **Check for context** - Look for `.docs/brief.json` and `.docs/PROJECT-MODE.md`
2. **Analyze architectural dimensions** - Identify which questions need resolution
3. **Ask targeted questions** - Only what's needed to resolve ambiguity
4. **Discuss implications** - Explain how answers affect downstream decisions
5. **Run checkpoint** - Validate understanding based on MODE
6. **Create handoff document** - `.docs/architecture-context.json`

---

## Standalone Capability

This skill can be invoked independently on any project. If `.docs/brief.json` is not available, the skill gathers equivalent information conversationally.

If using the workflow system, invoke `workflow-status` to see recommended next steps.

---

## Architectural Dimensions

### 1. Project Context (Greenfield vs Brownfield)

| Type | Description | Implication |
|------|-------------|-------------|
| Greenfield | Brand new project | Full freedom on tech choices |
| Brownfield | Extending existing system | Constrained by existing stack |
| Migration | Replacing current solution | Data migration considerations |

### 2. Platform Determination

| Platform | Examples | Key Questions |
|----------|----------|---------------|
| Web | Website, web app, SPA | Mobile considerations? |
| iOS | iPhone app, iPad app | Apple-specific features? |
| macOS | Mac app, menubar utility | System integrations? |
| Desktop | Cross-platform desktop | Which OSes? |
| CLI | Command-line tool | Distribution method? |
| API | Backend service | Who consumes it? |

### 3. Integration Landscape

- External APIs and third-party services
- Existing databases and data sources
- Authentication systems
- File systems and storage
- Hardware and sensors

### 4. Scale & Performance

| Profile | Concurrent Users | Tech Implications |
|---------|------------------|-------------------|
| Personal | 1 | Simplest viable solution |
| Team | 2-10 | Simple auth, shared state |
| Organization | 10-100 | Role-based access, audit |
| Public | Unknown | Security critical, scalability |

---

## Advisory Mode

This is a CONSULTANT role, not a BUILDER role:

- Will NOT recommend specific technologies (tech-stack-advisor's job)
- Will NOT recommend hosting platforms (deployment-advisor's job)
- Will NOT make implementation decisions (project-spinup's job)
- CAN surface questions that affect those downstream decisions
- CAN explain architectural implications

---

## Checkpoint System

### LEARNING Mode

Answer questions about architectural implications:
1. What trade-off does your platform choice create?
2. Why might your integration requirements matter for tech stack?
3. What would change if you needed to scale up later?

### BALANCED Mode

Simple checklist - confirm understanding of platform, integrations, and scale.

### DELIVERY Mode

Quick confirmation: "Platform: X | Scale: Y | Type: Z — Look right?"

---

## Output Files

| File | Purpose |
|------|---------|
| `.docs/architecture-context.json` | Structured architectural decisions with rationale |

---

## Version History

### v1.0.0 (2025-12-16)

**Initial Release**

- Architectural dimension analysis (greenfield/brownfield, platform, integrations, scale)
- Targeted questioning based on brief analysis
- Mode-aware checkpoints
- JSON handoff document generation
- Edge case handling (early tech specification, brownfield with unknown stack)

---

**Version:** 1.0.0
**Last Updated:** 2025-12-16
