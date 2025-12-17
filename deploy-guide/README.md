# deploy-guide Skill

## Overview

Guide you step-by-step through deploying your application to production. This skill performs pre-deployment checks, executes deployment commands with your confirmation, verifies success, and creates deployment documentation for future reference.

**Use when:** Your project is ready to deploy to production (or you're doing a subsequent deployment).

**Output:**

- Deployed application
- `.docs/deployment-log.json` (deployment record and runbook)
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

## Standalone Capability

This skill can be invoked independently on any deployable project. If `.docs/deployment-strategy.json` is not available, the skill gathers deployment target information conversationally.

If using the workflow system, invoke `workflow-status` to see recommended next steps.

---

## Supported Deployment Targets

| Target | Description | Best For |
|--------|-------------|----------|
| VPS with Docker | Docker Compose deployment via SSH | Full-stack apps, self-hosted infrastructure |
| Cloudflare Pages | Static/JAMstack on global CDN | Frontend-only, static sites |
| Fly.io | Containerized apps with managed PostgreSQL | Full-stack with global distribution |
| Hostinger Shared | PHP + MySQL via FTP/rsync | Simple PHP applications |

---

## Deployment Log

The skill creates `.docs/deployment-log.json` containing:

- Deployment record (date, commit, status)
- Deployment runbook for future deployments
- Rollback procedure
- Environment variables reference
- Troubleshooting guide

This serves as both a record and a runbook for subsequent deployments.

---

## Output Files

| File | Purpose |
|------|---------|
| `.docs/deployment-log.json` | Deployment record and runbook for future deployments |

---

## Version History

### v1.1 (2025-12-16)

**Manifest-Based Refactor**

- Removed embedded workflow orchestration
- Skills now focus purely on their domain
- Workflow routing handled by manifest

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

**Version:** 1.1
**Last Updated:** 2025-12-16
