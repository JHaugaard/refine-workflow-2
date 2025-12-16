# Project Spinup - Example Scenarios

This document shows concrete examples of what project-spinup generates for different tech stacks and modes.

---

## Example 1: Next.js Recipe Manager (Learning Mode)

### Input

```
Project: recipe-manager
Description: A recipe management app with user accounts, recipe CRUD, and sharing features
Tech Stack: Next.js 15 + TypeScript + Supabase + Tailwind
Hosting: Hostinger VPS with Docker
Complexity: standard
Learning Mode: true
Special Features: Authentication, image uploads, search
```

### Generated Structure

```
recipe-manager/
├── CLAUDE.md                    # 15KB - Comprehensive project context
├── docker-compose.yml           # Dev environment: Next.js app
├── .env.example                # Supabase URLs, keys, upload config
├── .gitignore                  # Node, Next.js, env files
├── README.md                   # Quick start instructions
├── src/                        # Empty - to be filled via Learning Mode
├── tests/                      # Empty - to be filled via Learning Mode
└── docs/                       # Empty - for project documentation
```

### CLAUDE.md Excerpt (Learning Mode Section)

```markdown
## Next Steps (Learning Mode)

You now have the project foundation. Complete the setup by asking Claude Code
to help you build out the structure step-by-step.

Estimated total time: 4-6 hours

---

### Step 1: Initialize Next.js App Router Structure
⏱️ ~20 minutes | 🎯 Learning: Next.js 15 App Router, TypeScript configuration

**What you'll learn**: How Next.js projects are organized, App Router vs Pages
Router, TypeScript strict mode benefits, and why this structure supports
development and deployment.

**Say to Claude Code**:
```
Set up the Next.js 15 App Router structure with TypeScript as specified in
CLAUDE.md. Include app/layout.tsx, app/page.tsx, and core configuration files.
Please explain the purpose of each major file as you create them.
```

**What will be created**:
- next.config.js (with image optimization, strict mode)
- tsconfig.json (strict TypeScript settings)
- tailwind.config.ts (custom theme with recipe app colors)
- postcss.config.js
- app/layout.tsx (root layout with providers)
- app/page.tsx (landing page)
- app/globals.css (Tailwind directives)

**Verification**:
```bash
npm install
npm run dev
# Open http://localhost:3000 - should see default landing page
```

**Before moving to Step 2**: Review the generated files. Notice:
- How layout.tsx wraps all pages
- TypeScript strict mode catches errors early
- Tailwind configuration uses CSS variables for theming

---

### Step 2: Configure Supabase Client
⏱️ ~15 minutes | 🎯 Learning: Supabase client patterns, environment variables

**What you'll learn**: How to securely connect to Supabase, difference between
client-side and server-side clients, and environment variable management.

**Say to Claude Code**:
```
Set up the Supabase client configuration with environment variables as
specified in CLAUDE.md. Include both client-side and server-side clients, and
add a middleware example.
```

**What will be created**:
- lib/supabase/client.ts (browser client)
- lib/supabase/server.ts (server-side client for Server Components)
- lib/supabase/middleware.ts (auth middleware)
- middleware.ts (Next.js middleware entry point)
- Updated .env.example with Supabase URLs and keys

**Verification**:
```bash
# Add your Supabase credentials to .env.local
npm run dev
# No errors should appear in console
```

---

### Step 3: Implement Authentication
⏱️ ~30 minutes | 🎯 Learning: Auth flows, protected routes, session management

**What you'll learn**: Email/password authentication, protected routes, session
persistence, and user context management in Next.js.

**Say to Claude Code**:
```
Implement authentication using Supabase Auth as specified in CLAUDE.md. Include
signup, login, logout pages, and a protected dashboard route. Show me the auth
flow and security considerations.
```

**What will be created**:
- app/auth/login/page.tsx
- app/auth/signup/page.tsx
- app/auth/callback/route.ts (OAuth callback)
- app/dashboard/page.tsx (protected route example)
- components/auth/AuthProvider.tsx
- components/auth/LoginForm.tsx
- components/auth/SignupForm.tsx
- lib/auth/actions.ts (server actions for auth)

**Verification**:
- Visit /auth/signup - create account
- Check Supabase dashboard - user should appear
- Visit /dashboard - should redirect to login if not authenticated
- Login - should access dashboard

---

### Step 4: Create Recipe CRUD Feature
⏱️ ~45 minutes | 🎯 Learning: Full-stack data flow, Server Actions, form handling

**What you'll learn**: How data flows from UI → Server Action → Database and back,
form validation, error handling, and TypeScript types for database models.

**Say to Claude Code**:
```
Create a complete CRUD feature for recipes that demonstrates best practices.
Include form validation, error handling, optimistic updates, and TypeScript
types. Show me the full data flow.
```

**What will be created**:
- app/recipes/page.tsx (recipe list)
- app/recipes/new/page.tsx (create recipe)
- app/recipes/[id]/page.tsx (view recipe)
- app/recipes/[id]/edit/page.tsx (edit recipe)
- components/recipes/RecipeForm.tsx
- components/recipes/RecipeCard.tsx
- lib/recipes/actions.ts (Server Actions)
- lib/recipes/queries.ts (database queries)
- lib/recipes/types.ts (TypeScript types)
- Database migration file

**Verification**:
- Run migration to create recipes table
- Visit /recipes/new - create a recipe
- View recipe list - should display your recipe
- Edit and delete - should work
- Check Supabase - data should persist

---

### Step 5: Add Image Upload for Recipes
⏱️ ~30 minutes | 🎯 Learning: File uploads, Supabase Storage, image optimization

**What you'll learn**: File upload patterns, image optimization with Next.js,
Supabase Storage buckets, and signed URLs for security.

**Say to Claude Code**:
```
Add image upload functionality to recipes using Supabase Storage. Include image
preview, validation (size, type), and optimization. Explain the security model
for file access.
```

**What will be created**:
- lib/storage/upload.ts (upload utilities)
- lib/storage/config.ts (bucket configuration)
- components/recipes/ImageUpload.tsx
- Updated RecipeForm with image field
- Storage bucket setup instructions

**Verification**:
- Create Supabase Storage bucket (instructions provided)
- Upload image when creating recipe
- Image should display in recipe card
- Delete recipe - image should be removed from storage

---

### Step 6: Implement Recipe Search
⏱️ ~30 minutes | 🎯 Learning: Full-text search, debouncing, query optimization

**What you'll learn**: PostgreSQL full-text search, search input debouncing,
query performance optimization, and UX patterns for search.

**Say to Claude Code**:
```
Implement recipe search functionality with PostgreSQL full-text search. Include
search by title, ingredients, and tags. Add debouncing and show loading states.
Explain the search strategy.
```

**What will be created**:
- components/recipes/SearchBar.tsx
- lib/recipes/search.ts (search queries)
- app/search/page.tsx (search results page)
- Database migration for search indexes

**Verification**:
- Type in search bar - should debounce
- Search for recipe title - should find matches
- Search for ingredient - should work
- Empty search - should show all recipes

---

### Step 7: Add Testing Suite
⏱️ ~30 minutes | 🎯 Learning: Testing strategies, test structure, mocking

**What you'll learn**: How to write effective tests for Next.js + Supabase apps,
when to use unit vs integration tests, and testing best practices.

**Say to Claude Code**:
```
Set up Jest and React Testing Library. Write example tests for the recipe CRUD
feature including form validation, API actions, and component rendering. Explain
the testing strategy.
```

**What will be created**:
- jest.config.js
- jest.setup.js
- tests/unit/recipes/RecipeForm.test.tsx
- tests/unit/recipes/actions.test.ts
- tests/integration/recipes/crud.test.ts
- __mocks__/supabase.ts (mock Supabase client)

**Verification**:
```bash
npm test
# All tests should pass
```

---

### Step 8: Configure Docker Development Environment
⏱️ ~15 minutes | 🎯 Learning: Docker containerization, docker-compose orchestration

**What you'll learn**: How Docker containers work, docker-compose for multi-service
apps, volume management, and why this improves development consistency.

**Say to Claude Code**:
```
Review and enhance the docker-compose.yml to ensure Next.js is properly
containerized with hot-reload. Explain how to use Docker for local development
and troubleshooting tips.
```

**What will be created**:
- Dockerfile.dev (development container)
- Updated docker-compose.yml
- .dockerignore
- Docker setup documentation

**Verification**:
```bash
docker compose up --build -d
docker compose ps  # Should show 'running'
docker compose logs -f app  # Should show Next.js dev server
```

---

### 🎓 You're Ready to Build!

After completing these steps, you'll have:
- ✅ Complete Next.js 15 App Router structure
- ✅ Supabase connected (auth, database, storage)
- ✅ Authentication working (signup, login, protected routes)
- ✅ Recipe CRUD with image uploads
- ✅ Search functionality
- ✅ Testing framework ready
- ✅ Docker environment running
- ✅ Clear development workflow

**Next**: Continue building features! Add:
- Recipe sharing with other users
- Recipe collections/favorites
- Recipe ratings and reviews
- Meal planning calendar
- Shopping list generation
- Recipe import from URLs

Use the CRUD example as a template and ask Claude Code for help implementing
each new feature.

**Remember**: Use conventional commits, write tests as you go, and commit
working increments frequently.
```

