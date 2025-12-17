# test-orchestrator Skill

## Overview

Set up testing infrastructure and strategy for a project. This skill analyzes your tech stack and creates appropriate test framework configuration, initial test scaffolding, and documentation of your testing approach.

**Use when:** Your project has initial code and you're ready to add testing infrastructure.

**Output:**

- Test framework configuration files
- Initial test scaffolding (example tests)
- `.docs/test-strategy.md` (testing approach documentation)
- Test scripts in package.json or equivalent

---

## How It Works

When invoked, this skill will:

1. **Analyze Project** - Scan tech stack and existing test setup
2. **Determine Testing Approach** - Recommend framework and test types based on your stack
3. **Configure Framework** - Create configuration files (vitest, jest, pytest, phpunit)
4. **Create Scaffolding** - Generate example unit and integration tests
5. **Document Strategy** - Create `.docs/test-strategy.md` with testing guidance
6. **Provide Guidance** - Educational notes on testing patterns and best practices

---

## Standalone Capability

This skill can be invoked independently on any project. It analyzes the project structure to recommend appropriate testing tools — no prior workflow context required.

If using the workflow system, invoke `workflow-status` to see recommended next steps.

---

## Supported Tech Stacks

| Stack | Test Framework | Test Types |
|-------|---------------|------------|
| Next.js / React | Vitest or Jest + React Testing Library | Unit, Integration, E2E (Playwright) |
| FastAPI / Python | pytest + httpx | Unit, Integration |
| PHP | PHPUnit or Pest | Unit, Feature |
| Node.js / Express | Jest or Vitest + Supertest | Unit, Integration |

---

## Output Files

| File | Purpose |
|------|---------|
| `.docs/test-strategy.md` | Testing approach documentation and guidance |
| Config files | Framework-specific configuration (vitest.config.ts, pytest.ini, etc.) |
| Example tests | Scaffolding to get started |

---

## Version History

### v1.1 (2025-12-16)

**Manifest-Based Refactor**

- Removed embedded workflow orchestration
- Skills now focus purely on their domain
- Workflow routing handled by manifest

### v1.0 (2025-11-22)

**Initial Release**

- Test framework configuration for multiple tech stacks
- Example test scaffolding (unit and integration)
- Test strategy documentation
- Educational guidance on testing patterns
- Support for Vitest, Jest, pytest, PHPUnit

---

**Version:** 1.1
**Last Updated:** 2025-12-16
