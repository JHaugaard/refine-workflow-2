# Refine Workflow 2

This project is part of the Skill Foundry workspace.

## Project Structure
- `.claude/` - Claude Code session files and configuration
- `skills/` - Claude Code skill definitions (if applicable)

## Development Guidelines
- Follow the established patterns in the parent skill-foundry project
- Skills follow the YAML frontmatter + XML-structured content format

## External Resources

- Shared assets: [placeholder]
- Design files: [placeholder]

## Skill Location

Personal skills at: /Users/john/.claude/skills

## Environment & API Keys

API keys for external services are stored in `.env.local` at the project root. This file is gitignored and should never be committed.

**IMPORTANT:** When tasks require calling external LLM APIs (via Docker MCP Toolkit or directly), always read credentials from `.env.local`. Do not prompt for keys or guess at environment variable names.

### Available Keys

| Variable | Service | Usage |
|----------|---------|-------|
| `GEMINI_API_KEY` | Google Gemini | Gemini model API access |
| `OPENAI_API_KEY` | OpenAI | GPT models, embeddings |
| `OPENROUTER_API_KEY` | OpenRouter | Multi-model access |

### Loading Keys

```bash
# In shell scripts
source .env.local

# Or export individually
export $(grep -v '^#' .env.local | xargs)
```

```python
# In Python
from dotenv import load_dotenv
load_dotenv('.env.local')
```

```javascript
// In Node.js (with dotenv)
require('dotenv').config({ path: '.env.local' })
```

### Docker MCP Toolkit Secrets

The Docker MCP Toolkit stores secrets separately via `docker mcp secret set`.
View current secrets: `docker mcp secret ls`

To sync a key from .env.local to MCP (if needed):
```bash
source .env.local
docker mcp secret set zen.openai_api_key=$OPENAI_API_KEY
```

## MCP Server Workflow (Docker MCP Gateway)

### Architecture

Core gateway (always present, ~6 tools):
- mcp-find, mcp-add, mcp-remove, mcp-config-set, mcp-exec, code-mode

Server management (via Bash, persists to gateway):
- `docker mcp server enable [name]` - adds server and tools
- `docker mcp server disable [name]` - removes server and tools
- `docker mcp server reset` - nuclear option, back to 6 core tools
- `docker mcp tools count` - check current tool count

Tool-level control (granular):
- `docker mcp tools ls` - list all tools
- `docker mcp tools disable [tool1] [tool2] ...` - disable specific tools
- `docker mcp tools enable [tool1] [tool2] ...` - re-enable specific tools

After enable/disable: user needs /clear for Claude to see changes.

### Philosophy

- Clean slate: start sessions with only 6 core tools
- Human-in-the-loop: ask before adding servers
- Session-aware: track what's added, clean up at end
- Granular control: disable unwanted tools from verbose servers

### Workflow

At session start (`/session-start`):
- Check tool count (should be 6)
- Ask which servers needed
- Enable via: `docker mcp server enable [name]`
- If server has many tools, ask: "Want all tools or a subset?"
- Disable unwanted tools: `docker mcp tools disable [tool1] ...`
- User runs /clear to load new tools

During session:
- "I could add [server] for [task]. Should I?"
- If yes: enable via Bash, user /clear, log in session-context.md

At session end (`/session-end`):
- Disable each server via: `docker mcp server disable [name]`
- Verify tool count returns to 6
- If stuck: `docker mcp server reset`

### Phrasing

- "I could add Context7 for up-to-date docs. Should I?"
- "GitHub MCP would help with this PR. Add it?"
- "This needs database access - add postgres?"
- "Zen adds 18 tools. Want all, or just analyze/debug/codereview?"