### Key Features of Learning Mode Output

1. **Progressive complexity**: Each step builds on previous
2. **Clear learning objectives**: Know what you're learning
3. **Concrete prompts**: Just copy-paste to Claude Code
4. **Verification steps**: Confirm each step works
5. **Explanations**: Understand why, not just what
6. **Realistic time estimates**: Plan your learning sessions

---

## Example 2: PHP Blog Platform (Quick Start Mode)

### Input

```
Project: simple-blog
Description: Traditional blog with posts, comments, and admin panel
Tech Stack: PHP 8.2 + MySQL + Simple MVC
Hosting: Hostinger Shared Hosting
Complexity: simple
Learning Mode: false
Special Features: Admin panel, markdown support
```

### Generated Structure (Complete Scaffolding)

```
simple-blog/
├── CLAUDE.md                   # Project context for Claude Code
├── docker-compose.yml          # PHP-Apache, MySQL, phpMyAdmin
├── .env.example               # Database credentials
├── .gitignore                 # PHP, vendor, env files
├── README.md                  # Setup instructions
├── composer.json              # Dependencies: markdown parser, etc.
├── composer.lock
├── index.php                  # Entry point
│
├── config/
│   ├── database.php           # Database connection with PDO
│   └── config.php             # App configuration
│
├── src/
│   ├── Router.php             # Simple router
│   ├── Database.php           # Database wrapper
│   │
│   ├── Controllers/
│   │   ├── BaseController.php
│   │   ├── PostController.php   # CRUD for posts
│   │   ├── CommentController.php
│   │   └── AdminController.php   # Admin panel
│   │
│   ├── Models/
│   │   ├── Post.php           # Post model with validation
│   │   ├── Comment.php
│   │   └── User.php
│   │
│   └── Views/
│       ├── layout.php         # Base layout
│       ├── home.php           # Post list
│       ├── post.php           # Single post view
│       ├── admin/
│       │   ├── dashboard.php
│       │   ├── posts.php
│       │   └── edit-post.php
│       └── partials/
│           ├── header.php
│           ├── footer.php
│           └── comment-form.php
│
├── public/
│   ├── index.php              # Public entry point
│   ├── css/
│   │   └── style.css          # Basic styles
│   └── js/
│       └── main.js            # Basic interactivity
│
├── tests/
│   ├── PostTest.php           # PHPUnit tests
│   ├── CommentTest.php
│   └── bootstrap.php
│
├── docs/
│   ├── database-schema.sql    # Database setup
│   └── deployment.md          # Hostinger deployment guide
│
└── scripts/
    └── seed.php               # Sample data
```

