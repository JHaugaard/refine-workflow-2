---
name: deploy-guide
description: "Guide user through actual deployment steps for their application. This skill should be used when a project is ready to deploy to production. Walks through pre-deployment checks, deployment execution, and post-deployment verification. Supports VPS/Docker, Cloudflare Pages, fly.io, and Hostinger Shared Hosting."
allowed-tools:
  - Read
  - Grep
  - Glob
  - Write
  - Bash
  - WebFetch
---

# deploy-guide

<hard-boundaries>
BEFORE ANY OUTPUT, I MUST VERIFY:

1. I will NOT change deployment strategy - deployment-advisor already decided
2. I will NOT suggest alternative hosting platforms - use what was chosen
3. I will NOT modify tech stack decisions - those are locked
4. I will ONLY execute deployment to the chosen target
5. I will USE environment registry data for infrastructure context

If the deployment strategy seems problematic, I will ASK THE USER before proceeding differently.

MY SCOPE: Execute deployment steps based on LOCKED decisions from deployment-advisor. I am an EXECUTOR and GUIDE, not an ADVISOR.
</hard-boundaries>

<purpose>
Guide the user step-by-step through deploying their application to the chosen hosting target. Performs pre-deployment checks, provides deployment commands, and verifies successful deployment. Creates deployment documentation for future reference.
</purpose>

<role>
BUILDER role with GUIDE approach. Executes deployment steps with user confirmation.
- WILL run deployment commands (with user approval)
- WILL perform pre-deployment checks
- WILL verify deployment success
- WILL create deployment documentation
- WILL troubleshoot common deployment issues
</role>

<output>
- Deployed application
- .docs/deployment-log.json (structured deployment record)
- Post-deployment verification results
</output>

---

<workflow>

<phase id="0" name="load-environment">
<description>Load environment context scoped to this skill</description>

<steps>
1. Attempt to read ~/.claude/environment.json
2. If not found: Note and proceed (graceful degradation)
3. If found:
   a. Read skill_data_access["deploy-guide"]
   b. Extract ONLY the listed paths:
      - infrastructure
      - credentials
      - established_choices
   c. Do NOT reference out-of-scope data
</steps>

<environment-usage>
From environment registry, this skill uses:
- **infrastructure.vps**: VPS details (hostname, IP, SSH user, specs, paths)
- **infrastructure.services**: Running services for coordination (Caddy config, ports)
- **credentials.domains**: Domain configuration for deployment
- **credentials.api_endpoints**: Service URLs for verification
- **established_choices**: Locked decisions (proxy: caddy, containerization: docker)

