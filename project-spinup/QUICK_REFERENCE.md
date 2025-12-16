# Project Starter - Quick Reference Guide

## TL;DR

**What**: Generates personalized project foundations with comprehensive CLAUDE.md files
**When**: After tech-stack-advisor and deployment-advisor
**Output**: Foundation + Learning Mode prompts OR complete scaffolding

---

## Basic Usage

### Invoke the Skill

```
Say to Claude Code: "Use the project-spinup skill to initialize my project"
```

You'll be asked for:
- Project name (kebab-case, e.g., "recipe-manager")
- Project description
- Tech stack (from tech-stack-advisor)
- Hosting plan (from deployment-advisor)
- Complexity level: simple/standard/complex (default: standard)
- Learning mode: true/false (default: true)
- Special features (auth, real-time, file uploads, etc.)

---

## Two Modes

### Learning Mode (Default) ✅ Recommended for Learning

**What you get**:
- Project foundation (docker-compose.yml, directories, configs)
- Comprehensive CLAUDE.md with guided "Next Steps"
- Copy-paste prompts for Claude Code
- Explanations of what each step teaches

**When to use**:
- First time with a tech stack
- Want to understand each layer
- Learning is the primary goal
- Not in a rush

**Workflow**:
1. Skill generates foundation
2. Open CLAUDE.md
3. Copy first prompt from "Next Steps"
4. Paste to Claude Code
5. Review what was created
6. Repeat for each step

### Quick Start Mode

**What you get**:
- Everything from Learning Mode PLUS
- Complete project scaffolding
- All config files
- Starter/example code
- Sample tests
- Ready to code immediately

**When to use**:
- Already familiar with the tech stack
- Want to move fast
- Prototyping or time-constrained
- Confident in choices

**Workflow**:
1. Skill generates everything
2. Configure .env.local
3. Run docker compose up -d
4. Start dev server
5. Start coding features

**To activate**: Set `learning_mode: false` when invoking

---

## What Gets Generated

### Always Created

```
project-name/
├── CLAUDE.md              # Comprehensive project context
├── docker-compose.yml     # Local dev environment
├── .env.example          # Environment variables template
├── .gitignore            # Tech-stack-appropriate
├── README.md             # Setup instructions
├── src/                  # Source code directory
├── tests/                # Test files
└── docs/                 # Documentation
```

### Learning Mode Adds

- Detailed "Next Steps" section in CLAUDE.md
- Step-by-step prompts for Claude Code
- Learning objectives for each step
- Verification checkpoints
- Estimated time for each step

### Quick Start Mode Adds

- All configuration files (package.json, tsconfig.json, etc.)
- Complete source file structure
- Starter/example code with comments
- Sample tests (unit, integration)
- Middleware and utilities
- All tech-stack-specific files

---

## Supported Tech Stacks

The skill adapts to any tech stack, with built-in templates for:

### Next.js + Supabase
- Next.js 15 (App Router)
- TypeScript (strict mode)
- Tailwind CSS v4
- shadcn/ui components
- Supabase (auth, database, storage)
- React Query
- Jest or Vitest

### PHP + MySQL
- PHP 8.2+
- MySQL 8.0+
- Composer
- MVC structure
- PDO database access
- PHPUnit testing
- Docker (PHP-Apache, MySQL, phpMyAdmin)

### Laravel
- Laravel 11
- Eloquent ORM
- Blade templating
- Artisan CLI
- PHPUnit
- Docker

### FastAPI + PostgreSQL
- FastAPI
- PostgreSQL 15
- SQLAlchemy ORM
- Alembic migrations
- Pydantic validation
- pytest
- uvicorn ASGI server

### Native Platforms (iOS, macOS, Tauri)

**iOS/iPadOS (SwiftUI)**
- SwiftUI declarative UI
- Xcode project (created manually, files copied in)
- Supabase Swift SDK or PocketBase via URLSession
- XCTest for testing
- Device deployment via Xcode

**macOS (SwiftUI)**
- SwiftUI with macOS-specific patterns
- MenuBarExtra, Settings, WindowGroup
- Same backend integration as iOS
- Drag to Applications for distribution

**Tauri (Desktop)**
- Web frontend (React/Vue/Svelte) + Rust backend
- System webview (lightweight)
- Tauri commands for native APIs
- `npm run tauri build` for distribution

### Other Stacks
The skill adapts to any stack provided by tech-stack-advisor. Just provide the details.

---

## Workflow Integration

### Recommended Sequence

