# deployment-advisor Skill

## Overview

Recommend hosting strategy based on chosen tech stack and project needs. Provides deployment workflow, cost breakdown, and scaling path. Considers Hostinger infrastructure, budget, and complexity tolerance.

**Use when:** After tech-stack-advisor has recommended a tech stack, before invoking project-spinup skill, when evaluating hosting options for existing projects, or when comparing self-hosted vs managed services.

**Don't use when:** Before deciding on tech stack (tech stack drives hosting choice), for quick local-only prototypes, or when hosting is mandated by client/organization.

**Output:** deployment-strategy.md file containing complete hosting plan with deployment workflow, cost breakdown, scaling path, monitoring, backup strategy, and security considerations.

---

## How It Works

When invoked, this skill will:

1. **Check PROJECT-MODE.md** - Determine checkpoint strictness level
2. **Gather deployment information** - Tech stack, traffic, uptime, budget
3. **Consider user context** - Infrastructure, preferences, skills
4. **Generate recommendation** - Primary, alternatives, workflows, costs
5. **Create handoff document** - deployment-strategy.md for project-spinup
6. **Run checkpoint** - Validate understanding based on MODE

---

## Skills Workflow Integration

This skill is **Phase 2 of 3** in the learning-focused project workflow:

```
project-brief-writer (Phase 0)
    ↓
tech-stack-advisor (Phase 1)
    ↓
deployment-advisor ← YOU ARE HERE
    ↓
project-spinup (Phase 3)
```

---

## Five Deployment Options

This skill evaluates five deployment options relevant to small personal projects:

### 1. Localhost
- **Best For:** Personal utilities, learning, testing
- **Cost:** $0
- **Tech Stacks:** Any

### 2. Hostinger Shared Hosting
- **Best For:** Simple PHP sites, WordPress
- **Cost:** $0 marginal if account exists
- **Tech Stacks:** PHP + MySQL only

### 3. Cloudflare Pages
- **Best For:** Static sites, SPAs, JAMstack
- **Cost:** $0 (genuinely free)
- **Tech Stacks:** Any static generator, React, Vue, Next.js (static)

### 4. Fly.io
- **Best For:** Full-stack with database, global distribution
- **Cost:** $20-50/month
- **Tech Stacks:** Any Docker-based

### 5. VPS with Docker (Hostinger)
- **Best For:** Full control, self-hosted infrastructure, learning DevOps
- **Cost:** $0 marginal (already have VPS)
- **Tech Stacks:** Any

**Key Principle:** Code is portable. Deployment target is a separate decision.

---

## Checkpoint System

### LEARNING Mode
Answer 3 focused comprehension questions about deployment needs, trade-offs, and maintenance responsibilities.

### BALANCED Mode
Simple self-assessment checklist - confirm to proceed.

### DELIVERY Mode
Quick acknowledgment: "Ready to proceed? [Yes/No]"

---

## Decision Framework

### Choose VPS with Docker When:
- Tech stack runs well in Docker
- Traffic <10k daily users
- Want to learn DevOps
- Budget-conscious (marginal cost $0)
- Need maximum control, self-host tools

### Choose Cloudflare Pages When:
- Frontend-only application
- Static or JAMstack
- Want zero cost
- Need global CDN performance

### Choose Fly.io When:
- Full-stack with database
- Need global distribution
- Comfortable with Docker
- Budget $20-50/month acceptable

### Choose Shared Hosting When:
- Simple PHP application
- Want minimal maintenance
- Have existing shared hosting account

---

## Common Deployment Patterns

- **Static/JAMstack:** Cloudflare Pages, $0/month
- **Full-Stack Self-Hosted:** VPS + Docker, $0/month marginal
- **Full-Stack Global:** Fly.io, $20-50/month
- **Hybrid Frontend + Backend:** Pages + VPS, $0/month marginal
- **Simple PHP:** Shared Hosting, $3-5/month

---

## Advisory Mode

This is a CONSULTANT role, not a BUILDER role:
- Will NOT configure servers or infrastructure
- Will NOT create deployment scripts or automation
- Will NOT set up CI/CD pipelines
- CAN write reference documentation when explicitly requested

---

## Related Skills

- **project-brief-writer** - Creates PROJECT-MODE.md (Phase 0)
- **tech-stack-advisor** - Provides tech stack decisions that inform hosting (prerequisite)
- **project-spinup** - Scaffolds project with deployment configuration (next step)

---

## Version History

### v1.4 (2025-01-17)
- Reduced LEARNING mode checkpoint questions from 5 to 3
- 40% reduction in checkpoint burden

### v1.3 (2025-11-17)
- Updated infrastructure list with VPS8 specs, Caddy, PocketBase
- Revised localhost and shared hosting positioning

### v1.2 (2025-11-12)
- Aligned with canonical 5 deployment options
- Removed non-standard options (Vercel, Netlify, Railway, etc.)

### v1.1 (2025-11-11)
- Added 3-level checkpoint system
- Integrated PROJECT-MODE.md awareness
- Added self-hosted infrastructure evaluation framework

### v1.0 (2025-11-04)
Initial release with hosting recommendation framework.

---

**Version:** 1.4
**Last Updated:** 2025-01-17