### Generated docker-compose.yml

```yaml
version: '3.8'

services:
  php:
    image: php:8.2-apache
    container_name: simple-blog-php
    ports:
      - "8000:80"
    volumes:
      - ./:/var/www/html
    depends_on:
      - mysql
    environment:
      - PHP_ENV=development

  mysql:
    image: mysql:8.0
    container_name: simple-blog-mysql
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_DATABASE}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./docs/database-schema.sql:/docker-entrypoint-initdb.d/schema.sql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: simple-blog-phpmyadmin
    ports:
      - "8080:80"
    environment:
      PMA_HOST: mysql
      PMA_USER: ${DB_USER}
      PMA_PASSWORD: ${DB_PASSWORD}
    depends_on:
      - mysql

volumes:
  mysql_data:
```

### Sample Generated Code (src/Controllers/PostController.php)

```php
<?php

namespace App\Controllers;

use App\Models\Post;
use App\Database;

class PostController extends BaseController
{
    private $postModel;

    public function __construct()
    {
        $db = Database::getInstance();
        $this->postModel = new Post($db);
    }

    /**
     * Display all published posts
     */
    public function index()
    {
        $posts = $this->postModel->getAllPublished();

        $this->render('home', [
            'title' => 'Blog Posts',
            'posts' => $posts
        ]);
    }

    /**
     * Display a single post
     * @param int $id Post ID
     */
    public function show($id)
    {
        $post = $this->postModel->findById($id);

        if (!$post) {
            $this->notFound();
            return;
        }

        // Get comments for this post
        $comments = $this->postModel->getComments($id);

        $this->render('post', [
            'title' => $post['title'],
            'post' => $post,
            'comments' => $comments
        ]);
    }

    /**
     * Create new post (admin only)
     */
    public function create()
    {
        $this->requireAdmin();

        if ($_SERVER['REQUEST_METHOD'] === 'POST') {
            $data = [
                'title' => $_POST['title'] ?? '',
                'content' => $_POST['content'] ?? '',
                'author_id' => $_SESSION['user_id']
            ];

            $errors = $this->postModel->validate($data);

            if (empty($errors)) {
                $postId = $this->postModel->create($data);
                $this->redirect('/post/' . $postId);
            } else {
                $this->render('admin/edit-post', [
                    'title' => 'Create Post',
                    'errors' => $errors,
                    'post' => $data
                ]);
            }
        } else {
            $this->render('admin/edit-post', [
                'title' => 'Create Post',
                'post' => []
            ]);
        }
    }

    // ... update, delete methods ...
}
```

### Quick Start Workflow

With everything generated, John can immediately:

```bash
# 1. Install dependencies
composer install

# 2. Configure environment
cp .env.example .env
# Edit .env with database credentials

# 3. Start Docker
docker compose up -d

# 4. Run database migrations
docker exec simple-blog-php php scripts/seed.php

# 5. Open in browser
open http://localhost:8000

# 6. Start coding features!
```