```
┌─────────────────────────────────────────────────┐
│ 1. Brainstorm Project Idea                     │
│    - Write down goals, features, requirements  │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ 2. Run tech-stack-advisor                      │
│    - Analyzes requirements                      │
│    - Recommends tech stack with rationale      │
│    - Provides alternatives                      │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ 3. Run deployment-advisor                         │
│    - Recommends hosting approach               │
│    - Suggests deployment workflow              │
│    - Estimates costs                           │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ 4. Invoke project-spinup ← YOU ARE HERE       │
│    - Generates foundation or full scaffold     │
│    - Creates comprehensive CLAUDE.md           │
│    - Sets up Docker environment                │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│ 5. Build Features with Claude Code             │
│    - Follow Learning Mode steps OR             │
│    - Start coding immediately (Quick Start)    │
│    - Use CLAUDE.md as context for Claude Code  │
└─────────────────────────────────────────────────┘
```

---

## Your Personal Context (Hardcoded)

The skill automatically includes:

### Developer Profile
- **Name**: John
- **Level**: Hobbyist, learner, beginner-to-intermediate
- **Goal**: Deep understanding of full-stack development
- **Approach**: Heavy reliance on Claude Code

### Development Environment
- **Computers**: MacBook Pro and Mac Mini
- **Sync**: iCloud Documents, portable SSDs, GitHub repos
- **IDE**: VS Code
- **Git**: main+dev branches, conventional commits
- **Directory Structure**: src/, docs/, tests/

### Infrastructure
- **Self-Hosted**: Supabase, n8n, Ollama, Redis, Nginx, PostgreSQL (on Hostinger VPS)
- **Simple Projects**: PocketBase
- **File Storage**: Backblaze B2 or local VPS
- **DNS**: Cloudflare
- **SSH User**: john
- **Deployment**: Hostinger VPS or shared hosting

All of this context is automatically included in every CLAUDE.md file generated.

---

## Common Scenarios

### Scenario 1: Learning a New Stack

```
You: "I want to learn Next.js by building a recipe manager"

1. Run tech-stack-advisor → recommends Next.js + Supabase
2. Run deployment-advisor → recommends Vercel or Hostinger VPS
3. Invoke project-spinup with learning_mode=true
4. Follow guided steps in CLAUDE.md
5. Build incrementally, understanding each layer
```

### Scenario 2: Quick Prototype

```
You: "I need a quick prototype for a client meeting tomorrow"

1. Skip meta-skills (you already know what to use)
2. Invoke project-spinup with learning_mode=false
3. Provide tech stack manually: "Next.js + Supabase"
4. Skill generates everything immediately
5. Configure .env, run docker, start coding
6. Focus on features, not setup
```

### Scenario 3: Exploring Multiple Stacks

```
You: "I want to compare Next.js vs Laravel for this project"

1. Run tech-stack-advisor → get comparison
2. Create two projects:
   - project-spinup for "recipe-manager-nextjs" (learning_mode=true)
   - project-spinup for "recipe-manager-laravel" (learning_mode=true)
3. Build same feature in both
4. Compare experience, performance, complexity
5. Choose winner, continue with that stack
```

### Scenario 4: Experienced Developer Mode

```
You: "I've built 3 Next.js projects, starting another one"

1. Invoke project-spinup with learning_mode=false
2. Everything generated immediately
3. Modify generated code to fit specific needs
4. Start building features right away
```

### Scenario 5: Native iOS App with Backend

```
You: "I want to build a habit tracker iOS app with cloud sync"

1. Run tech-stack-advisor → recommends SwiftUI + Supabase
2. Run deployment-advisor → detects native platform, creates minimal handoff
3. Invoke project-spinup → generates:
   - SwiftUI project structure (to copy into Xcode)
   - Pre-configured SupabaseService.swift pointing to homelab
   - CLAUDE.md with SwiftUI conventions
   - Setup instructions for Xcode
4. Create Xcode project manually
5. Copy generated files into Xcode structure
6. Add Supabase Swift package
7. Start building!
```

### Scenario 6: Tauri Desktop App

```
You: "I want a local markdown editor desktop app"

1. Run tech-stack-advisor → recommends Tauri + React
2. Run deployment-advisor → detects native platform (tauri)
3. Invoke project-spinup → generates:
   - Tauri project structure (src/ + src-tauri/)
   - React components scaffold
   - Rust commands for file system access
   - CLAUDE.md with Tauri conventions
4. npm install && npm run tauri dev
5. Start building!
```

---

## Tips & Best Practices

### For Learning Mode

✅ **Do**:
- Take your time on each step
- Review generated code before moving on
- Ask Claude Code "why" questions
- Commit after each step
- Test incrementally

❌ **Don't**:
- Rush through steps without understanding
- Skip verifications
- Copy-paste without reading
- Move to next step if current isn't working

### For Quick Start Mode

✅ **Do**:
- Review all generated files before editing
- Understand project structure
- Check configuration files
- Run tests to ensure setup works
- Commit initial state before changes

❌ **Don't**:
- Assume everything is perfect
- Skip reading CLAUDE.md
- Forget to configure .env
- Start coding without running dev server first

### General

