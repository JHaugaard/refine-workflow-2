---
name: project-spinup
description: "Generate complete project foundations with personalized CLAUDE.md, Docker setup, and project structure. Supports Guided Setup or Quick Start."
---

# project-spinup

<hard-boundaries>
BEFORE ANY OUTPUT, I MUST VERIFY:

1. I will NOT make new technology decisions - tech-stack-advisor already decided
2. I will NOT change deployment strategy - deployment-advisor already decided
3. I will NOT suggest alternative platforms or hosting options
4. I will ONLY implement decisions from upstream handoffs
5. I will USE environment registry data for infrastructure context

If an upstream decision seems problematic, I will ASK THE USER before proceeding differently.

MY SCOPE: Generate project foundation files based on LOCKED decisions from tech-stack-advisor and deployment-advisor. I am an EXECUTOR, not an ADVISOR.
</hard-boundaries>

<purpose>
Generate a complete, ready-to-code project foundation tailored to development workflow, infrastructure, and chosen tech stack. Creates comprehensive CLAUDE.md for AI assistant context, Docker configuration, directory structure, and either guided learning prompts or full scaffolding.
</purpose>

<output>
Complete project foundation including:
- CLAUDE.md (comprehensive project context)
- docker-compose.yml (local development)
- Directory structure (src/, tests/, docs/)
- .env.example, .gitignore, README.md
- Guided setup prompts OR full scaffolding
- .docs/project-foundation.json (structured handoff)
</output>

---

<workflow>

<phase id="0" name="load-environment">
<description>Load environment context scoped to this skill</description>

<steps>
1. Attempt to read ~/.claude/environment.json
2. If not found: Note and proceed (graceful degradation)
3. If found:
   a. Read skill_data_access["project-spinup"]
   b. Extract ONLY the listed paths:
      - infrastructure.services
      - credentials.api_endpoints
      - established_choices
   c. Do NOT reference out-of-scope data
</steps>

<environment-usage>
From environment registry, this skill uses:
- **infrastructure.services**: Available services (Supabase, PocketBase, Redis, etc.) for docker-compose and connection config
- **credentials.api_endpoints**: API URLs for environment variable templates
- **established_choices**: Locked decisions (containerization: docker, proxy: caddy, etc.)