No setup steps needed - everything is ready to go.

---

## Example 3: FastAPI Task Manager (Learning Mode)

### Input

```
Project: task-manager-api
Description: REST API for task management with teams and assignments
Tech Stack: FastAPI + PostgreSQL + SQLAlchemy + Alembic
Hosting: Hostinger VPS with Docker
Complexity: standard
Learning Mode: true
Special Features: JWT auth, team collaboration, real-time updates
```

### Generated Structure

```
task-manager-api/
├── CLAUDE.md                  # Comprehensive context
├── docker-compose.yml         # FastAPI + PostgreSQL
├── .env.example              # Database URL, JWT secret
├── .gitignore                # Python, venv, __pycache__
├── README.md
├── requirements.txt          # FastAPI, SQLAlchemy, etc. (empty initially)
├── app/                      # Empty - Learning Mode
├── tests/                    # Empty - Learning Mode
└── docs/                     # Empty - will hold API docs
```

### CLAUDE.md Excerpt (Learning Mode Prompts)

```markdown
## Next Steps (Learning Mode)

Estimated total time: 5-7 hours

### Step 1: Initialize FastAPI Project Structure
⏱️ ~20 minutes | 🎯 Learning: FastAPI project organization, Python type hints

**Say to Claude Code**:
```
Set up the FastAPI project structure with main.py, routers, models, and schemas
as specified in CLAUDE.md. Explain the purpose of each directory and how FastAPI
uses type hints for automatic validation.
```

**What will be created**:
- main.py (FastAPI app entry point)
- app/__init__.py
- app/core/config.py (settings with pydantic)
- app/core/security.py (JWT utilities)
- app/api/ (routers directory)
- app/models/ (SQLAlchemy models)
- app/schemas/ (Pydantic schemas)
- app/db/ (database session management)
- requirements.txt (populated with dependencies)

**Verification**:
```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload
# Open http://localhost:8000/docs - should see Swagger UI
```

---

### Step 2: Configure PostgreSQL Database Connection
⏱️ ~20 minutes | 🎯 Learning: SQLAlchemy, connection pooling, async patterns

**Say to Claude Code**:
```
Set up SQLAlchemy async connection to PostgreSQL with connection pooling and
session management. Include Alembic for migrations. Explain async database
patterns in FastAPI.
```

**What will be created**:
- app/db/session.py (async session factory)
- app/db/base.py (declarative base)
- alembic.ini (migration configuration)
- alembic/ (migration directory)
- alembic/env.py (migration environment)

**Verification**:
```bash
docker compose up -d db
alembic revision --autogenerate -m "initial"
alembic upgrade head
# Check PostgreSQL - tables should be created
```

---

### Step 3: Implement JWT Authentication
⏱️ ~30 minutes | 🎯 Learning: JWT tokens, password hashing, dependency injection

**Say to Claude Code**:
```
Implement JWT authentication with registration, login, and token refresh. Include
password hashing with bcrypt and explain FastAPI's dependency injection for
protected routes.
```

**What will be created**:
- app/models/user.py
- app/schemas/user.py
- app/schemas/token.py
- app/api/auth.py (auth endpoints)
- app/core/deps.py (dependency injection)

**Verification**:
- POST /auth/register - create user
- POST /auth/login - get JWT token
- GET /auth/me (protected) - should require token
- Test with invalid token - should reject

---

### Step 4: Create Task CRUD Endpoints
⏱️ ~40 minutes | 🎯 Learning: REST API design, CRUD patterns, query parameters

[... continues with similar detailed steps ...]

---

### 🎓 You're Ready to Build!

After completing these steps, you'll have:
- ✅ FastAPI application structure
- ✅ PostgreSQL database connected
- ✅ JWT authentication working
- ✅ Task CRUD endpoints with validation
- ✅ Team collaboration features
- ✅ WebSocket for real-time updates
- ✅ API documentation (auto-generated)
- ✅ Testing suite with pytest

**Next**: Deploy to Hostinger VPS, add monitoring, implement advanced features.
```

---

## Comparison: Learning Mode vs Quick Start Mode

### Same Project, Different Modes

| Aspect | Learning Mode | Quick Start Mode |
|--------|---------------|------------------|
| **Initial Files** | 7 files (foundation only) | 50+ files (complete scaffold) |
| **Time to First Code** | After Step 1 (~20 min) | Immediate (0 min) |
| **Understanding Level** | Deep (explained step-by-step) | Surface (must explore on own) |
| **Total Setup Time** | 4-7 hours (spread over days) | 15-30 minutes (all at once) |
| **Learning Value** | High - teaches each layer | Medium - learn by reading code |
| **Customization** | Easy - build as you go | Harder - must understand existing code |
| **Best For** | First 2-3 projects in stack | Experienced with stack |
| **Overwhelm Factor** | Low - incremental | High - everything at once |
| **Control** | High - you choose what to add | Medium - must remove what you don't want |

