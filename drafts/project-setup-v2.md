Initialize project with Claude Code config, MCP workflow integration, and environment setup.

Uses **conditional rules** for context efficiency - MCP and API key docs only load when relevant.

## Steps

1. Create structure:
   ```bash
   mkdir -p .claude/rules
   touch .claude/session-context.md .claude/session.md .claude/todo.md
   ```

2. Create `.gitignore` with best-practices defaults:

```gitignore
# =============================================================================
# Environment & Secrets
# =============================================================================
.env
.env.local
.env.*.local
*.pem
*.key

# =============================================================================
# OS & Editor
# =============================================================================
.DS_Store
Thumbs.db
*.swp
*.swo
*~
.idea/
.vscode/
*.sublime-*

# =============================================================================
# Dependencies
# =============================================================================
node_modules/
vendor/
__pycache__/
*.pyc
.venv/
venv/

# =============================================================================
# Build Output
# =============================================================================
dist/
build/
out/
*.egg-info/

# =============================================================================
# Logs & Temp Files
# =============================================================================
*.log
logs/
tmp/
temp/
.cache/
```

3. Create `.env.local` template:

```env
# =============================================================================
# LOCAL ENVIRONMENT VARIABLES - DO NOT COMMIT
# =============================================================================

# LLM API Keys
GEMINI_API_KEY=
OPENAI_API_KEY=
OPENROUTER_API_KEY=

# Project-Specific Keys (add as needed)
```

4. Create `.claude/rules/mcp-workflow.md`:

```markdown
---
paths:
  - ".claude/session-context.md"
  - ".claude/session.md"
---
# MCP Server Workflow

## Commands
| Command | Purpose |
|---------|---------|
| `docker mcp server enable [name]` | Add server and tools |
| `docker mcp server disable [name]` | Remove server and tools |
| `docker mcp server reset` | Return to 6 core tools |
| `docker mcp tools disable [tool1] ...` | Disable specific tools |
| `docker mcp tools enable [tool1] ...` | Re-enable tools |

After enable/disable: user runs `/clear` to reload.

## Session Protocol

**Start (`/session-start`):**
1. Check tool count (should be 6)
2. Ask which servers needed
3. Enable via: `docker mcp server enable [name]`
4. If many tools: "Want all, or just [subset]?"
5. User runs /clear

**During session:**
- "I could add [server] for [task]. Should I?"
- Log additions in session-context.md

**End (`/session-end`):**
1. Disable each server
2. Verify count returns to 6
3. If stuck: `docker mcp server reset`

## Phrasing Examples
- "Context7 would help with up-to-date docs. Add it?"
- "Zen adds 18 tools. Want all, or just analyze/debug/codereview?"
```

5. Create `.claude/rules/api-keys.md`:

```markdown
---
paths:
  - ".env*"
  - "*.env"
---
# API Keys Reference

Keys stored in `.env.local` (gitignored). Never commit secrets.

## Available Keys
| Variable | Service | Usage |
|----------|---------|-------|
| `GEMINI_API_KEY` | Google Gemini | Gemini models |
| `OPENAI_API_KEY` | OpenAI | GPT models, embeddings |
| `OPENROUTER_API_KEY` | OpenRouter | Multi-model access |

## Loading in Code

```bash
# Shell
source .env.local
```

```python
# Python
from dotenv import load_dotenv
load_dotenv('.env.local')
```

```javascript
// Node.js
require('dotenv').config({ path: '.env.local' })
```

## Docker MCP Secrets
```bash
# Sync to MCP if needed
source .env.local
docker mcp secret set zen.openai_api_key=$OPENAI_API_KEY
```
```

6. Run `/init` to generate base CLAUDE.md

7. Append to CLAUDE.md:

```markdown
# Project Name
<!-- Replace with your project name and description -->

## Workflow
This project uses the skill-foundry workflow system.
- **Status:** Invoke `workflow-status` skill
- **Handoffs:** `.docs/` directory

## Quick Reference
| Skill                | Purpose                                     |
| -------------------- | ------------------------------------------- |
| project-brief-writer | Transform idea into structured brief        |
| solution-architect   | Resolve architectural ambiguity (optional)  |
| tech-stack-advisor   | Get technology recommendations              |
| deployment-advisor   | Plan hosting and deployment                 |
| project-spinup       | Generate project foundation                 |
| test-orchestrator    | Set up testing (optional)                   |
| deploy-guide         | Walk through deployment                     |
| ci-cd-implement      | Set up CI/CD (optional)                     |

## Project-Specific Notes
<!-- Populated by project-spinup -->

---

## External Resources
- Shared assets: [placeholder]
- Design files: [placeholder]

## Skill Location
Personal skills at: /Users/john/.claude/skills
```

8. Initialize session-context.md:

```markdown
# Session Context

## Current Focus
[TBD]

## MCP Servers Added This Session
| Server | Tools | Status |
|--------|-------|--------|

## Key Decisions
-

## Notes
-
```

9. Wrap-up:

Tell user:
```
Project initialized with context-efficient structure!

**What's different:**
- Lean CLAUDE.md (~50 lines vs ~140)
- MCP docs in `.claude/rules/mcp-workflow.md` - loads only during session ops
- API docs in `.claude/rules/api-keys.md` - loads only when touching .env files

**API Keys:**
- Add your keys to `.env.local`
- Gitignored - safe to add real values

**Next steps:**
1. Add API keys to `.env.local`
2. Run `/session-start` to begin
3. Run `/session-end` when done
```

## Reference

Files created:
- `.gitignore` - ignore patterns
- `.env.local` - API key template (gitignored)
- `.claude/rules/mcp-workflow.md` - MCP docs (conditional)
- `.claude/rules/api-keys.md` - API docs (conditional)
- `.claude/session-context.md` - session tracking
- `.claude/session.md` - session notes
- `.claude/todo.md` - task tracking
- `CLAUDE.md` - core project instructions (lean)

Commands:
- `/session-start` - begin with MCP selection
- `/session-end` - cleanup servers
