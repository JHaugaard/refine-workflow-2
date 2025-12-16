# ci-cd-implement Skill

## Overview

Analyzes an existing project and generates production-ready CI/CD pipelines tailored to its tech stack and deployment target. Works as a standalone utility on any project—does not require prior workflow skills to have run.

**Use when:** A project is mature enough for automated testing and/or deployment

**Output:** GitHub Actions workflows, deployment scripts, and secrets documentation

---

## How It Works

When invoked, this skill will:

1. **Analyze project** - Scan for tech stack, test commands, build steps, and deployment target
2. **Gather preferences** - Ask whether to generate CI only, CD only, or both
3. **Determine complexity** - Assess if project needs staging+production or just production
4. **Generate CI** - Create GitHub Actions workflow for testing, linting, and building
5. **Generate CD** - Create deployment workflow and scripts for the target platform
6. **Document secrets** - Create CICD-SECRETS.md with all required secrets and setup instructions
7. **Summarize** - Present what was created with clear next steps

---

## Supported Deployment Targets

| Target | Description | Method |
|--------|-------------|--------|
| cloudflare-pages | Static/JAMstack sites | Wrangler CLI |
| fly-io | Containerized applications | flyctl |
| vps-docker | Docker on VPS | SSH + docker compose |
| hostinger-shared | PHP applications | rsync/FTP |

**Not supported:** localhost (CI/CD not applicable)

---

## Generated Files

Depending on preferences, the skill generates:

- `.github/workflows/ci.yml` - CI pipeline (test, lint, type-check, build)
- `.github/workflows/deploy.yml` - CD pipeline for deployment target
- `scripts/deploy.sh` - Manual deployment script (VPS targets)
- `scripts/rollback.sh` - Rollback script (VPS targets)
- `CICD-SECRETS.md` - Secrets documentation with setup instructions

---

## Version History

### v1.0 (November 2025)
**Initial Release**

- Project analysis for tech stack and deployment target detection
- CI pipeline generation with language-specific setup
- CD pipeline generation for 4 deployment targets
- Complexity-based environment strategy (production only vs staging+production)
- Secrets documentation with target-specific setup instructions
- VPS deployment and rollback scripts

---

**Version:** 1.0
**Last Updated:** November 2025
