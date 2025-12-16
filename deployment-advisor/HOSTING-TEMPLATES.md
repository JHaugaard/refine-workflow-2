# Hosting Templates Reference

> **Purpose**: Detailed hosting options, deployment patterns, and output templates for deployment-advisor.
> Referenced by: SKILL.md Phase 3 (Analyze & Recommend)

---

## Hosting Options

### Localhost

| Attribute | Value |
|-----------|-------|
| **Description** | Running application on personal computer, not publicly accessible |
| **Best For** | Personal utility apps, testing before deployment, development tools |
| **Tech Stacks** | Any |
| **Pros** | No infrastructure to manage, immediate feedback, full control, any database locally, maximum privacy |
| **Cons** | Not accessible outside network, dependent on computer being on, no global distribution |
| **When to Recommend** | Personal-use only apps, development utilities, testing environments |
| **Vendor Lock-in** | NONE |
| **Workflow Note** | TERMINATION POINT - workflow complete after project-spinup |

---

### Hostinger Shared Hosting

| Attribute | Value |
|-----------|-------|
| **Description** | Traditional web hosting with cPanel, PHP/MySQL stack |
| **Best For** | Simple PHP sites, WordPress, static websites |
| **Tech Stacks** | PHP + MySQL only |
| **Pros** | Managed infrastructure, cPanel interface, minimal maintenance |
| **Cons** | Limited to PHP + MySQL, no Docker, shared resources, single region |
| **When to Recommend** | Simple PHP apps, WordPress, existing shared hosting account |
| **Vendor Lock-in** | LOW - standard PHP |

---

### Cloudflare Pages

| Attribute | Value |
|-----------|-------|
| **Description** | Serverless platform for static sites and JAMstack on global edge network |
| **Best For** | Static sites, SPAs, JAMstack, frontend-only projects |
| **Tech Stacks** | Any static generator, React, Vue, Next.js (static), Svelte, Astro |
| **Pros** | Zero-config deployment, automatic HTTPS, global CDN, git-connected auto-deploy |
| **Cons** | Static/frontend only, no persistent backend, 100MB project limit |
| **When to Recommend** | Frontend-only app, need global performance, already using Cloudflare DNS |
| **Vendor Lock-in** | LOW - vanilla HTML/CSS/JS, portable |

---

### Fly.io

| Attribute | Value |
|-----------|-------|
| **Description** | Platform for containerized applications globally across 35 regions with managed PostgreSQL |
| **Best For** | Full-stack with database, global distribution, managed infrastructure |
| **Tech Stacks** | Any Docker-based |
| **Pros** | Docker-based, 35 global regions, auto-scaling, managed PostgreSQL, CLI-first |
| **Cons** | Requires Docker, pay-per-use pricing |
| **When to Recommend** | Full-stack needing database, global distribution |
| **Vendor Lock-in** | LOW - standard Docker containers portable anywhere |

---

### VPS with Docker

| Attribute | Value |
|-----------|-------|
| **Description** | Virtual Private Server running Docker containers for complete control |
| **Best For** | Full-stack with custom requirements, self-hosted infrastructure |
| **Tech Stacks** | Any |
| **Pros** | Full control, use existing infrastructure, predictable resources, run any database |
| **Cons** | More maintenance, manage security updates, single point of failure |
| **When to Recommend** | Already have VPS with capacity, tech stack runs in Docker, traffic <10k daily, need maximum control |
| **Vendor Lock-in** | NONE - standard Docker containers, move anywhere |

---

## Decision Logic

### Recommend Localhost When

- Personal utility app for own use only
- Development tools and scripts
- Testing before public deployment
- Privacy-sensitive personal data

### Recommend Shared Hosting When

- Simple PHP-based application
- Traditional LAMP stack preferred
- Minimal maintenance desired
- Want to use existing shared hosting

### Recommend Cloudflare Pages When

- Frontend-only application
- Static or JAMstack architecture
- Need global CDN performance

### Recommend Fly.io When

- Full-stack app with database
- Need global distribution
- Want managed infrastructure

### Recommend VPS Docker When

- Tech stack runs well in Docker
- Traffic low-to-moderate (<10k daily)
- Need maximum control
- Want to self-host tools

---

## Deployment Patterns

### Pattern: Localhost Development

```
Option: Localhost
Stack: Any
Database: Local PostgreSQL/SQLite/MySQL
Deployment: docker compose up (or native)
Termination: Workflow complete after project-spinup
```

### Pattern: Static JAMstack

```
Option: Cloudflare Pages
Stack: React, Vue, Svelte, Astro, Next.js (static)
Database: External API or none
Deployment: Git push (auto-deploy)
```

### Pattern: Full-Stack Self-Hosted

```
Option: VPS with Docker
Stack: Next.js, FastAPI, PHP, Node.js
Database: Self-hosted PostgreSQL/MySQL
Deployment: Git push -> SSH -> docker compose up
```

### Pattern: Full-Stack Global

```
Option: Fly.io
Stack: Next.js, FastAPI, Node.js, Go
Database: Fly.io managed PostgreSQL
Deployment: fly deploy
```

### Pattern: Hybrid Frontend/Backend

```
Frontend: Cloudflare Pages
Backend API: VPS with Docker
Database: Self-hosted PostgreSQL
Best for: Fast frontend, controlled backend
```

### Pattern: Simple PHP

```
Option: Hostinger Shared Hosting
Stack: PHP + MySQL
Deployment: FTP or Git
```

---

## Scoring Criteria

When evaluating options, rate each 1-5:

| Criterion | What to Evaluate |
|-----------|------------------|
| **Capability Fit** | Can handle chosen stack? Required services available? |
| **Performance Requirements** | Handle expected traffic? Acceptable latency? |
| **Ecosystem/Community** | Good documentation? Active maintenance? |
| **Scalability Path** | Can it grow with the project if needed? |