### When to Switch from Learning to Quick Start

You're ready for Quick Start Mode when:
- ✅ You've completed 2-3 projects in Learning Mode
- ✅ You recognize patterns from previous projects
- ✅ You can explain each layer's purpose
- ✅ You're confident debugging the stack
- ✅ Setup steps feel routine, not educational

---

## Real-World Usage Examples

### Example A: John's First Next.js Project (Month 1)

**Context**: Never used Next.js before, coming from vanilla JS

**Approach**: Learning Mode

**Timeline**:
- Day 1: Steps 1-3 (structure, database, auth) - 2 hours
- Day 2: Steps 4-5 (CRUD, file upload) - 2 hours
- Day 3: Steps 6-8 (search, testing, Docker) - 2 hours
- Day 4+: Build actual features

**Outcome**: Deep understanding of Next.js patterns, confident to continue

---

### Example B: John's Third Next.js Project (Month 2)

**Context**: Built 2 Next.js apps already, familiar with patterns

**Approach**: Quick Start Mode

**Timeline**:
- Invoke skill: 2 minutes
- Review generated code: 15 minutes
- Customize for specific needs: 20 minutes
- Start building features: Immediate

**Outcome**: Productive immediately, knows what to modify

---

### Example C: Learning PHP (Month 3)

**Context**: Experienced with Next.js, learning PHP for first time

**Approach**: Back to Learning Mode (new stack)

**Timeline**:
- Steps 1-3: Understanding PHP patterns - 1.5 hours
- Steps 4-6: Building confidence - 2 hours
- Steps 7-8: Testing and deployment - 1 hour
- Built features: Using Next.js knowledge + PHP patterns

**Outcome**: Can compare paradigms, understands both

---

## Tips from Examples

### What Makes Learning Mode Effective

1. **Concrete prompts**: No guessing what to ask Claude Code
2. **Incremental complexity**: Each step builds on previous
3. **Verification steps**: Immediate feedback if something's wrong
4. **Learning objectives**: Clear purpose for each step
5. **Realistic timing**: Know how long each part takes

### What Makes Quick Start Effective

1. **Complete code examples**: See patterns in action
2. **Commented code**: Explains why choices were made
3. **Working tests**: Understand expected behavior
4. **Production-ready**: Not just hello world
5. **Easy to customize**: Well-organized for modifications

---

## Conclusion

The project-spinup skill adapts to your learning journey:

**Beginning**: Use Learning Mode extensively
**Intermediate**: Mix of both modes
**Advanced**: Primarily Quick Start, Learning for new concepts

The goal is progressive autonomy - start guided, end independent.

---

## Example 4: iOS Habit Tracker (Learning Mode) — Native Platform

### Input

```
Project: habit-tracker
Description: Personal habit tracking app with daily streaks and reminders
Tech Stack: SwiftUI + Supabase (from tech-stack-decision.json)
Platform: ios (from deployment-strategy.json native_platform)
Complexity: standard
Learning Mode: true
Special Features: Local notifications, streak tracking, simple auth
```

### Generated Structure

```
habit-tracker/
├── CLAUDE.md                    # SwiftUI-focused project context
├── README.md                    # Xcode setup instructions
├── .gitignore                   # Xcode/Swift ignores
├── .env.example                 # SUPABASE_URL, SUPABASE_ANON_KEY
├── HabitTracker/                # Main app target (to be created in Xcode)
│   ├── App/
│   │   └── HabitTrackerApp.swift
│   ├── Views/
│   │   ├── ContentView.swift
│   │   ├── HabitListView.swift
│   │   └── Components/
│   │       ├── HabitRow.swift
│   │       └── StreakBadge.swift
│   ├── Models/
│   │   └── Habit.swift
│   ├── Services/
│   │   └── SupabaseService.swift   # Pre-configured for homelab
│   ├── Utilities/
│   │   └── Config.swift            # Environment configuration
│   └── Resources/
│       └── Assets.xcassets
├── HabitTrackerTests/
├── docs/
│   └── setup-instructions.md    # Detailed Xcode walkthrough
└── .docs/
    └── project-foundation.json
```

### CLAUDE.md Excerpt (SwiftUI-Focused)