This skill does NOT access:
- deployment_options (deployment-advisor's scope)
- skill_guidance (advisory skills' scope)
- database_options (tech-stack-advisor's scope)
</environment-usage>

<graceful-degradation>
If environment.json is missing or inaccessible:
- Note: "Environment registry not found - using defaults"
- Proceed with information gathered conversationally
- Ask for infrastructure details as needed
</graceful-degradation>
</phase>

<phase id="1" name="load-handoffs">
<action>Load and validate upstream JSON handoffs.</action>

<expected-handoffs>
1. .docs/deployment-strategy.json (from deployment-advisor)
2. .docs/project-foundation.json (from project-spinup, optional)
</expected-handoffs>

<loading-process>
1. Scan .docs/ for expected handoff documents
2. Parse JSON handoffs and validate structure
3. Extract deployment target, storage strategy, and locked decisions
4. If handoffs missing: Gather equivalent information conversationally
5. Summarize loaded context before proceeding
</loading-process>

<when-handoffs-exist>
"I've loaded your deployment strategy:

**Deployment Target:** {target from deployment-strategy.json}
**Storage Strategy:** {storage from deployment-strategy.json}
**VPS:** {vps details if applicable}

These are LOCKED decisions from the deployment-advisor phase.

Let me verify your project is ready for deployment, then we'll proceed step by step."
</when-handoffs-exist>

<when-handoffs-missing>
"I don't see .docs/deployment-strategy.json. No problem - let me understand your deployment target.

Where are you deploying?
1. **VPS with Docker** - Your Hostinger VPS using Docker Compose
2. **Cloudflare Pages** - Static/JAMstack deployment
3. **Fly.io** - Containerized full-stack deployment
4. **Hostinger Shared Hosting** - PHP + MySQL deployment

Which target? [1/2/3/4]"
</when-handoffs-missing>

<gather-additional-info>
Based on target, gather:
- VPS: Hostname/IP, SSH username, project path
- Cloudflare: Project name, build command, output directory
- Fly.io: App name, region preference
- Shared: FTP/SSH credentials, remote path
</gather-additional-info>

<gather-storage-info>
If storage strategy not in handoff document, ask:

"Does your application need object storage for files (uploads, images, documents)?

1. **No storage needed** - Application doesn't handle file uploads
2. **Local VPS storage** - Files stored on VPS filesystem
3. **Backblaze B2** - S3-compatible cloud storage (you have an account)
4. **Tigris** - Fly.io native storage (if deploying to Fly.io)

Which applies? [1/2/3/4]"

If storage is needed, gather:
- Storage provider choice
- Bucket name (for B2/Tigris)
- Whether credentials are ready
</gather-storage-info>

<key-principle>
This skill NEVER blocks on missing prerequisites. It gathers information conversationally and proceeds.
</key-principle>

<native-platform-check>
If .docs/deployment-strategy.json contains `"native_platform_handoff": true` OR platform is ios/macos/tauri:

"This is a **native platform project** ({platform}).

Native apps don't deploy to servers — they run directly on devices:

**iOS/macOS (SwiftUI):**
- Open project in Xcode
- Connect device via USB or enable wireless debugging
- Select device in Xcode toolbar
- Click Run (Cmd+R) or Archive for distribution

**Tauri (Desktop):**
- Build: `npm run tauri build`
- Executable created in `src-tauri/target/release/`
- Distribute the .app (macOS), .exe (Windows), or AppImage (Linux)

No server deployment needed. The **deploy-guide** skill is not applicable for native platforms.

Would you like guidance on:
1. **Building for release** — optimize and package your app
2. **Device testing** — run on physical devices
3. **Distribution** — share with others (TestFlight, direct install)

Or you can continue developing — your app runs locally via Xcode or `npm run tauri dev`."

[End workflow here for native platforms — do not proceed to pre-deployment checks]
</native-platform-check>
</phase>

<phase id="2" name="pre-deployment-checks">
<action>Verify project is ready for deployment.</action>

<checklist>
Pre-deployment Verification:

**Code Readiness:**
- [ ] All changes committed to git
- [ ] Working branch merged to main (or deploy branch)
- [ ] No uncommitted changes
- [ ] Build passes locally

**Configuration:**
- [ ] Environment variables documented
- [ ] Production config separate from development
- [ ] Secrets not committed to git

**Testing (if applicable):**
- [ ] Tests passing
- [ ] No critical bugs open

**Infrastructure:**
- [ ] Target environment accessible
- [ ] Required services running (database, etc.)
- [ ] DNS configured (if first deployment)

**Storage (if applicable):**
- [ ] Storage credentials available (B2 keys / Tigris via Fly.io)
- [ ] Bucket created (for B2/Tigris)
- [ ] Storage environment variables documented
- [ ] Upload directory exists and writable (for local VPS storage)
</checklist>

<verification-commands>
```bash
# Check git status
git status

# Verify on correct branch
git branch --show-current

# Check build passes
{build_command}

# Verify tests pass (if configured)
{test_command}
```
</verification-commands>

<prompt-to-user>
Running pre-deployment checks...

{checklist_results}

**Issues Found:** {count}
{issue_details}

Ready to proceed with deployment? [yes/no/fix issues]
</prompt-to-user>
</phase>

<phase id="3" name="deploy">
<action>Execute deployment based on target.</action>

<deployment-targets>

<vps-docker>
<name>VPS with Docker (Hostinger)</name>

<pre-steps>
1. Ensure Docker Compose file is ready
2. Verify SSH access to VPS
3. Confirm project directory exists on VPS
</pre-steps>

<deployment-process>
**Step 1: Connect to VPS**
```bash
ssh {username}@{host}
```

**Step 2: Navigate to project**
```bash
cd /var/www/{project_name}
```

**Step 3: Pull latest code**
```bash
git pull origin main
```

**Step 4: Build and restart containers**
```bash
docker compose pull
docker compose up -d --build
```

**Step 5: Verify containers running**
```bash
docker compose ps
```

**Step 6: Check application logs**
```bash
docker compose logs --tail=50
```

**Step 7: Clean up**
```bash
docker system prune -f
```
</deployment-process>

<first-deployment-extras>
For first-time deployment:

1. Create project directory:
   ```bash
   sudo mkdir -p /var/www/{project_name}
   sudo chown {username}:{username} /var/www/{project_name}
   ```

2. Clone repository:
   ```bash
   cd /var/www/{project_name}
   git clone {repo_url} .
   ```

3. Create production .env:
   ```bash
   cp .env.example .env
   nano .env  # Configure production values
   ```

4. Configure Caddy (reverse proxy):
   ```
   {domain} {
       reverse_proxy localhost:{port}
   }
   ```

5. Reload Caddy:
   ```bash
   sudo systemctl reload caddy
   ```
</first-deployment-extras>

<storage-setup-vps>
**Storage Configuration for VPS Deployment:**

[If Local VPS Storage:]

1. Create upload directory:
   ```bash
   sudo mkdir -p /var/www/{project_name}/uploads
   sudo chown {username}:{username} /var/www/{project_name}/uploads
   chmod 755 /var/www/{project_name}/uploads
   ```

2. Add to .env:
   ```bash
   UPLOAD_DIR=/var/www/{project_name}/uploads
   # Or for Docker, use volume mount path
   UPLOAD_DIR=/app/uploads
   ```

3. Ensure Docker volume mount in docker-compose.yml:
   ```yaml
   volumes:
     - ./uploads:/app/uploads
   ```

[If Backblaze B2:]

1. Create bucket in Backblaze B2 dashboard (if not exists)

2. Generate application key:
   - Go to App Keys in B2 dashboard
   - Create new key with access to your bucket
   - Save keyID and applicationKey securely

3. Add to production .env:
   ```bash
   # Backblaze B2 Configuration
   B2_APPLICATION_KEY_ID=your_key_id
   B2_APPLICATION_KEY=your_application_key
   B2_BUCKET_NAME=your_bucket_name
   B2_ENDPOINT=https://s3.us-west-004.backblazeb2.com  # Your region endpoint

   # S3-compatible format (for SDKs expecting AWS S3)
   AWS_ACCESS_KEY_ID=${B2_APPLICATION_KEY_ID}
   AWS_SECRET_ACCESS_KEY=${B2_APPLICATION_KEY}
   AWS_ENDPOINT_URL=${B2_ENDPOINT}
   S3_BUCKET=${B2_BUCKET_NAME}
   ```

4. Verify B2 endpoint matches your bucket region:
   - Check bucket details in B2 dashboard for correct endpoint

5. Test connection (optional, can defer to test-orchestrator):
   ```bash
   # Using aws-cli with B2
   aws s3 ls s3://{bucket_name} --endpoint-url={B2_ENDPOINT}
   ```

**Note:** For Cloudflare CDN in front of B2, that's a separate configuration step - see deployment-advisor references.
</storage-setup-vps>
</vps-docker>

<cloudflare-pages>
<name>Cloudflare Pages</name>

<pre-steps>
1. Ensure Cloudflare account connected
2. Verify build configuration
3. Confirm project name
</pre-steps>

<deployment-process>
**Option A: Git-Connected (Automatic)**

If connected to GitHub:
1. Push to main branch
2. Cloudflare automatically deploys
3. Monitor build in Cloudflare dashboard

**Option B: Direct Deploy (Manual)**

Using Wrangler CLI:
```bash
# Install/update Wrangler
npm install -g wrangler

# Login to Cloudflare
wrangler login

# Build project
{build_command}

# Deploy
wrangler pages deploy {output_dir} --project-name={project_name}
```
</deployment-process>

<first-deployment-extras>
For first-time deployment:

1. Create project in Cloudflare Pages dashboard
2. Connect to GitHub repository (recommended)
3. Configure build settings:
   - Build command: `npm run build`
   - Build output directory: `out` or `dist` or `.next`
4. Set environment variables in dashboard
</first-deployment-extras>
</cloudflare-pages>

<fly-io>
<name>Fly.io</name>

<pre-steps>
1. Ensure fly.toml exists
2. Verify Fly CLI installed and authenticated
3. Confirm app created
</pre-steps>

<deployment-process>
**Step 1: Verify fly.toml**
Ensure fly.toml is configured correctly.

**Step 2: Deploy**
```bash
flyctl deploy
```

**Step 3: Monitor deployment**
```bash
flyctl logs
```

**Step 4: Verify running**
```bash
flyctl status
```

**Step 5: Open application**
```bash
flyctl open
```
</deployment-process>

<first-deployment-extras>
For first-time deployment:

1. Install Fly CLI:
   ```bash
   curl -L https://fly.io/install.sh | sh
   ```

2. Authenticate:
   ```bash
   flyctl auth login
   ```

3. Create app:
   ```bash
   flyctl apps create {app_name}
   ```

4. Create fly.toml (or use `flyctl launch`)

5. Set secrets:
   ```bash
   flyctl secrets set DATABASE_URL="..."
   flyctl secrets set SECRET_KEY="..."
   ```

6. Create database (if needed):
   ```bash
   flyctl postgres create
   flyctl postgres attach
   ```
</first-deployment-extras>

<storage-setup-flyio>
**Storage Configuration for Fly.io Deployment:**

[If Tigris (Recommended for Fly.io):]

1. Create Tigris bucket:
   ```bash
   flyctl storage create
   ```
   Follow prompts to name your bucket.

2. Tigris credentials are auto-injected as secrets:
   - `AWS_ACCESS_KEY_ID` - Auto-set by Fly.io
   - `AWS_SECRET_ACCESS_KEY` - Auto-set by Fly.io
   - `AWS_ENDPOINT_URL_S3` - Auto-set by Fly.io
   - `BUCKET_NAME` - Auto-set by Fly.io

3. Verify secrets are set:
   ```bash
   flyctl secrets list
   ```

4. Your application can use standard S3 SDK with these environment variables.

5. Verify bucket created:
   ```bash
   flyctl storage list
   ```

**Tigris Environment Variables (auto-injected):**
```bash
AWS_ACCESS_KEY_ID=tid_...
AWS_SECRET_ACCESS_KEY=tsec_...
AWS_ENDPOINT_URL_S3=https://fly.storage.tigris.dev
AWS_REGION=auto
BUCKET_NAME=your-bucket-name
```

[If Backblaze B2 (alternative for Fly.io):]

1. Create bucket in Backblaze B2 dashboard (if not exists)

2. Generate application key in B2 dashboard

3. Set secrets in Fly.io:
   ```bash
   flyctl secrets set B2_APPLICATION_KEY_ID="your_key_id"
   flyctl secrets set B2_APPLICATION_KEY="your_application_key"
   flyctl secrets set B2_BUCKET_NAME="your_bucket_name"
   flyctl secrets set B2_ENDPOINT="https://s3.us-west-004.backblazeb2.com"

   # S3-compatible aliases (if your app expects these)
   flyctl secrets set AWS_ACCESS_KEY_ID="your_key_id"
   flyctl secrets set AWS_SECRET_ACCESS_KEY="your_application_key"
   flyctl secrets set AWS_ENDPOINT_URL="https://s3.us-west-004.backblazeb2.com"
   flyctl secrets set S3_BUCKET="your_bucket_name"
   ```

4. Verify secrets set:
   ```bash
   flyctl secrets list
   ```
</storage-setup-flyio>
</fly-io>

<hostinger-shared>
<name>Hostinger Shared Hosting</name>

<pre-steps>
1. Ensure FTP/SSH credentials available
2. Verify remote path
3. Confirm PHP version compatibility
</pre-steps>

<deployment-process>
**Option A: Using Git (if SSH available)**

```bash
# SSH to server
ssh {username}@{host}

# Navigate to public_html
cd ~/public_html/{subdirectory}

# Pull latest code
git pull origin main

# Install dependencies (if composer)
composer install --no-dev --optimize-autoloader
```

**Option B: Using rsync**

```bash
rsync -avz --delete \
  --exclude='.git' \
  --exclude='.env' \
  --exclude='node_modules' \
  ./ {username}@{host}:~/public_html/{subdirectory}/
```

**Option C: Using FTP**

Use FileZilla or similar:
1. Connect to FTP server
2. Navigate to public_html
3. Upload files (excluding .env, node_modules, .git)
</deployment-process>

<first-deployment-extras>
For first-time deployment:

1. Create subdirectory (if not root):
   ```bash
   mkdir -p ~/public_html/{subdirectory}
   ```

2. Create .htaccess (for PHP routing):
   ```
   RewriteEngine On
   RewriteCond %{REQUEST_FILENAME} !-f
   RewriteCond %{REQUEST_FILENAME} !-d
   RewriteRule ^(.*)$ index.php [QSA,L]
   ```

3. Configure database in cPanel
4. Create production .env on server
5. Set correct file permissions:
   ```bash
   find . -type f -exec chmod 644 {} \;
   find . -type d -exec chmod 755 {} \;
   ```
</first-deployment-extras>
</hostinger-shared>

</deployment-targets>

<execution-approach>
For each deployment step:
1. Show the command about to run
2. Ask for confirmation before executing
3. Show output
4. Verify success before proceeding
5. Offer to stop if errors occur
</execution-approach>
</phase>

<phase id="4" name="post-deployment-verification">
<action>Verify deployment was successful.</action>

<verification-checks>
**Accessibility:**
- [ ] Application loads at expected URL
- [ ] No 500 errors on main pages
- [ ] Static assets loading correctly

**Functionality:**
- [ ] Authentication works (if applicable)
- [ ] Database connections working
- [ ] API endpoints responding

**Performance:**
- [ ] Reasonable load time
- [ ] No console errors
- [ ] SSL certificate valid

**Storage (if applicable):**
- [ ] Storage credentials loaded by application
- [ ] Can connect to storage (B2/Tigris)
- [ ] Upload directory writable (local VPS)

Note: Full storage integration testing should use **test-orchestrator** skill.
</verification-checks>

<verification-commands>
```bash
# Check HTTP response
curl -I https://{domain}

# Check SSL certificate
curl -vI https://{domain} 2>&1 | grep "SSL certificate"

# Check specific endpoints
curl https://{domain}/api/health
```
</verification-commands>

<prompt-to-user>
Verifying deployment...

{verification_results}

**Status:** {SUCCESS/ISSUES_FOUND}
{details}

{if success}
Your application is live at: https://{domain}

{if issues}
Issues detected. Would you like help troubleshooting? [yes/no]
</prompt-to-user>
</phase>

<phase id="5" name="create-deployment-log">
<action>Create .docs/deployment-log.json structured handoff.</action>

<json-schema>
{
  "document_type": "deployment-log",
  "version": "1.0",
  "created": "[YYYY-MM-DD]",
  "project": "[project-name]",

  "rationale": "[1-2 sentence summary of deployment execution]",

  "latest_deployment": {
    "date": "[YYYY-MM-DD HH:MM]",
    "target": "[deployment target ID]",
    "branch": "[git branch]",
    "commit": "[commit hash]",
    "status": "SUCCESS | FAILED",
    "deployed_by": "[user]"
  },

  "deployment_target": {
    "type": "[vps-docker | fly-io | cloudflare-pages | hostinger-shared]",
    "host": "[hostname or service name]",
    "url": "[production URL]",
    "api_url": "[API URL if applicable]"
  },

  "pre_deployment_checks": {
    "code_committed": true,
    "build_passed": true,
    "tests_passed": true,
    "environment_configured": true
  },

  "post_deployment_verification": {
    "application_accessible": true,
    "no_errors_in_logs": true,
    "core_functionality_working": true,
    "storage_connected": true
  },

  "storage": {
    "type": "[none | local-vps | backblaze-b2 | tigris]",
    "bucket_name": "[bucket name if applicable]",
    "endpoint": "[endpoint URL if applicable]",
    "env_vars": ["[list of env var names]"]
  },

  "runbook": {
    "deploy_command": "[quick deploy command]",
    "rollback_command": "[rollback command]",
    "log_command": "[command to view logs]"
  },

  "environment_variables": [
    {"name": "[VAR_NAME]", "description": "[description]", "location": "[where to set]"}
  ],

  "deployment_history": [
    {"date": "[YYYY-MM-DD]", "commit": "[hash]", "status": "SUCCESS", "notes": "[notes]"}
  ],

  "workflow_status": {
    "completed_phases": ["project-brief-writer", "tech-stack-advisor", "deployment-advisor", "project-spinup", "deploy-guide"],
    "is_termination_point": true,
    "next_phases": ["ci-cd-implement"]
  },

  "handoff_to": ["ci-cd-implement"]
}
</json-schema>

<save-location>.docs/deployment-log.json</save-location>

<additional-output>
Also create a human-readable runbook summary in the terminal output for immediate reference.
</additional-output>
</phase>

<phase id="6" name="summarize">
<action>Provide deployment summary and next steps.</action>

<summary-template>
## Deployment Complete

**Application:** {project_name}
**Target:** {deployment_target}
**URL:** https://{domain}
**Status:** SUCCESS

---

### Deployment Record

Created: .docs/deployment-log.md

This file contains:
- Deployment runbook for future deployments
- Rollback procedure
- Environment variables reference
- Troubleshooting guide

---

### Workflow Status

**TERMINATION POINT - MANUAL DEPLOYMENT**

Your application is deployed. You can stop here if you don't need CI/CD automation.

**Next Options:**
1. **Stop here** - Use .docs/deployment-log.md for future manual deployments
2. **Add CI/CD** - Use **ci-cd-implement** skill to automate deployments

---

### For Future Deployments

Quick deployment:
```bash
{quick_deploy_command}
```

See .docs/deployment-log.md for full runbook.

---

### Monitoring Recommendations

- Check logs regularly: `{log_command}`
- Monitor uptime with external service
- Set up alerts for errors

---

Congratulations on your deployment!
</summary-template>
</phase>

</workflow>

---

<troubleshooting>

<common-issues>

<docker-issues>
**Container won't start:**
```bash
# Check logs
docker compose logs {service_name}

# Rebuild from scratch
docker compose down
docker compose build --no-cache
docker compose up -d
```

**Port already in use:**
```bash
# Find process using port
sudo lsof -i :{port}

# Kill process or change port in docker-compose.yml
```

**Out of disk space:**
```bash
# Clean up Docker
docker system prune -a --volumes
```
</docker-issues>

<networking-issues>
**Application not accessible:**
1. Check if container is running: `docker compose ps`
2. Test locally: `curl localhost:{port}`
3. Check Caddy/reverse proxy logs
4. Verify firewall allows traffic: `sudo ufw status`

**SSL certificate not working:**
1. Verify DNS points to server
2. Check Caddy logs: `sudo journalctl -u caddy -f`
3. Wait for certificate propagation (up to 15 minutes)
</networking-issues>

<database-issues>
**Connection refused:**
1. Check database container: `docker compose ps db`
2. Verify DATABASE_URL format
3. Check network: containers must be on same Docker network

**Permission denied:**
1. Verify user credentials in .env
2. Check database user has required permissions
</database-issues>

<cloudflare-issues>
**Build failed:**
1. Check build command matches local
2. Verify Node.js version in dashboard
3. Check build logs in Cloudflare dashboard

**Environment variables:**
1. Set in Cloudflare Pages dashboard
2. Redeploy after changing
</cloudflare-issues>

<fly-issues>
**Deploy failed:**
1. Check fly.toml configuration
2. Verify app name matches
3. Check resource limits (memory, CPU)

**App crashing:**
1. Check logs: `flyctl logs`
2. Verify health check endpoint
3. Check memory usage: `flyctl status`
</fly-issues>

<storage-issues>
**Local VPS Storage Issues:**

*Permission denied on upload:*
1. Check directory ownership: `ls -la /var/www/{project_name}/uploads`
2. Fix ownership: `sudo chown -R {username}:{username} /var/www/{project_name}/uploads`
3. Fix permissions: `chmod 755 /var/www/{project_name}/uploads`
4. For Docker: verify volume mount in docker-compose.yml

*Uploads not persisting after container restart:*
1. Ensure volume mount exists in docker-compose.yml
2. Check volume is mounted correctly: `docker compose exec {service} ls -la /app/uploads`
3. Verify host directory exists and has correct permissions

---

**Backblaze B2 Issues:**

*Access denied / 403 error:*
1. Verify B2_APPLICATION_KEY_ID and B2_APPLICATION_KEY are correct
2. Check application key has access to the specific bucket
3. Verify bucket name is correct (case-sensitive)
4. Ensure key hasn't expired or been revoked

*Endpoint connection failed:*
1. Verify B2_ENDPOINT matches your bucket's region
2. Check bucket details in B2 dashboard for correct endpoint
3. Common endpoints:
   - US West: `https://s3.us-west-004.backblazeb2.com`
   - US East: `https://s3.us-east-005.backblazeb2.com`
   - EU Central: `https://s3.eu-central-003.backblazeb2.com`

*CORS errors (browser uploads):*
1. Configure CORS rules in B2 bucket settings
2. Add allowed origins for your domain

*Test B2 connection:*
```bash
# Using aws-cli
aws s3 ls s3://{bucket_name} \
  --endpoint-url={B2_ENDPOINT} \
  --profile b2  # if using named profile

# Or inline credentials
AWS_ACCESS_KEY_ID={key_id} AWS_SECRET_ACCESS_KEY={key} \
  aws s3 ls s3://{bucket_name} --endpoint-url={endpoint}
```

---

**Tigris Issues (Fly.io):**

*Credentials not available:*
1. Verify bucket was created: `flyctl storage list`
2. Check secrets are set: `flyctl secrets list`
3. Redeploy after creating storage: `flyctl deploy`

*Bucket not found:*
1. Verify bucket name in `flyctl storage list`
2. Check BUCKET_NAME environment variable matches

*Permission denied:*
1. Verify the Tigris bucket is attached to your app
2. Check `flyctl storage list` shows bucket associated with app

*Test Tigris connection:*
```bash
# SSH into Fly.io machine
flyctl ssh console

# Inside the machine, test with aws-cli or your app's storage test
env | grep AWS  # Verify credentials are set
env | grep BUCKET  # Verify bucket name is set
```

---

**General Storage Debugging:**

*Verify environment variables are loaded:*
```bash
# VPS/Docker
docker compose exec {service} env | grep -E "(B2_|AWS_|S3_|BUCKET)"

# Fly.io
flyctl ssh console -C "env | grep -E '(AWS_|BUCKET)'"
```

*Check application logs for storage errors:*
```bash
# VPS/Docker
docker compose logs {service} | grep -i "storage\|s3\|upload\|bucket"

# Fly.io
flyctl logs | grep -i "storage\|s3\|upload\|bucket"
```
</storage-issues>

</common-issues>

</troubleshooting>

---

<guardrails>

<must-do>
- Run pre-deployment checks before deploying
- Ask for confirmation before executing commands
- Verify deployment success
- Create deployment-log.md documentation
- Handle errors gracefully with troubleshooting guidance
- Provide rollback instructions
- Configure storage based on deployment-strategy.md (or gather conversationally)
- Document storage configuration in deployment log
- Verify storage credentials are set before deploying
</must-do>

<must-not-do>
- Deploy without user confirmation
- Skip pre-deployment checks
- Leave failed deployment without troubleshooting help
- Expose sensitive credentials in logs or documentation
- Skip post-deployment verification
- Skip storage setup when storage strategy is defined
- Log actual credential values (only document variable names)
</must-not-do>

</guardrails>

---

<workflow-status>
Phase 5 of 6: Deployment

Status:
  Phase 0: Project Brief (project-brief-writer)
  Phase 1: Tech Stack (tech-stack-advisor)
  Phase 2: Deployment Strategy (deployment-advisor)
  Phase 3: Project Foundation (project-spinup) <- TERMINATION POINT (localhost)
  Phase 4: Test Strategy (test-orchestrator) - optional, can run anytime
  Phase 5: Deployment (you are here) <- TERMINATION POINT (manual deploy)
  Phase 6: CI/CD (ci-cd-implement) <- TERMINATION POINT (full automation)
</workflow-status>

---

<integration-notes>

<workflow-position>
Phase 5 of 6 in the Skills workflow chain.
Expected input: Deployable project, .docs/deployment-strategy.json (gathered conversationally if missing)
Produces: Deployed application, .docs/deployment-log.json

This is a TERMINATION POINT for projects not needing CI/CD automation.
</workflow-position>

<flexible-entry>
This skill can be invoked standalone on any deployable project. It gathers deployment target information conversationally if not available in handoff documents.
</flexible-entry>

<when-to-invoke>
- When project development is complete (or MVP ready)
- When user is ready to deploy to production
- For subsequent deployments (use deployment-log.md as runbook)
</when-to-invoke>

<status-utility>
Users can invoke the **workflow-status** skill at any time to:
- See current workflow progress
- Check which phases are complete
- Get guidance on next steps
- Review all handoff documents

Mention this option when users seem uncertain about their progress.
</status-utility>

</integration-notes>