---

## Output Templates

### Standard Recommendation Template

```markdown
## PRIMARY HOSTING RECOMMENDATION: {Hosting Approach}

{2-3 sentence summary}

### Why This Fits Your Project

- {Reason 1 - capability fit}
- {Reason 2 - traffic/performance match}
- {Reason 3 - ecosystem/community}

### Why This Fits Your Infrastructure

- {How it uses existing VPS}
- {Integration with current setup}

### Hosting Details

| Aspect | Value |
|--------|-------|
| Provider | {Hostinger VPS / Shared / Localhost / Other} |
| Server Type | {VPS / Shared / PaaS / Serverless / Local} |
| Container | {Docker / Native / Platform-managed} |
| Database | {Self-hosted / Managed / Local} |
| File Storage | {Local VPS / Backblaze B2 / Local disk} |
| CDN | {Cloudflare / Provider CDN / None} |
| SSL | {Caddy / Let's Encrypt / Cloudflare / None} |

### Deployment Workflow

**Initial Setup (One-time):**
{steps}

**Regular Deployment (Updates):**
{steps}

**Rollback Procedure:**
{steps}

---

## Scaling Path

**Current State:** {recommendation}
**Phase 1 Optimization:** {when and what}
**Phase 2 Scaling:** {when and what}

**When to Scale:**
- Phase 1: {specific metric}
- Phase 2: {specific metric}

---

## Alternative Hosting Options

### Alternative 1: {approach}

**Pros:** {list}
**Cons:** {list}
**When to Choose:** {conditions}

---

## Monitoring & Maintenance

**Daily:** {tasks}
**Weekly:** {tasks}
**Monthly:** {tasks}

---

## Backup Strategy

**Database Backups:** {frequency, method, storage, retention}
**File Storage Backups:** {frequency, method}
**Configuration Backups:** {method}
**Disaster Recovery:** {procedure, estimated time}

---

## Security Considerations

**Implemented:** {list}
**Recommended Additions:** {list}

---

## Next Steps

**Handoff document created:** .docs/deployment-strategy.md

[If Localhost:]
1. Proceed to project-spinup to generate your project foundation
2. After project-spinup, your localhost project is ready for development
3. **WORKFLOW TERMINATION POINT** - no further phases needed
4. If you later decide to deploy publicly, re-run deployment-advisor

[If Public Deployment:]
1. Review and ask questions
2. If agreed -> Invoke project-spinup skill
3. Build your features
4. When ready -> Use deploy-guide to deploy
5. Optional -> Use ci-cd-implement for automation
```

### Localhost-Specific Template

```markdown
## PRIMARY HOSTING RECOMMENDATION: Localhost

This project will run locally on your development machine.

### Why Localhost

- Personal utility project - no public access needed
- Full control over environment
- Privacy for personal data
- Immediate development feedback

### Local Development Setup

| Aspect | Value |
|--------|-------|
| Environment | Local machine (Mac) |
| Container | Docker Compose (recommended) or native |
| Database | Local {database type} |
| Storage | Local filesystem |
| Access | localhost only |

### Development Workflow

**Start Development:**
docker compose up -d  # or native start commands

**Access Application:**
- Frontend: http://localhost:3000
- Backend: http://localhost:8000
- Database: localhost:5432

### Future Public Deployment
If you later decide to deploy publicly:
1. Re-run deployment-advisor
2. Choose a public hosting option
3. Continue with deploy-guide and ci-cd-implement

---

## Next Steps

**Handoff document created:** .docs/deployment-strategy.md

1. Proceed to project-spinup to generate your project foundation
2. After project-spinup, start building your features
3. **WORKFLOW TERMINATION POINT** - no hosting phases needed

Your localhost project workflow is: project-brief-writer -> tech-stack-advisor -> deployment-advisor -> project-spinup -> DONE
```

---

## Native Platform Handoff

When tech-stack-advisor recommends a native platform (iOS, macOS, or Tauri), deployment-advisor uses a short-circuit path instead of the full hosting workflow.

### Why Short-Circuit?

Native platforms don't need server hosting:
- **iOS/macOS**: Apps run on user's device, deployed via Xcode
- **Tauri**: Desktop apps run locally as binaries

The "deployment" for these platforms is project setup (project-spinup) and device installation (deploy-guide), not hosting strategy.

### Detection

Check `tech-stack-decision.json` for:
```json
"platform": {
  "primary": "ios" | "macos" | "tauri"
}
```

If detected → skip to native platform handoff.

### Minimal Handoff Schema

```json
{
  "document_type": "deployment-strategy",
  "version": "1.0",
  "created": "[YYYY-MM-DD]",
  "project": "[project-name]",
  "mode": "[MODE]",

  "native_platform": {
    "platform": "ios | macos | tauri",
    "hosting_required": false,
    "note": "Native platform — no server hosting needed. Runs on user device."
  },

  "handoff_to": ["project-spinup"],
  "termination_point": "project-spinup"
}
```

### Response Template

```markdown
Your project targets **{platform}** — a native platform that runs on your device rather than a server.

This means you don't need a hosting strategy. Instead:
- **project-spinup** will scaffold your {platform} project with the right build configuration
- **deploy-guide** can walk you through installing on your device (if needed)

Ready to proceed to **project-spinup**?
```

### Workflow After Handoff

```
deployment-advisor (short-circuit) → project-spinup → DONE
                                                    ↓
                                        (optional) deploy-guide for device installation
```

---

*Reference document for deployment-advisor skill*
*Created: 2025-12-09*
*Updated: 2025-12-12 — Added native platform handoff pattern*
