# deploy-guide Skill

## Overview

Guide you step-by-step through deploying your application to production. This skill performs pre-deployment checks, executes deployment commands with your confirmation, verifies success, and creates deployment documentation for future reference.

**Use when:** Your project is ready to deploy to production (or you're doing a subsequent deployment).

**Output:**
- Deployed application
- `.docs/deployment-log.md` (deployment record and runbook)
- Post-deployment verification results

---

## How It Works

When invoked, this skill will:

1. **Gather Context** - Read deployment strategy or ask about your target
2. **Pre-Deployment Checks** - Verify code is committed, build passes, config is ready
3. **Deploy** - Execute deployment steps with your confirmation at each step
4. **Post-Deployment Verification** - Check application is accessible and working
5. **Create Deployment Log** - Document the deployment for future reference

---

## Supported Deployment Targets

| Target | Description | Best For |
|--------|-------------|----------|
| VPS with Docker | Docker Compose deployment via SSH | Full-stack apps, self-hosted infrastructure |
| Cloudflare Pages | Static/JAMstack on global CDN | Frontend-only, static sites |
| Fly.io | Containerized apps with managed PostgreSQL | Full-stack with global distribution |
| Hostinger Shared | PHP + MySQL via FTP/rsync | Simple PHP applications |

---

## Workflow Position

This is **Phase 5** in the Skills workflow - a termination point for projects that don't need CI/CD automation.

```
Phase 0: project-brief-writer
Phase 1: tech-stack-advisor
Phase 2: deployment-advisor
Phase 3: project-spinup         <- TERMINATION POINT (localhost)
Phase 4: test-orchestrator      <- optional
Phase 5: deploy-guide           <- YOU ARE HERE (TERMINATION POINT)
Phase 6: ci-cd-implement        <- TERMINATION POINT (full automation)
```

After deploy-guide, you can:
- **Stop here** - Use `.docs/deployment-log.md` for future manual deployments
- **Continue** - Use `ci-cd-implement` to automate deployments

---

## Flexible Entry

This skill can be invoked standalone on any deployable project. It gathers deployment target information conversationally if `.docs/deployment-strategy.md` is not available.

---

## Deployment Log

The skill creates `.docs/deployment-log.md` containing:
- Deployment record (date, commit, status)
- Deployment runbook for future deployments
- Rollback procedure
- Environment variables reference
- Troubleshooting guide

This serves as both a record and a runbook for subsequent deployments.

---

## Version History

### v1.0 (2025-11-22)
**Initial Release**

- Pre-deployment verification checklist
- Step-by-step deployment with user confirmation
- Support for 4 deployment targets (VPS/Docker, Cloudflare Pages, Fly.io, Hostinger Shared)
- Post-deployment verification
- Deployment log creation
- Troubleshooting guidance for common issues
- First-deployment setup instructions for each target

---

**Version:** 1.0
**Last Updated:** 2025-11-22