✅ **Always**:
- Read the generated CLAUDE.md thoroughly
- Keep CLAUDE.md updated as project evolves
- Use Docker for consistent environments
- Follow conventional commit format
- Write tests as you build
- Commit working increments frequently

❌ **Never**:
- Commit .env files (only .env.example)
- Skip Docker setup
- Ignore test failures
- Make large changes without committing
- Deploy without testing locally first

---

## Troubleshooting

### "Skill isn't generating files"

**Check**:
- Did you provide all required inputs?
- Is the tech stack clearly specified?
- Try providing more details about the project

**Fix**: Be explicit with tech stack choice. Say: "Use Next.js 15 with TypeScript and Supabase"

### "Generated code doesn't match my tech stack"

**Cause**: Tech stack info might be ambiguous

**Fix**:
- Be very specific about framework versions
- Mention key libraries explicitly
- Reference output from tech-stack-advisor

### "Learning Mode steps are too basic/advanced"

**Adjust**:
- Set complexity level (simple/standard/complex)
- Switch to Quick Start Mode if too basic
- Ask Claude Code to elaborate on steps if too advanced

### "I want different directory structure"

**Fix**: After generation, tell Claude Code:
```
"Reorganize the project to use [your preferred structure].
Update CLAUDE.md to reflect the new organization."
```

### "Docker isn't working"

**Common issues**:
- Docker Desktop not running
- Port conflicts (another service using same port)
- Missing .env configuration

**Debug**:
```bash
# Check Docker is running
docker ps

# Check logs
docker compose logs

# Rebuild
docker compose down -v
docker compose up --build -d
```

---

## Customization

### Changing Defaults

After generation, you can customize:

**Port numbers**: Edit docker-compose.yml
**Directory structure**: Move files, update imports
**Dependencies**: Add to package.json/composer.json/requirements.txt
**Testing framework**: Switch to preferred tool
**Code style**: Configure linter/formatter

Claude Code will help with any customization. Just reference the existing structure.

### Extending Learning Mode

Add your own steps to the "Next Steps" section in CLAUDE.md:

```markdown
### Step X: [Your Custom Step]
⏱️ ~X minutes | 🎯 Learning: [What this teaches]

**Say to Claude Code**:
```
[Your prompt]
```

**What will be created**: [Files/functionality]
**Verification**: [How to test]
```

---

## Progressive Learning Path

### First Project (Learning Mode)
- Follow every step carefully
- Read all generated code
- Ask questions about everything
- Take notes on patterns you see
- Time estimate: 3-6 hours total

### Second Project (Learning Mode)
- Steps feel more familiar
- Recognize patterns from first project
- Modify steps to fit your needs
- Time estimate: 2-4 hours

### Third Project (Mixed)
- Use Learning Mode for new features
- Use Quick Start for familiar parts
- Start customizing default templates
- Time estimate: 1-2 hours

### Beyond (Quick Start)
- Generate everything immediately
- Confident in stack choices
- Know what to modify for your needs
- Time estimate: 15-30 minutes

---

## Cheat Sheet

### Invoke with Defaults
```
"Use project-spinup skill for my new project"
→ Learning Mode, standard complexity
```

### Invoke Quick Start
```
"Use project-spinup skill with quick start mode"
→ Full scaffolding immediately
```

### Specify Everything
```
"Use project-spinup skill:
- Project: recipe-manager
- Stack: Next.js 15 + Supabase
- Complexity: standard
- Learning mode: true
- Features: auth, real-time, file uploads"
```

### After Generation
```
# Initialize git
cd project-name
git init
git checkout -b main
git add .
git commit -m "chore: initial project setup"
git checkout -b dev

# Start development
docker compose up -d
[dev server command from README]
```

---

## Getting Help

### During Setup
- Read CLAUDE.md thoroughly
- Check README.md for setup steps
- Review SKILL.md for skill details
- This file (QUICK_REFERENCE.md) for common questions

### During Development
- Ask Claude Code (it has CLAUDE.md context)
- Check framework documentation
- Review example code generated in Learning Mode
- Look at docker logs for environment issues

### Stuck?
```
Say to Claude Code: "I'm stuck with [specific issue].
According to CLAUDE.md, I should be able to [expected outcome],
but I'm getting [actual outcome]. Can you help debug?"
```

---

## Remember

🎯 **Learning Mode is your friend** - Use it for the first 2-3 projects in any stack

🚀 **Quick Start when confident** - Switch when setup becomes routine

📚 **CLAUDE.md is your guide** - Keep it updated, reference it often

🐳 **Docker is consistent** - Use it even if you think you don't need it

✅ **Test as you build** - Don't skip testing setup

💾 **Commit frequently** - Small, working increments

🤝 **Claude Code is your partner** - Ask questions, request explanations

---

**Created**: 2025-11-04
**Skill Version**: 1.0
**For**: John's learning journey