This skill does NOT access:
- deployment_options (deployment-advisor's scope)
- skill_guidance (advisory skills' scope)
- infrastructure.vps (deployment-advisor/deploy-guide scope)
</environment-usage>

<graceful-degradation>
If environment.json is missing or inaccessible:
- Note: "Environment registry not found - using defaults"
- Proceed with hardcoded user context
- Generate standard templates without infrastructure-specific customization
</graceful-degradation>
</phase>

<phase id="1" name="load-handoffs">
<action>Load and validate upstream JSON handoffs.</action>

<expected-handoffs>
1. .docs/PROJECT-MODE.md (workflow mode declaration)
2. .docs/brief.json (project brief from project-brief-writer)
3. .docs/tech-stack-decision.json (from tech-stack-advisor)
4. .docs/deployment-strategy.json (from deployment-advisor)
</expected-handoffs>

<loading-process>
1. Scan .docs/ for expected handoff documents
2. Parse JSON handoffs and validate structure
3. Extract LOCKED decisions - these are non-negotiable
4. **Check for native platform**: If deployment-strategy.json contains `native_platform.platform` of "ios", "macos", or "tauri" → proceed to Phase 1.5 (native-platform-scaffolding)
5. If handoffs missing: Gather equivalent information conversationally
6. Summarize loaded context before proceeding
</loading-process>

<when-handoffs-exist>
"I've loaded your workflow handoffs:

**Mode:** {mode from PROJECT-MODE.md}
**Project:** {project name from brief.json}
**Tech Stack:** {primary technologies from tech-stack-decision.json}
**Deployment:** {hosting approach from deployment-strategy.json}

These are LOCKED decisions from earlier workflow phases.

Ready to generate your project foundation?"
</when-handoffs-exist>

<when-handoffs-missing>
"I don't see all the expected handoff documents in .docs/. No problem - let me gather what I need.

To generate your project foundation, I need to understand:
1. **Project name?** (kebab-case preferred)
2. **Tech stack?** (Frontend, backend, database)
3. **Deployment target?** (Localhost, VPS, cloud platform)
4. **Learning or delivery focus?** (Affects scaffolding approach)

Once you share these, I can generate your project foundation."
</when-handoffs-missing>

<key-principle>
This skill NEVER blocks on missing prerequisites. It gathers information conversationally and proceeds.
</key-principle>
</phase>

<phase id="1.5" name="native-platform-scaffolding">
<action>Handle native platform projects (iOS, macOS, Tauri) with specialized scaffolding.</action>

<trigger>
deployment-strategy.json contains `native_platform.platform` of: "ios", "macos", or "tauri"
</trigger>

<context>
Native platforms don't need Docker-based development environments. They need:
- Platform-appropriate project structure
- Xcode project setup instructions (iOS/macOS) or Tauri initialization (Tauri)
- Platform-tailored CLAUDE.md with SwiftUI/Tauri conventions
- Backend service integration (if tech-stack includes Supabase/PocketBase)
</context>

<response>
"Your project targets **{platform}** — a native platform with specialized scaffolding needs.

I'll generate:
- Directory structure appropriate for {platform}
- CLAUDE.md tailored to {platform} development patterns
- Setup instructions for Xcode/Tauri
{If backend: - Pre-configured API service layer for {backend}}
- .docs/project-foundation.json handoff

Would you prefer **Guided Setup** (step-by-step with explanations) or **Quick Start** (complete scaffold)?"
</response>

<platform-specific-scaffolding>

<ios-ipados>
<directory-structure>
{project-name}/
├── CLAUDE.md                    # SwiftUI-focused project context
├── README.md                    # Xcode setup instructions
├── .gitignore                   # Xcode/Swift ignores
├── .env.example                 # API keys, backend URLs (if applicable)
├── {ProjectName}/               # Main app target (created by Xcode)
│   ├── App/
│   │   └── {ProjectName}App.swift
│   ├── Views/
│   │   ├── ContentView.swift
│   │   └── Components/
│   ├── Models/
│   ├── Services/                # API integration (if backend)
│   │   └── APIService.swift
│   ├── Utilities/
│   ├── Resources/
│   │   └── Assets.xcassets
│   └── Info.plist
├── {ProjectName}Tests/
├── docs/
│   └── setup-instructions.md    # Detailed Xcode walkthrough
└── .docs/
    └── project-foundation.json
</directory-structure>

<claude-md-sections>
Key sections for iOS/iPadOS CLAUDE.md:
- SwiftUI Conventions (@State, @Binding, @Observable, @Environment)
- View Composition Patterns (extract components, ViewBuilders)
- Navigation Patterns (NavigationStack, NavigationSplitView)
- Data Flow (MVVM-lite, service layer pattern)
- Common Commands (xcodebuild, swift build, xcrun simctl)
- Backend Integration (Supabase Swift SDK or PocketBase patterns)
- Testing Approach (XCTest, SwiftUI previews)
- Device Deployment (Xcode → device via USB/Wi-Fi)
</claude-md-sections>

<setup-instructions>
README.md includes:
1. Prerequisites: macOS, Xcode (version), Apple Developer account (free tier OK)
2. Create Xcode project: File → New → Project → iOS App → SwiftUI
3. Copy generated files into Xcode project structure
4. Configure signing (personal team for device deployment)
5. Add Swift packages (if backend: Supabase SDK)
6. Run on simulator or device
</setup-instructions>

<backend-integration>
When tech-stack includes Supabase or PocketBase:
- Add Swift package dependency (supabase-swift or custom PocketBase client)
- Generate Services/APIService.swift with:
  - Connection setup pointing to homelab URLs (from environment.json)
  - Auth flow skeleton (signIn, signOut, currentUser)
  - Basic CRUD pattern for primary model
  - Error handling patterns
- Generate .env.example with SUPABASE_URL, SUPABASE_ANON_KEY (or POCKETBASE_URL)
- Note: iOS apps read env via Config.swift or xcconfig files
</backend-integration>
</ios-ipados>

<macos>
<directory-structure>
{project-name}/
├── CLAUDE.md                    # macOS SwiftUI-focused context
├── README.md                    # Xcode setup for macOS
├── .gitignore
├── .env.example
├── {ProjectName}/
│   ├── App/
│   │   └── {ProjectName}App.swift
│   ├── Views/
│   │   ├── ContentView.swift
│   │   ├── Sidebar/
│   │   └── Detail/
│   ├── Models/
│   ├── Services/
│   │   └── APIService.swift
│   ├── Utilities/
│   ├── Resources/
│   │   └── Assets.xcassets
│   └── Info.plist
├── {ProjectName}Tests/
├── docs/
│   └── setup-instructions.md
└── .docs/
    └── project-foundation.json
</directory-structure>

<claude-md-sections>
Key sections for macOS CLAUDE.md:
- macOS SwiftUI Patterns (NavigationSplitView, Settings, MenuBarExtra)
- Window Management (WindowGroup, Window, MenuBar)
- AppKit Integration (when needed for advanced features)
- Sandboxing & Entitlements
- Menu Bar Apps (if applicable)
- File System Access (NSOpenPanel, document-based apps)
- Backend Integration patterns
- Distribution (Developer ID, notarization for sharing)
</claude-md-sections>
</macos>

<tauri>
<directory-structure>
{project-name}/
├── CLAUDE.md                    # Tauri + web frontend context
├── README.md                    # Tauri setup instructions
├── .gitignore                   # Rust + Node ignores
├── .env.example
├── package.json                 # Frontend dependencies
├── src/                         # Web frontend (React/Vue/Svelte/vanilla)
│   ├── index.html
│   ├── main.ts
│   ├── App.tsx                  # (or .vue, .svelte)
│   ├── components/
│   ├── services/
│   │   └── api.ts               # Backend integration
│   └── styles/
├── src-tauri/                   # Rust backend
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   ├── src/
│   │   ├── main.rs
│   │   └── lib.rs
│   ├── icons/
│   └── capabilities/
├── tests/
├── docs/
│   └── setup-instructions.md
└── .docs/
    └── project-foundation.json
</directory-structure>

<claude-md-sections>
Key sections for Tauri CLAUDE.md:
- Tauri Architecture (web frontend + Rust backend + system webview)
- Frontend Framework Conventions (based on tech-stack-decision)
- Tauri Commands (invoke Rust from JS)
- IPC Patterns (frontend ↔ Rust communication)
- File System Access (Tauri APIs)
- Window Management
- Build & Bundle (tauri build, platform targets)
- Backend Integration (standard web patterns via fetch/axios)
- Distribution (DMG for macOS, unsigned for personal use)
</claude-md-sections>

<setup-instructions>
README.md includes:
1. Prerequisites: Node.js, Rust, Xcode Command Line Tools (macOS)
2. Install Tauri CLI: `cargo install tauri-cli` or `npm install -g @tauri-apps/cli`
3. Install dependencies: `npm install`
4. Development: `npm run tauri dev`
5. Build: `npm run tauri build`
6. Output: target/release/bundle/
</setup-instructions>

<backend-integration>
When tech-stack includes Supabase or PocketBase:
- Standard web client libraries (@supabase/supabase-js or pocketbase)
- Generate src/services/api.ts with connection setup
- Environment variables via .env (Vite/webpack) or tauri.conf.json
</backend-integration>
</tauri>

</platform-specific-scaffolding>

<skip-phases>
When native platform detected:
- Skip Phase 2 (spinup-approach prompt) — already asked above
- Skip Phase 4 (generate-core-files for Docker) — not applicable
- Proceed to Phase 3 (approval-gate) with native-specific plan
- Then Phase 5-native (apply native scaffolding)
- Then Phase 6-7 (handoff and summary)
</skip-phases>

<docker-exception>
If the project includes a **separate backend component** (not just connecting to existing infrastructure):
- Generate docker-compose.yml for the backend only
- Native app connects to locally-running backend during development
- Example: iOS app + custom Express API → docker-compose for Express, iOS connects to localhost:3000
</docker-exception>
</phase>

<phase id="2" name="spinup-approach">
<action>Determine scaffolding approach based on mode and user preference.</action>

<mode-vs-spinup-distinction>
- **PROJECT-MODE** (LEARNING/BALANCED/DELIVERY): Governs advisory skills - exploration, discussion, checkpoints. Strategic learning.
- **spinup_approach** (Guided/Quick Start): Governs scaffolding style - step-by-step or all-at-once. Tactical implementation.

Key insight: You can be in LEARNING mode but use Quick Start if familiar with the stack. Or DELIVERY mode but use Guided Setup for a new framework.
</mode-vs-spinup-distinction>

<prompt-to-user>
For the project scaffolding, would you prefer:

**1. Guided Setup** - I'll create the foundation, then provide step-by-step prompts to build out the structure incrementally. You'll learn how each layer works.
(Recommended if you're new to {tech stack})

**2. Quick Start** - I'll generate the complete project scaffolding immediately with all boilerplate code and configuration.
(Recommended if you're familiar with {tech stack})

Which approach would you like?
</prompt-to-user>
</phase>

<phase id="3" name="approval-gate">
<action>Present foundation plan for user approval before generating files.</action>

<planning-mindset>
This gate ensures alignment before committing to file generation.
</planning-mindset>

<approval-prompt>
Before I generate your project foundation, let me confirm the plan:

**Project:** {project_name}
**Tech Stack:** {tech_stack_summary}
**Deployment Target:** {deployment_target}
**Spinup Approach:** {guided/quick-start}

**Files I'll Create:**
- CLAUDE.md - Comprehensive project context ({estimated_size})
- docker-compose.yml - Local development environment
- Directory structure: src/, tests/, docs/
- .env.example - Environment variable template
- .gitignore - Tech-stack-appropriate
- README.md - Setup instructions
- .docs/project-foundation.json - Handoff marker

{If Quick Start: - Complete project scaffolding with starter code}
{If Guided Setup: - Step-by-step prompts in CLAUDE.md "Next Steps" section}

**Infrastructure context loaded:**
{If environment loaded: List relevant services (Supabase, PocketBase, etc.)}
{If no environment: Using standard defaults}

Does this look correct? Should I proceed?
</approval-prompt>

<on-user-concerns>
If user has concerns about the plan:
1. Clarify what can be adjusted (spinup approach, specific features)
2. Explain what CANNOT be changed here (tech stack, deployment - those are upstream decisions)
3. If upstream decisions need revision, recommend re-running the appropriate skill
</on-user-concerns>
</phase>

<phase id="4" name="generate-core-files">
<action>Create foundation files after user approval.</action>

<always-create>
1. CLAUDE.md - Comprehensive project context (see CLAUDE-MD-TEMPLATE section)
2. docker-compose.yml - Local development environment
3. Directory structure - src/, tests/, docs/
4. .gitignore - Tech-stack-appropriate
5. README.md - Setup and development instructions
6. .env.example - Required environment variables
7. Git initialization guidance
</always-create>

<environment-integration>
When environment.json is available:
- Use infrastructure.services for docker-compose service references
- Use credentials.api_endpoints for .env.example URLs
- Use established_choices for configuration defaults (e.g., containerization: docker)
</environment-integration>
</phase>

<phase id="5" name="apply-spinup-approach">

<guided-setup>
Generate foundation only, plus detailed "Next Steps" section in CLAUDE.md with:
- Step-by-step prompts to give Claude Code
- Explanation of what each step creates
- Learning objectives for each step
- Verification checkpoints
- Estimated time for each step

See EXAMPLES.md for detailed guided setup templates.
</guided-setup>

<quick-start>
Generate complete scaffolding including:
- All tech-stack-specific configuration files
- Complete source file structure
- Starter/example code with comments
- Sample tests
- All middleware/utilities
- Initial git commit prepared

See EXAMPLES.md for quick start structure examples.
</quick-start>

</phase>

<phase id="6" name="create-handoff">
<action>Create .docs/project-foundation.json structured handoff.</action>

<json-schema>
{
  "document_type": "project-foundation",
  "version": "1.0",
  "created": "[YYYY-MM-DD]",
  "project": "[project-name]",
  "mode": "[LEARNING/DELIVERY/BALANCED]",

  "rationale": "[1-2 sentence summary of what was generated and why]",

  "tech_stack": {
    "frontend": "[from tech-stack-decision.json]",
    "backend": "[from tech-stack-decision.json]",
    "database": "[from tech-stack-decision.json]",
    "key_libraries": ["[lib1]", "[lib2]"]
  },

  "deployment": {
    "target": "[from deployment-strategy.json]",
    "approach": "[docker/native/serverless]",
    "is_termination_point": "[true if localhost or native platform]"
  },

  "native_platform": {
    "platform": "ios | macos | tauri | null",
    "xcode_project_name": "[PascalCase project name, if iOS/macOS]",
    "backend_integration": {
      "service": "[supabase | pocketbase | none]",
      "endpoint": "[URL from environment.json or null]",
      "sdk": "[supabase-swift | pocketbase | @supabase/supabase-js | null]"
    },
    "setup_method": "instructions | scaffold",
    "distribution": "personal-device"
  },
  // NOTE: native_platform is OPTIONAL. Omit entirely for web projects.

  "generated_files": [
    {"path": "CLAUDE.md", "purpose": "Project context for AI assistant"},
    {"path": "docker-compose.yml", "purpose": "Local development environment"},
    {"path": ".env.example", "purpose": "Environment variable template"},
    {"path": "README.md", "purpose": "Setup instructions"}
  ],

  "spinup_approach": "guided | quick-start",

  "environment_integration": {
    "registry_loaded": "[true/false]",
    "services_referenced": ["[service1]", "[service2]"],
    "endpoints_configured": ["[endpoint1]", "[endpoint2]"]
  },

  "workflow_status": {
    "completed_phases": ["project-brief-writer", "tech-stack-advisor", "deployment-advisor", "project-spinup"],
    "is_termination_point": "[true if localhost deployment]",
    "next_phases": ["test-orchestrator", "deploy-guide", "ci-cd-implement"]
  },

  "next_actions": [
    "[Action 1 based on spinup approach]",
    "[Action 2]"
  ],

  "handoff_to": ["test-orchestrator", "deploy-guide"]
}
</json-schema>

<save-location>.docs/project-foundation.json</save-location>
</phase>

<phase id="7" name="provide-summary">
<action>Summarize what was created and provide clear next steps.</action>

<summary-template>
## Project Foundation Created

**Project:** {project_name}
**Tech Stack:** {primary technologies}
**Spinup Approach:** {Guided Setup or Quick Start}
**Deployment Target:** {localhost / hosting approach}

---

### Workflow Status

**PLANNING PHASES - COMPLETE**
- Phase 0: project-brief-writer
- Phase 1: tech-stack-advisor
- Phase 2: deployment-advisor

**SETUP PHASE - COMPLETE**
- Phase 3: project-spinup (this skill)

[If Localhost:]
**WORKFLOW TERMINATION POINT**

Your localhost project is ready for development. No further workflow phases needed.

If you later decide to deploy publicly:
1. Re-run deployment-advisor to choose a hosting target
2. Continue with deploy-guide and optionally ci-cd-implement

[If Public Deployment:]
**DEVELOPMENT PHASE - START**

Build your features! When you're ready:
- Phase 4: test-orchestrator (optional - set up testing infrastructure)
- Phase 5: deploy-guide (deploy your application)
- Phase 6: ci-cd-implement (optional - automate deployments)

---

### Quick Start Commands

```bash
cd {project_name}
git init && git checkout -b main
git add . && git commit -m "chore: initial project setup via project-spinup"
git checkout -b dev

docker compose up -d
{dev server command}
```

Open http://localhost:{port} to see your application.

---

### Next Actions

{If Guided Setup:}
1. Open CLAUDE.md and read "Next Steps (Guided Setup)"
2. Copy the first prompt and paste it to Claude Code
3. Follow the guided steps incrementally

{If Quick Start:}
1. Review generated code structure
2. Copy .env.example to .env.local and configure
3. Start building features!
</summary-template>
</phase>

</workflow>

---

<claude-md-template>
See QUICK_REFERENCE.md for the complete CLAUDE.md template structure.

Key sections:
- Developer Profile
- Project Overview
- Development Environment
- Infrastructure & Hosting
- Development Workflow
- Code Conventions
- Common Commands
- Project-Specific Notes
- Deployment
- Resources & References
- Troubleshooting
- Next Steps (Guided Setup) or Immediate Actions (Quick Start)
</claude-md-template>

---

<tech-stack-templates>
See EXAMPLES.md for detailed tech stack templates including:
- Next.js + Supabase
- PHP + MySQL
- FastAPI + PostgreSQL
- Laravel + SQLite/PostgreSQL
- **iOS/iPadOS (SwiftUI) + Supabase/PocketBase**
- **macOS (SwiftUI) + Supabase/PocketBase**
- **Tauri + React/Vue + Supabase/PocketBase**

Each template includes:
- Default configuration
- Files to generate (Quick Start)
- Guided Setup step sequence
- Platform-specific CLAUDE.md sections (for native platforms)
</tech-stack-templates>

---

<guardrails>

<must-do>
- Load environment registry first (Phase 0)
- Load ALL JSON handoff documents (if they exist)
- Use handoff documents as primary source for decisions
- Gather missing information conversationally (never block)
- **Detect native platforms early** (Phase 1) and route to Phase 1.5
- Present approval gate before generating files
- Ask about spinup approach with MODE-informed suggestion
- Adapt templates to tech stack (including native platforms)
- Be comprehensive in CLAUDE.md (platform-tailored for native)
- Include user's context (workflow, infrastructure, learning goals)
- Respect user choice (Guided/Quick Start equally valid)
- Include testing setup and examples
- Include docker-compose.yml for web projects (skip for pure native)
- Security-conscious (.env for secrets, .gitignore)
- Create .docs/project-foundation.json structured handoff
- Show workflow status with termination point guidance
- **For native platforms**: Include backend integration when tech-stack specifies Supabase/PocketBase
- **For native platforms**: Prefer homelab infrastructure (Supabase/PocketBase on vps8) when available
</must-do>

<must-not-do>
- Make new technology decisions (tech-stack-advisor's scope)
- Change deployment strategy (deployment-advisor's scope)
- Suggest alternative platforms or hosting options
- Skip reading handoff documents (if they exist)
- Skip the approval gate
- Use wrong templates for tech stack
- Assume spinup approach without asking
- Generate incomplete foundation
- Block on missing prerequisites (gather info instead)
- Treat localhost projects as incomplete workflows
- Access environment registry paths outside my scope
</must-not-do>

</guardrails>

---

<workflow-status>
Phase 3 of 6: Project Foundation

Upstream:
  Phase 0: Project Brief (project-brief-writer) - produces brief.json
  Phase 1: Tech Stack (tech-stack-advisor) - produces tech-stack-decision.json
  Phase 2: Deployment Strategy (deployment-advisor) - produces deployment-strategy.json

Current:
  Phase 3: Project Foundation (you are here) <- TERMINATION POINT (localhost OR native platforms)

Downstream:
  Phase 4: Test Strategy (test-orchestrator) - optional
  Phase 5: Deployment (deploy-guide) <- TERMINATION POINT (manual deploy)
  Phase 6: CI/CD (ci-cd-implement) <- TERMINATION POINT (full automation)

Note: Native platforms (ios, macos, tauri) terminate here. Device installation is handled
via Xcode/Tauri build, not deploy-guide. deploy-guide may later add native device walkthrough.
</workflow-status>

---

<integration-notes>

<workflow-position>
Phase 3 of 6 in the Skills workflow chain.
Expected input: .docs/brief.json, .docs/tech-stack-decision.json, .docs/deployment-strategy.json
Produces: Project foundation + .docs/project-foundation.json

This is a TERMINATION POINT for localhost/learning projects.
</workflow-position>

<flexible-entry>
This skill can be invoked standalone without prior phases. Missing context is gathered through conversation rather than blocking.
</flexible-entry>

<termination-points>
- If deployment target is localhost: Workflow terminates here
- If deployment target is native platform (ios, macos, tauri): Workflow terminates here
- If deployment target is public web: Workflow continues to deploy-guide (Phase 5)
</termination-points>

<status-utility>
Users can invoke the **workflow-status** skill at any time to:
- See current workflow progress
- Check which phases are complete
- Get guidance on next steps
- Review all handoff documents

Mention this option when users seem uncertain about their progress.
</status-utility>

</integration-notes>
