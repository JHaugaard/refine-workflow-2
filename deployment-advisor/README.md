# deployment-advisor Skill

## Overview

Recommend hosting strategy based on chosen tech stack and project needs. Provides deployment workflow, cost breakdown, and scaling path.

**Use when:** After selecting a tech stack, when evaluating hosting options for existing projects, or when comparing self-hosted vs managed services.

**Don't use when:** Before deciding on tech stack (tech stack drives hosting choice), for quick local-only prototypes, or when hosting is mandated by client/organization.

**Output:** `.docs/deployment-strategy.json` — structured hosting plan with deployment workflow, cost breakdown, scaling path, monitoring, backup strategy, and security considerations.

---

## How It Works

When invoked, this skill will:

1. **Check for context** - Look for `.docs/tech-stack-decision.json` and `.docs/PROJECT-MODE.md`
2. **Gather deployment information** - Tech stack, traffic, uptime, budget
3. **Consider user context** - Infrastructure, preferences, skills
4. **Generate recommendation** - Primary, alternatives, workflows, costs
5. **Create handoff document** - `.docs/deployment-strategy.json`
6. **Run checkpoint** - Validate understanding based on MODE

---

## Standalone Capability

This skill can be invoked independently on any project. If `.docs/tech-stack-decision.json` is not available, the skill gathers equivalent information conversationally.

If using the workflow system, invoke `workflow-status` to see recommended next steps.

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

## Output Files

| File | Purpose |
|------|---------|
| `.docs/deployment-strategy.json` | Structured deployment strategy with hosting, workflow, costs, and scaling path |

---

## Version History

### v1.5 (2025-12-16)

**Manifest-Based Refactor**

- Removed embedded workflow orchestration
- Skills now focus purely on their domain
- Workflow routing handled by manifest

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

**Version:** 1.5
**Last Updated:** 2025-12-16