```markdown
# Habit Tracker - Project Context

## Developer Profile
- **Name**: John
- **Level**: Beginner-to-intermediate, learning SwiftUI
- **Goal**: Build a functional iOS app while learning Swift patterns
- **Approach**: Heavy reliance on Claude Code

---

## Project Overview

**Name**: Habit Tracker
**Platform**: iOS/iPadOS (SwiftUI)
**Backend**: Supabase (self-hosted on vps8)
**Distribution**: Personal device via Xcode

---

## SwiftUI Conventions

### Property Wrappers
- `@State` - Local view state (private, single view)
- `@Binding` - Parent-to-child two-way data flow
- `@Observable` - Observable class (iOS 17+, preferred over ObservableObject)
- `@Environment` - Environment values (colorScheme, dismiss, etc.)
- `@AppStorage` - UserDefaults persistence

### View Composition
- Extract reusable views into `Views/Components/`
- Use ViewBuilders for conditional content
- Prefer composition over inheritance
- Keep views focused (single responsibility)

### Navigation (iOS 16+)
- `NavigationStack` for push navigation
- `NavigationSplitView` for iPad sidebar layouts
- Use `.navigationDestination(for:)` for type-safe navigation

---

## Data Flow Pattern

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│    View     │ ──▶ │   Service    │ ──▶ │  Supabase   │
│  (SwiftUI)  │ ◀── │   Layer      │ ◀── │   Backend   │
└─────────────┘     └──────────────┘     └─────────────┘
      │
      ▼
┌─────────────┐
│   Model     │
│  (Codable)  │
└─────────────┘
```

- Views call Service methods
- Service handles Supabase communication
- Models are Codable structs
- Use async/await for all network calls

---

## Backend Integration (Supabase)

### Connection
```swift
// Services/SupabaseService.swift
import Supabase

class SupabaseService {
    static let shared = SupabaseService()

    let client: SupabaseClient

    private init() {
        client = SupabaseClient(
            supabaseURL: URL(string: Config.supabaseURL)!,
            supabaseKey: Config.supabaseAnonKey
        )
    }
}
```

### Environment Setup
Backend URL: https://supabase.haugaard.dev (homelab)
- Create `Config.swift` with URL and anon key
- Never commit actual keys (use .env.example as reference)

---

## Common Commands

```bash
# Build from command line
xcodebuild -scheme HabitTracker -destination 'platform=iOS Simulator,name=iPhone 15'

# Run tests
xcodebuild test -scheme HabitTracker -destination 'platform=iOS Simulator,name=iPhone 15'

# List available simulators
xcrun simctl list devices

# Open simulator
open -a Simulator
```

---

## Testing Approach

- **Unit Tests**: Test models and service logic (XCTest)
- **UI Tests**: Test critical user flows (XCUITest)
- **Previews**: Use SwiftUI previews for visual testing

```swift
#Preview {
    HabitRow(habit: .sample)
}
```

---

## Device Deployment

1. Connect iPhone/iPad via USB or Wi-Fi
2. In Xcode: Select your device from the device dropdown
3. Sign with Personal Team (free Apple ID)
4. Build and run (⌘R)
5. First run: Trust developer in Settings → General → VPN & Device Management

**Note**: Apps signed with free account expire after 7 days and need re-deployment.

---

## Next Steps (Guided Setup)

### Step 1: Create Xcode Project
⏱️ ~10 minutes | 🎯 Learning: Xcode project setup, SwiftUI basics

**What you'll learn**: How Xcode organizes iOS projects, SwiftUI app lifecycle,
and basic project configuration.

