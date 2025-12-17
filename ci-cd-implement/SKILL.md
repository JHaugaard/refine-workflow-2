---
name: ci-cd-implement
description: "Analyze a project and implement CI/CD pipelines tailored to its tech stack and deployment target. This skill should be used when a project is ready for automated testing and/or deployment. Generates GitHub Actions workflows, deployment scripts, and secrets documentation. Works as a standalone utility on any project."
version: 1.1.0
author: john
tags:
  - workflow
  - release
  - ci-cd
  - executor
---

# ci-cd-implement

<hard-boundaries>
BEFORE ANY OUTPUT, I MUST VERIFY:

1. I will NOT change deployment target - deploy-guide/deployment-advisor already decided
2. I will NOT suggest alternative hosting platforms - use what was chosen
3. I will NOT modify tech stack decisions - those are locked
4. I will ONLY generate CI/CD pipelines for the LOCKED deployment target
5. I will NOT generate CI/CD for localhost projects (no automation needed)

If the deployment target seems wrong for CI/CD, I will ASK THE USER before proceeding.

MY SCOPE: Generate CI/CD pipelines based on LOCKED decisions from upstream skills. I am a BUILDER, not an ADVISOR.
</hard-boundaries>

<purpose>
Analyze an existing project's structure, tech stack, and deployment target to generate production-ready CI/CD pipelines. Provides GitHub Actions workflows for continuous integration (testing, linting, building) and continuous deployment to supported hosting targets.
</purpose>

<role>
BUILDER role. Generates infrastructure based on established decisions.
- WILL create GitHub Actions workflows
- WILL create deployment and rollback scripts
- WILL document required secrets
- Will NOT recommend different deployment targets
- Will NOT suggest tech stack changes
</role>

<output>
- .github/workflows/ci.yml (if CI requested)
- .github/workflows/deploy.yml (if CD requested)
- scripts/deploy.sh (for VPS/manual deployment targets)
- scripts/rollback.sh (for VPS deployments)
- CICD-SECRETS.md (documentation of required secrets)
</output>

---

<workflow>

<phase id="0" name="load-environment">
<description>Load environment context scoped to this skill</description>

<steps>
1. Attempt to read ~/.claude/environment.json
2. If not found: Note and proceed (graceful degradation)
3. If found:
   a. Read skill_data_access["ci-cd-implement"]
   b. Extract ONLY the listed paths:
      - deployment_options
      - external_accounts.github
   c. Do NOT reference out-of-scope data
</steps>

<environment-usage>
From environment registry, this skill uses:
- **deployment_options.allowed**: Valid deployment targets for CI/CD
- **deployment_options.not_using**: Platforms to never generate pipelines for
- **external_accounts.github**: GitHub account details (actions_available)

