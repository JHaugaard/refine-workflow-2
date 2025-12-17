# Project Templates

This directory contains templates for new projects using the skill-foundry workflow system.

## new-project/

A minimal template for starting a new project with the workflow system.

### Structure

```
new-project/
├── .claude/
│   └── workflow-manifest.yaml  # Workflow definition (v3 schema)
├── .docs/
│   └── .gitkeep               # Handoff documents directory
└── CLAUDE.md                  # Project context with workflow pointer
```

### Usage

1. **Copy the template:**
   ```bash
   cp -r templates/new-project /path/to/your-project
   cd /path/to/your-project
   ```

2. **Initialize git (optional):**
   ```bash
   git init
   ```

3. **Start the workflow:**
   - Open the project in your editor
   - Invoke the `project-brief-writer` skill to begin

### What Each File Does

| File | Purpose |
|------|---------|
| `.claude/workflow-manifest.yaml` | Defines the workflow phases, skills, and routing logic. Loaded just-in-time by the `workflow-status` orchestrator. |
| `.docs/` | Directory for handoff documents created by each skill. These persist decisions across sessions. |
| `CLAUDE.md` | Initial project context with workflow pointer. Will be expanded by `project-spinup` skill. |

### Customization

The workflow manifest supports:

- **Variants:** Use `minimal`, `standard`, or `full` workflow paths
- **Decision gates:** Conditional routing based on project decisions
- **Termination points:** Three natural stopping points based on project needs

See `workflow-status/references/HANDOFF-DOCUMENTATION.md` for full documentation.