**Manual steps** (Xcode required):
1. Open Xcode → File → New → Project
2. Choose: iOS → App
3. Product Name: HabitTracker
4. Interface: SwiftUI
5. Language: Swift
6. Storage: None (we'll use Supabase)
7. Save to habit-tracker/ directory

**Then say to Claude Code**:
```
I've created the Xcode project for HabitTracker. Now help me:
1. Copy the generated file contents into the appropriate locations
2. Set up the project structure as defined in CLAUDE.md
3. Explain the SwiftUI app lifecycle (@main, App protocol, WindowGroup)
```

**Verification**:
- Build succeeds (⌘B)
- Run on simulator shows "Hello, world!" (⌘R)

---

### Step 2: Add Supabase Swift Package
⏱️ ~10 minutes | 🎯 Learning: Swift Package Manager, dependency management

**What you'll learn**: How to add external dependencies to an iOS project,
understand Package.swift manifests, and configure build settings.

**Manual steps**:
1. In Xcode: File → Add Package Dependencies
2. Enter URL: https://github.com/supabase-community/supabase-swift
3. Select version: Up to Next Major Version
4. Add to target: HabitTracker

**Then say to Claude Code**:
```
I've added the Supabase Swift package. Now help me:
1. Create Services/SupabaseService.swift with connection setup
2. Create Utilities/Config.swift for environment configuration
3. Explain how the Supabase Swift SDK differs from the JS version
```

**Verification**:
- Build succeeds with Supabase imported
- No linker errors

---

### Step 3: Implement Habit Model and Service
⏱️ ~20 minutes | 🎯 Learning: Codable, async/await, Supabase CRUD

**Say to Claude Code**:
```
Create the Habit model and SupabaseService methods as specified in CLAUDE.md:
1. Models/Habit.swift with Codable conformance
2. CRUD methods in SupabaseService (fetchHabits, createHabit, updateHabit, deleteHabit)
3. Show me how async/await works with Supabase calls
```

**What will be created**:
- Habit struct with id, name, streak, lastCompleted
- Async service methods
- Error handling patterns

**Verification**:
- Create test habit in Supabase dashboard
- Fetch habits in app → should display

---

### Step 4: Build Habit List View
⏱️ ~25 minutes | 🎯 Learning: List, ForEach, @State, navigation

**Say to Claude Code**:
```
Build the main HabitListView that displays habits from Supabase:
1. Use @State for local habits array
2. Fetch on .task modifier
3. Add pull-to-refresh
4. Include add button in toolbar
5. Show streak badge for each habit
```

**Verification**:
- List displays habits from database
- Pull-to-refresh works
- Add button shows (functionality in next step)

---

[Additional steps for: Add/Edit habit forms, streak logic, local notifications, etc.]

---

### 🎓 You're Ready to Build!

After completing these steps, you'll have:
- ✅ Working Xcode project with SwiftUI
- ✅ Supabase connected to homelab instance
- ✅ Habit CRUD functionality
- ✅ Understanding of SwiftUI data flow
- ✅ Service layer pattern for API calls

**Next**: Add local notifications for reminders, implement streak calculation,
add habit completion animation, sync across devices.
```

### Key Differences from Web Projects

| Aspect | Web Project | Native iOS Project |
|--------|-------------|-------------------|
| **Project creation** | Files generated directly | Xcode creates project, files copied in |
| **Dependencies** | npm/composer | Swift Package Manager (in Xcode) |
| **Dev server** | docker compose up | Xcode simulator/device |
| **Backend access** | fetch/axios | Supabase Swift SDK |
| **Environment vars** | .env files | Config.swift or xcconfig |
| **Testing** | Jest/PHPUnit | XCTest |
| **Deployment** | docker/hosting | Xcode → device |

---

## Example 5: macOS Menu Bar Utility (Quick Start Mode)

### Input

```
Project: quick-note
Description: Menu bar app for quick note capture with keyboard shortcut
Tech Stack: SwiftUI + PocketBase (from tech-stack-decision.json)
Platform: macos (from deployment-strategy.json native_platform)
Complexity: simple
Learning Mode: false
Special Features: Menu bar icon, global keyboard shortcut, simple sync
```

### Generated Structure (Complete Scaffold)

```
quick-note/
├── CLAUDE.md                    # macOS-focused project context
├── README.md                    # Setup instructions
├── .gitignore
├── .env.example                 # POCKETBASE_URL
├── QuickNote/
│   ├── App/
│   │   └── QuickNoteApp.swift   # MenuBarExtra setup
│   ├── Views/
│   │   ├── NoteInputView.swift  # Main capture view
│   │   ├── NotesListView.swift  # History popover
│   │   └── SettingsView.swift   # Preferences
│   ├── Models/
│   │   └── Note.swift           # Codable model
│   ├── Services/
│   │   └── PocketBaseService.swift  # Pre-configured
│   ├── Utilities/
│   │   ├── Config.swift
│   │   └── KeyboardShortcut.swift   # Global hotkey
│   └── Resources/
│       ├── Assets.xcassets
│       └── menubar-icon.png
├── QuickNoteTests/
│   └── NoteTests.swift
├── docs/
│   └── setup-instructions.md
└── .docs/
    └── project-foundation.json
```

### Generated Code Sample (QuickNoteApp.swift)

```swift
import SwiftUI

@main
struct QuickNoteApp: App {
    @State private var isPopoverShown = false

    var body: some Scene {
        // Menu bar app - no main window
        MenuBarExtra("Quick Note", systemImage: "note.text") {
            NoteInputView()
                .frame(width: 300, height: 200)
        }
        .menuBarExtraStyle(.window)

        // Settings window
        Settings {
            SettingsView()
        }
    }
}
```

### Generated Code Sample (PocketBaseService.swift)

```swift
import Foundation

actor PocketBaseService {
    static let shared = PocketBaseService()

    private let baseURL: URL

    private init() {
        // Uses homelab PocketBase instance
        baseURL = URL(string: Config.pocketbaseURL)!
    }

    func fetchNotes() async throws -> [Note] {
        let url = baseURL.appendingPathComponent("api/collections/notes/records")
        let (data, _) = try await URLSession.shared.data(from: url)
        let response = try JSONDecoder().decode(PocketBaseResponse<Note>.self, from: data)
        return response.items
    }

    func createNote(_ note: Note) async throws -> Note {
        var request = URLRequest(url: baseURL.appendingPathComponent("api/collections/notes/records"))
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try JSONEncoder().encode(note)

        let (data, _) = try await URLSession.shared.data(for: request)
        return try JSONDecoder().decode(Note.self, from: data)
    }

    // ... update, delete methods
}

struct PocketBaseResponse<T: Codable>: Codable {
    let items: [T]
    let totalItems: Int
}
```

### Quick Start Commands

```bash
# After creating Xcode project and copying files:
cd quick-note

# Initialize git
git init && git checkout -b main
git add . && git commit -m "chore: initial project setup via project-spinup"
git checkout -b dev

# Open in Xcode
open QuickNote.xcodeproj

# Build and run
# ⌘R in Xcode, or:
xcodebuild -scheme QuickNote -configuration Debug build

# The app appears in your menu bar, not the Dock
```

---

## Example 6: Tauri Desktop App (Learning Mode)

### Input

```
Project: markdown-editor
Description: Local-first markdown editor with live preview
Tech Stack: Tauri + React + TypeScript (from tech-stack-decision.json)
Platform: tauri (from deployment-strategy.json native_platform)
Complexity: standard
Learning Mode: true
Special Features: File system access, live preview, syntax highlighting
```

### Generated Structure

```
markdown-editor/
├── CLAUDE.md                    # Tauri + React context
├── README.md                    # Setup instructions
├── .gitignore                   # Rust + Node ignores
├── .env.example
├── package.json                 # React + Tauri dependencies
├── tsconfig.json
├── vite.config.ts
├── index.html
├── src/                         # React frontend
│   ├── main.tsx
│   ├── App.tsx
│   ├── components/
│   │   ├── Editor.tsx
│   │   ├── Preview.tsx
│   │   └── Toolbar.tsx
│   ├── hooks/
│   │   └── useFileSystem.ts     # Tauri FS integration
│   ├── services/
│   │   └── fileService.ts       # Tauri invoke wrappers
│   └── styles/
│       └── app.css
├── src-tauri/                   # Rust backend
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   ├── capabilities/
│   │   └── default.json         # File system permissions
│   └── src/
│       ├── main.rs
│       └── lib.rs               # Tauri commands
├── tests/
├── docs/
│   └── setup-instructions.md
└── .docs/
    └── project-foundation.json
```

### CLAUDE.md Excerpt (Tauri-Focused)

```markdown
# Markdown Editor - Project Context

## Tauri Architecture

```
┌─────────────────────────────────────────────┐
│              Your Application               │
├─────────────────────────────────────────────┤
│  Frontend (React)    │    Backend (Rust)    │
│  ─────────────────   │   ────────────────   │
│  • UI Components     │   • File system      │
│  • State mgmt        │   • Native APIs      │
│  • Event handling    │   • Commands         │
├─────────────────────────────────────────────┤
│              Tauri Core                     │
│  • IPC bridge        • Window management    │
│  • Security          • System tray          │
├─────────────────────────────────────────────┤
│           System Webview (WKWebView)        │
└─────────────────────────────────────────────┘
```

### Key Concepts

**Tauri Commands**: Rust functions callable from JavaScript
```rust
// src-tauri/src/lib.rs
#[tauri::command]
fn read_file(path: String) -> Result<String, String> {
    std::fs::read_to_string(&path).map_err(|e| e.to_string())
}
```

**Invoking from Frontend**:
```typescript
// src/services/fileService.ts
import { invoke } from '@tauri-apps/api/core';

export async function readFile(path: string): Promise<string> {
    return await invoke('read_file', { path });
}
```

---

## Development Workflow

```bash
# Start development (hot-reload for both frontend and Rust)
npm run tauri dev

# Build for production
npm run tauri build

# Output location
target/release/bundle/macos/Markdown Editor.app
```

---

## Next Steps (Guided Setup)

### Step 1: Initialize Tauri Project
⏱️ ~15 minutes | 🎯 Learning: Tauri CLI, project structure

**Say to Claude Code**:
```
Help me set up the Tauri project structure:
1. Run npm create tauri-app with React + TypeScript
2. Explain the src/ vs src-tauri/ separation
3. Walk through tauri.conf.json key settings
```

### Step 2: Implement File System Access
⏱️ ~20 minutes | 🎯 Learning: Tauri commands, capabilities, IPC

**Say to Claude Code**:
```
Add file system access for opening and saving markdown files:
1. Create Rust commands for read_file, write_file, pick_file
2. Set up capabilities in capabilities/default.json
3. Create TypeScript service layer to invoke commands
4. Explain the security model (why capabilities matter)
```

[Additional steps...]
```

---

## Native Platform Quick Reference

### When to Use Each Platform

| If the project needs... | Consider... |
|------------------------|-------------|
| iPhone/iPad app | iOS (SwiftUI) |
| Mac app with system integration | macOS (SwiftUI) |
| Desktop app using web skills | Tauri |
| Cross-platform mobile | PWA first, then consider native |

### Backend Integration Patterns

| Platform | Supabase | PocketBase |
|----------|----------|------------|
| iOS/macOS | supabase-swift SDK | Custom URLSession |
| Tauri | @supabase/supabase-js | pocketbase JS SDK |

### Distribution (Personal Device)

| Platform | Method |
|----------|--------|
| iOS | Xcode → USB/Wi-Fi to device |
| macOS | Build → drag to Applications |
| Tauri | `npm run tauri build` → .app bundle |

---

**Created**: 2025-11-04
**Updated**: 2025-12-12 (added native platform examples)
**For**: John's learning journey
**See Also**: QUICK_REFERENCE.md, SKILL.md