This skill does NOT access:
- infrastructure details (deploy-guide's scope for execution)
- credentials (only document what's needed, never actual values)
- skill_guidance (advisory skills' scope)
</environment-usage>

<graceful-degradation>
If environment.json is missing or inaccessible:
- Note: "Environment registry not found - using defaults"
- Proceed with project analysis and handoff documents
</graceful-degradation>
</phase>

<phase id="1" name="load-handoffs">
<action>Load and validate upstream JSON handoffs.</action>

<expected-handoffs>
1. .docs/deployment-strategy.json (from deployment-advisor) - REQUIRED
2. .docs/deployment-log.json (from deploy-guide) - OPTIONAL
3. .docs/project-foundation.json (from project-spinup) - OPTIONAL
</expected-handoffs>

<loading-process>
1. Scan .docs/ for expected handoff documents
2. Parse JSON handoffs and validate structure
3. Extract deployment target, storage strategy from deployment-strategy.json
4. If deployment-log.json exists, use it for verification
5. If handoffs missing: Gather equivalent information conversationally
</loading-process>

<when-handoffs-exist>
"I've loaded your deployment configuration:

**Deployment Target:** {target from deployment-strategy.json}
**Storage Strategy:** {storage from deployment-strategy.json}
**Already Deployed:** {yes/no based on deployment-log.json}

These are LOCKED decisions. I'll generate CI/CD pipelines for this target."
</when-handoffs-exist>

<when-handoffs-missing>
"I don't see .docs/deployment-strategy.json. Let me understand your deployment target.

Where does this project deploy?
1. **VPS with Docker** - Your Hostinger VPS using Docker Compose
2. **Cloudflare Pages** - Static/JAMstack deployment
3. **Fly.io** - Containerized full-stack deployment
4. **Hostinger Shared Hosting** - PHP + MySQL deployment
5. **Localhost** - No CI/CD needed (development only)

Which target? [1/2/3/4/5]"
</when-handoffs-missing>

<localhost-check>
If deployment target is localhost:
"This project deploys to localhost - CI/CD automation isn't applicable.

The workflow is complete for localhost projects. You can still add CI (testing/linting) if desired, but there's no deployment to automate.

Would you like CI only (automated testing on push)? [yes/no]"
</localhost-check>

<native-platform-check>
If .docs/deployment-strategy.json contains `"native_platform_handoff": true` OR platform is ios/macos/tauri:

"This is a **native platform project** ({platform}).

Native platform CI/CD requires specialized tooling:

**iOS/macOS:**
- Xcode Cloud (Apple's native CI/CD)
- fastlane + GitHub Actions (requires macOS runner)
- Code signing and provisioning profile management

**Tauri:**
- tauri-action GitHub Action for cross-platform builds
- Builds for macOS, Windows, and Linux from single workflow

**Status:** Native platform CI/CD is planned for a future enhancement. For now:

- **iOS/macOS:** Consider Xcode Cloud (free tier available) or manual fastlane setup
- **Tauri:** See [tauri-action](https://github.com/tauri-apps/tauri-action) for GitHub Actions workflow

Would you like CI only (automated testing on push)? This skill can set up:
- XCTest runs via `xcodebuild test` (requires macOS runner)
- Tauri frontend tests via Vitest/Jest + Rust tests via cargo test

[yes/no]"
</native-platform-check>

<key-principle>
This skill NEVER blocks on missing prerequisites. It gathers information conversationally and proceeds.
</key-principle>
</phase>

<phase id="2" name="analyze-project">
<action>Scan project to understand tech stack and existing configuration.</action>

<project-analysis>
Look for and analyze:

1. Package/dependency files:
   - package.json (Node.js/frontend)
   - requirements.txt / pyproject.toml (Python)
   - composer.json (PHP)
   - go.mod (Go)
   - Cargo.toml (Rust)

2. Existing CI/CD:
   - .github/workflows/ (existing GitHub Actions)
   - Dockerfile, docker-compose.yml

3. Test configuration:
   - Test commands in package.json scripts
   - pytest.ini, jest.config.js, vitest.config.ts
   - phpunit.xml

4. Linting/formatting:
   - .eslintrc, .prettierrc
   - ruff.toml, pyproject.toml [tool.ruff]
   - phpcs.xml

5. Type checking:
   - tsconfig.json (TypeScript)
   - mypy.ini, pyrightconfig.json (Python)

6. Build configuration:
   - next.config.js, vite.config.ts
   - Build scripts in package.json

7. Deployment configuration:
   - fly.toml (Fly.io)
   - wrangler.toml (Cloudflare)
   - Caddyfile references
</project-analysis>

<extract-information>
From analysis, determine:
- Primary language/framework
- Test command (e.g., npm test, pytest, phpunit)
- Lint command (e.g., npm run lint, ruff check)
- Type check command (e.g., npx tsc --noEmit)
- Build command (e.g., npm run build)
- Database type (if any)
- Environment variables needed
</extract-information>

<check-storage-strategy>
From deployment-strategy.json or user confirmation, determine storage approach:

1. **No storage** - Application doesn't handle file uploads
2. **Local VPS storage** - Files stored on VPS filesystem (no CI/CD secrets needed)
3. **Backblaze B2** - S3-compatible cloud storage (secrets needed)
4. **Tigris** - Fly.io native storage (auto-injected, no secrets needed)

Storage affects:
- Which secrets to document in CICD-SECRETS.md
- Whether environment variables need to be passed in workflows
</check-storage-strategy>
</phase>

<phase id="3" name="gather-preferences">
<action>Ask user what pipelines to generate.</action>

<prompt-to-user>
I've analyzed your project. Before generating pipelines, what would you like?

**Pipeline Options:**

1. **CI only** - Automated testing, linting, and builds on every push/PR
   Best for: Projects not ready for automated deployment, or deploying manually

2. **CD only** - Automated deployment to your hosting target
   Best for: Projects with existing CI, or simple projects where you trust manual testing

3. **Both CI + CD** - Complete pipeline from code push to deployment
   Best for: Most production projects

Which would you like? [1/2/3]
</prompt-to-user>

<store-preference>
Record user choice as: ci_only, cd_only, or both
</store-preference>
</phase>

<phase id="4" name="determine-complexity">
<action>Assess project complexity to determine environment strategy.</action>

<complexity-indicators>
Simple project (production only):
- Single developer
- Low traffic expectations
- No sensitive data
- Personal or small public project
- No existing staging setup

Complex project (staging + production):
- Multiple contributors
- Higher traffic expectations
- Handles user data or payments
- Existing staging/production separation
- deployment-strategy.md indicates professional uptime needs
</complexity-indicators>

<environment-decision>
Based on indicators:
- Simple: Generate single production deployment
- Complex: Generate staging + production with promotion workflow

If unclear from analysis, default to production-only for first implementation.
Note in output that staging can be added later.
</environment-decision>
</phase>

<phase id="5" name="generate-ci">
<action>Create GitHub Actions CI workflow if requested.</action>

<skip-condition>If user selected cd_only, skip to phase 6.</skip-condition>

<ci-components>
Include based on project analysis:
- Checkout code
- Set up language runtime (Node.js, Python, PHP, etc.)
- Install dependencies
- Run linting (if detected)
- Run type checking (if detected)
- Run tests (if detected)
- Run build (if detected)
- Cache dependencies for speed
</ci-components>

<ci-workflow-template>
```yaml
name: CI

on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Language-specific setup inserted here

      - name: Install dependencies
        run: {install_command}

      # Conditional steps based on detection:

      - name: Lint
        run: {lint_command}

      - name: Type check
        run: {typecheck_command}

      - name: Test
        run: {test_command}

      - name: Build
        run: {build_command}
```
</ci-workflow-template>

<language-setup-templates>

<nodejs>
```yaml
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
```
</nodejs>

<python>
```yaml
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: 'pip'
```
</python>

<php>
```yaml
      - name: Set up PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
          tools: composer
```
</php>

</language-setup-templates>
</phase>

<phase id="6" name="generate-cd">
<action>Create deployment workflow and scripts based on deployment target.</action>

<skip-condition>If user selected ci_only, skip to phase 7.</skip-condition>

<deployment-targets>

<cloudflare-pages>
<description>Static/JAMstack deployment via Wrangler CLI</description>

<workflow-template>
```yaml
name: Deploy to Cloudflare Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Deploy to Cloudflare Pages
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy {build_output_dir} --project-name={project_name}
```
</workflow-template>

<secrets-needed>
- CLOUDFLARE_API_TOKEN: API token with Pages edit permissions
- CLOUDFLARE_ACCOUNT_ID: Your Cloudflare account ID
</secrets-needed>
</cloudflare-pages>

<fly-io>
<description>Containerized deployment via Fly.io CLI</description>

<workflow-template>
```yaml
name: Deploy to Fly.io

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Fly.io CLI
        uses: superfly/flyctl-actions/setup-flyctl@master

      - name: Deploy to Fly.io
        run: flyctl deploy --remote-only
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```
</workflow-template>

<secrets-needed>
- FLY_API_TOKEN: Fly.io API token (flyctl tokens create deploy)
</secrets-needed>

<prerequisites>
- fly.toml must exist in project root
- App must be created: flyctl apps create {app-name}
</prerequisites>
</fly-io>

<vps-docker>
<description>Docker deployment to VPS via SSH</description>

<workflow-template>
```yaml
name: Deploy to VPS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to VPS
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USERNAME }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /var/www/{project_name}
            git pull origin main
            docker compose pull
            docker compose up -d --build
            docker system prune -f
```
</workflow-template>

<secrets-needed>
- VPS_HOST: VPS IP address or hostname
- VPS_USERNAME: SSH username (e.g., john)
- VPS_SSH_KEY: Private SSH key for authentication
</secrets-needed>

<deploy-script>
Also generate scripts/deploy.sh for manual deployment:

```bash
#!/bin/bash
set -e

echo "Deploying {project_name} to VPS..."

# SSH to VPS and deploy
ssh {username}@{host} << 'EOF'
  cd /var/www/{project_name}
  git pull origin main
  docker compose pull
  docker compose up -d --build
  docker system prune -f
  echo "Deployment complete!"
EOF
```
</deploy-script>

<rollback-script>
Generate scripts/rollback.sh:

```bash
#!/bin/bash
set -e

if [ -z "$1" ]; then
  echo "Usage: ./scripts/rollback.sh <commit-hash>"
  exit 1
fi

echo "Rolling back to $1..."

ssh {username}@{host} << EOF
  cd /var/www/{project_name}
  git fetch origin
  git checkout $1
  docker compose up -d --build
  echo "Rolled back to $1"
EOF
```
</rollback-script>
</vps-docker>

<hostinger-shared>
<description>PHP deployment via rsync/FTP</description>

<workflow-template>
```yaml
name: Deploy to Shared Hosting

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy via rsync
        uses: burnett01/rsync-deployments@6.0.0
        with:
          switches: -avzr --delete --exclude='.git' --exclude='.github' --exclude='.env'
          path: ./
          remote_path: ${{ secrets.REMOTE_PATH }}
          remote_host: ${{ secrets.FTP_HOST }}
          remote_user: ${{ secrets.FTP_USERNAME }}
          remote_key: ${{ secrets.SSH_KEY }}
```
</workflow-template>

<secrets-needed>
- FTP_HOST: Shared hosting server hostname
- FTP_USERNAME: FTP/SSH username
- SSH_KEY: Private key for SSH access (or use FTP credentials)
- REMOTE_PATH: Path on server (e.g., /home/user/public_html)
</secrets-needed>

<alternative-ftp>
If SSH not available, use FTP action instead:

```yaml
      - name: Deploy via FTP
        uses: SamKirkland/FTP-Deploy-Action@v4.3.4
        with:
          server: ${{ secrets.FTP_HOST }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          local-dir: ./
          server-dir: ${{ secrets.REMOTE_PATH }}
```
</alternative-ftp>
</hostinger-shared>

</deployment-targets>

<staging-production>
If complexity warrants staging + production:

<staging-workflow>
Deploy to staging on push to dev branch:
```yaml
on:
  push:
    branches: [dev]
```
Use staging-specific secrets (STAGING_* prefix).
</staging-workflow>

<production-workflow>
Deploy to production on push to main branch:
```yaml
on:
  push:
    branches: [main]
```
Use production secrets.
</production-workflow>

<promotion-note>
Document in README: "To promote staging to production, merge dev into main."
</promotion-note>
</staging-production>
</phase>

<phase id="7" name="document-secrets">
<action>Create CICD-SECRETS.md documenting all required secrets.</action>

<secrets-template>
```markdown
# CI/CD Secrets Configuration

This document lists the secrets required for the CI/CD pipelines.

## GitHub Actions Secrets

Add these secrets in your repository settings:
**Settings -> Secrets and variables -> Actions -> New repository secret**

### Required Secrets

| Secret Name | Description | How to Obtain |
|-------------|-------------|---------------|
{secrets_table}

{storage_secrets_section}

## Setup Instructions

{target_specific_instructions}

{storage_setup_instructions}

## Verification

After adding secrets, trigger a workflow run to verify configuration:

```bash
git commit --allow-empty -m "test: verify CI/CD pipeline"
git push
```

Check the Actions tab for workflow results.
```
</secrets-template>

<storage-secrets-templates>

<no-storage>
<!-- No storage secrets section needed -->
</no-storage>

<local-vps-storage>
### Storage Secrets

**Storage Type:** Local VPS Filesystem

No additional secrets required. Files are stored directly on the VPS at the configured upload directory.

Note: Ensure the upload directory exists and has correct permissions on the VPS.
</local-vps-storage>

<backblaze-b2-storage>
### Storage Secrets

**Storage Type:** Backblaze B2

| Secret Name | Description | How to Obtain |
|-------------|-------------|---------------|
| B2_APPLICATION_KEY_ID | Backblaze B2 key ID | B2 Dashboard -> App Keys -> Add a New Application Key |
| B2_APPLICATION_KEY | Backblaze B2 application key | Generated when creating the key (save immediately) |
| B2_BUCKET_NAME | Your B2 bucket name | B2 Dashboard -> Buckets |
| B2_ENDPOINT | S3-compatible endpoint URL | B2 Dashboard -> Bucket details (e.g., `https://s3.us-west-004.backblazeb2.com`) |

**Note:** These secrets use S3-compatible naming. Your application may expect `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, etc. Map accordingly in your workflow or application config.
</backblaze-b2-storage>

<tigris-storage>
### Storage Secrets

**Storage Type:** Tigris (Fly.io Native)

No additional GitHub secrets required for Tigris storage.

Tigris credentials are automatically injected into your Fly.io application as environment variables:
- `AWS_ACCESS_KEY_ID` - Auto-set by Fly.io
- `AWS_SECRET_ACCESS_KEY` - Auto-set by Fly.io
- `AWS_ENDPOINT_URL_S3` - Auto-set by Fly.io
- `BUCKET_NAME` - Auto-set by Fly.io

**Setup:** Create Tigris bucket via `flyctl storage create` (done during initial deployment).
</tigris-storage>

</storage-secrets-templates>

<target-instructions>

<cloudflare-instructions>
### Cloudflare Pages Setup

1. Log in to Cloudflare Dashboard
2. Go to **My Profile -> API Tokens -> Create Token**
3. Use "Edit Cloudflare Workers" template or create custom with Pages permissions
4. Copy the token as CLOUDFLARE_API_TOKEN
5. Find Account ID in dashboard URL or **Workers & Pages -> Overview** sidebar
</cloudflare-instructions>

<fly-instructions>
### Fly.io Setup

1. Install Fly CLI: `curl -L https://fly.io/install.sh | sh`
2. Authenticate: `flyctl auth login`
3. Create deploy token: `flyctl tokens create deploy -x 999999h`
4. Copy the token as FLY_API_TOKEN
</fly-instructions>

<vps-instructions>
### VPS (Hostinger) Setup

1. Generate SSH key pair (if not exists):
   ```bash
   ssh-keygen -t ed25519 -C "github-actions-deploy"
   ```
2. Add public key to VPS:
   ```bash
   ssh-copy-id -i ~/.ssh/id_ed25519.pub john@your-vps-ip
   ```
3. Copy private key content as VPS_SSH_KEY
4. Set VPS_HOST to your VPS IP or hostname
5. Set VPS_USERNAME to your SSH username (john)
</vps-instructions>

<shared-instructions>
### Shared Hosting Setup

1. Get FTP/SSH credentials from Hostinger hPanel
2. Set FTP_HOST to the server hostname
3. Set FTP_USERNAME to your FTP username
4. For SSH: Generate and add key pair, copy private key as SSH_KEY
5. For FTP: Set FTP_PASSWORD (less secure than SSH)
6. Set REMOTE_PATH to your document root (e.g., /home/user/public_html)
</shared-instructions>

</target-instructions>

<storage-setup-instructions>

<backblaze-b2-setup>
### Backblaze B2 Storage Setup

1. Log in to Backblaze B2 Dashboard
2. **Create bucket** (if not exists):
   - Go to Buckets -> Create a Bucket
   - Choose bucket name and privacy settings
3. **Create application key:**
   - Go to App Keys -> Add a New Application Key
   - Restrict to your bucket for security
   - **Save the key immediately** - it's only shown once
4. **Find your endpoint:**
   - Go to Buckets -> click your bucket
   - Find "Endpoint" in bucket details
   - Format: `https://s3.{region}.backblazeb2.com`
5. **Add secrets to GitHub:**
   - B2_APPLICATION_KEY_ID: The keyID from step 3
   - B2_APPLICATION_KEY: The applicationKey from step 3
   - B2_BUCKET_NAME: Your bucket name
   - B2_ENDPOINT: The endpoint URL from step 4

**Cloudflare CDN (optional):**
If using Cloudflare in front of B2 for free egress, configure Cloudflare to proxy B2. See deployment-advisor documentation for details.
</backblaze-b2-setup>

<tigris-setup>
### Tigris Storage Setup (Fly.io)

Tigris storage is native to Fly.io and requires no GitHub secrets.

1. **Create bucket** (if not done during deployment):
   ```bash
   flyctl storage create
   ```
   Follow prompts to name your bucket.

2. **Verify credentials injected:**
   ```bash
   flyctl secrets list
   ```
   Should show AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, etc.

3. **Redeploy** if bucket was created after initial deployment:
   ```bash
   flyctl deploy
   ```

Credentials are automatically available to your application - no additional configuration needed.
</tigris-setup>

<local-vps-setup>
### Local VPS Storage Setup

No GitHub secrets needed for local storage.

**Ensure on VPS:**
1. Upload directory exists:
   ```bash
   sudo mkdir -p /var/www/{project_name}/uploads
   sudo chown {username}:{username} /var/www/{project_name}/uploads
   chmod 755 /var/www/{project_name}/uploads
   ```

2. Docker volume mount configured in docker-compose.yml:
   ```yaml
   volumes:
     - ./uploads:/app/uploads
   ```

3. Application configured with correct upload path in .env
</local-vps-setup>

</storage-setup-instructions>
</phase>

<phase id="8" name="summarize">
<action>Present what was created and provide next steps.</action>

<summary-template>
## CI/CD Implementation Complete

**Project:** {project_name}
**Deployment Target:** {deployment_target}
**Pipelines Generated:** {ci_and_or_cd}

---

### Files Created

{files_list}

---

### Workflow Status

**WORKFLOW TERMINATION POINT - FULL AUTOMATION**

Your project now has complete CI/CD automation:
- Automated testing on every push/PR (if CI generated)
- Automated deployment to {target} (if CD generated)

This completes the Skills workflow.

---

### Next Steps

1. **Review generated workflows**
   - Check `.github/workflows/` files
   - Verify commands match your project

2. **Configure secrets**
   - Open `CICD-SECRETS.md` for instructions
   - Add secrets in GitHub repository settings

3. **Test the pipeline**
   ```bash
   git add .
   git commit -m "ci: add CI/CD pipeline"
   git push
   ```

4. **Monitor first run**
   - Go to repository -> Actions tab
   - Watch workflow execution
   - Debug any failures

{additional_notes}

---

Happy deploying!
</summary-template>

<additional-notes-options>
- For VPS: "Ensure /var/www/{project_name} exists and has correct permissions"
- For staging+production: "Staging deploys from dev branch, production from main"
- For first deployment: "You may need to manually deploy once to initialize the environment"
</additional-notes-options>
</phase>

</workflow>

---

<guardrails>

<must-do>
- Analyze project before generating anything
- Ask about CI/CD preference before generating
- Generate only what was requested (CI, CD, or both)
- Include secrets documentation for every deployment
- Use project-specific values (not placeholders) where detectable
- Make scripts executable (include shebang, set -e)
- Include rollback capability for VPS deployments
- Note prerequisites (fly.toml, initial app creation, etc.)
- Read .docs/deployment-strategy.md if it exists
- Check storage strategy and include storage secrets in CICD-SECRETS.md when applicable
- Document Backblaze B2 secrets for VPS/Fly.io deployments using B2
- Note that Tigris credentials are auto-injected (no secrets needed)
</must-do>

<must-not-do>
- Generate pipelines for localhost deployment target
- Include actual secret values in generated files
- Skip project analysis phase
- Generate staging+production for simple projects without justification
- Assume deployment target without checking project
- Generate deployment to targets not in the supported list
- Skip storage secrets documentation when storage strategy requires it
- Recommend Tigris for non-Fly.io deployments (it's Fly.io native)
</must-not-do>

</guardrails>

---

<outputs>
**.github/workflows/ci.yml** — Continuous integration workflow (if CI requested):
- Automated testing, linting, type checking on push/PR
- Language-specific setup and dependency caching

**.github/workflows/deploy.yml** — Continuous deployment workflow (if CD requested):
- Automated deployment to configured target
- Target-specific deployment commands

**scripts/deploy.sh, scripts/rollback.sh** — Manual deployment scripts (for VPS targets)

**CICD-SECRETS.md** — Documentation of required GitHub secrets with setup instructions

**Flexible Entry:** This skill analyzes project structure to generate appropriate pipelines. Reads deployment-strategy.json for target configuration but gathers missing info conversationally.

**Note:** CI/CD is not applicable for localhost projects.
</outputs>
