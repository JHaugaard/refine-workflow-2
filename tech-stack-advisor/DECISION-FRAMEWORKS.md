# Decision Frameworks Reference

> **Purpose**: Detailed decision frameworks, tech stack patterns, and reference materials for tech-stack-advisor.
> Referenced by: SKILL.md Phase 3 (Analyze & Recommend)

---

## Enterprise vs Hacker Framework

### Purpose

Surface the tension between "Enterprise" and "Hacker" approaches to tech stacks. This is not about quality—both can produce excellent software. It's about philosophy, trade-offs, and what the user values.

### Spectrum Definition

#### Enterprise End

**Characteristics:**
- Strong typing, compile-time safety (TypeScript strict, Go, Rust, Java)
- Established frameworks with corporate backing (Spring, .NET, Angular)
- Explicit over implicit (verbose but predictable)
- Extensive documentation and enterprise support options
- Designed for large teams with varied skill levels
- Long-term maintainability prioritized over speed-to-ship
- Structured patterns (dependency injection, interface contracts)

**Examples:** TypeScript + Angular + NestJS, Java + Spring Boot, C# + .NET, Go + standard library

**Best for:** Projects that may grow to multiple developers, long maintenance horizons, regulated industries, risk-averse environments

#### Hacker End

**Characteristics:**
- Dynamic typing or type inference (Python, Ruby, JavaScript)
- Lightweight frameworks, minimal boilerplate (Flask, Sinatra, Express)
- Convention over configuration
- Single-developer productivity optimized
- "Move fast" ethos, iterate quickly
- Community-driven, often opinionated tools
- Pragmatic shortcuts acceptable

**Examples:** Python + Flask, Ruby + Sinatra, Node.js + Express, PHP + Laravel, SQLite + single-file backends

**Best for:** Solo developers, MVPs, learning projects, rapid prototyping, personal tools, "scratch your own itch" projects

#### Balanced Middle

**Characteristics:**
- Optional typing (TypeScript with moderate strictness, Python + type hints)
- Modern frameworks that balance productivity and structure (Next.js, FastAPI, SvelteKit)
- Good defaults with escape hatches
- Can scale from solo to small team
- Active communities with professional adoption

**Examples:** Next.js + TypeScript (moderate), FastAPI + Pydantic, SvelteKit, Laravel (PHP)

**Best for:** Projects that start small but might grow, learning with real-world applicability, balancing speed with maintainability

### When to Surface

Always include Enterprise vs Hacker analysis when:
- Recommending alternatives (show options across the spectrum)
- The project could reasonably be built either way
- User's learning goals align with one end of the spectrum
- Trade-offs between speed-to-ship and long-term maintainability are relevant

Frame as a choice, not a judgment. Neither end is "better"—they optimize for different things.

### Integration with Recommendations

In the Primary Recommendation and Alternatives:
- Label where each option falls: "Enterprise", "Balanced", or "Hacker"
- Explain WHY it falls there (specific characteristics)
- If recommending a Balanced option, note what Enterprise or Hacker alternatives exist
- Let user consciously choose based on their priorities

---

## Scoring Criteria

Rate each option 1-5:

| Criterion | What to Evaluate |
|-----------|------------------|
| **Capability Fit** | Feature support, does it do what you need? |
| **Ecosystem/Community** | Documentation quality, active maintenance, Claude's training data coverage |
| **Scalability Path** | Can it grow with the project if needed? |
| **Stack Philosophy** | Where it falls on Enterprise vs Hacker spectrum (note, don't score) |

---

## Recommendation Logic

### Simple Stack When

- Project has few features
- Straightforward CRUD operations

**Example:** Plain PHP + MySQL, simple MVC

### Modern JavaScript When

- Rich interactivity required
- Real-time features needed
- Project may grow in scope

**Example:** Next.js + Supabase, React + Node.js

### Traditional Framework When

- Convention over configuration preferred
- Ecosystem maturity important
- Well-documented patterns beneficial

**Example:** Laravel, Django, Ruby on Rails

### API-First When

- Mobile app planned
- Multiple frontends
- Microservices architecture

**Example:** FastAPI + PostgreSQL, Express + MongoDB

---

## Backend Tool Selection

### Supabase - Recommend When

- Relational database with advanced PostgreSQL features needed
- Auth + database + storage + realtime all needed
- Real-time subscriptions or WebSocket features required
- Vector embeddings needed (pgvector)
- Complex queries, full-text search, JSON operations
- Future scaling anticipated
- Row-level security beneficial

### PocketBase - Recommend When

- Authentication is primary need (minimal database use)
- Simple CRUD operations sufficient
- Embedded SQLite appropriate for scale
- Single-binary simplicity valued
- Project scope is small and well-defined

### PocketBase - Rule Out When

- Vector embeddings required (no pgvector equivalent)
- Complex relational queries needed
- Real-time subscriptions essential
- PostgreSQL-specific features required

---

## Ancillary Tools

### n8n - Recommend When

Brief mentions automation, workflows, integrations, scheduled tasks, data pipelines.

**Examples:** "automate user onboarding emails", "sync data between services"

### Ollama - Recommend When

Brief mentions embeddings, semantic search, RAG, AI features, content generation.

**Examples:** "semantic search over documents", "AI-powered recommendations"

### Wiki.js - Recommend When

Brief mentions documentation-heavy, knowledge base, team wiki, technical docs.

**Examples:** "internal knowledge base", "project documentation site"

---

## Common Patterns

### Content-Heavy Site

| Role | Option |
|------|--------|
| **Primary** | Next.js + Markdown/CMS |
| **Alternatives** | WordPress, Gatsby + Headless CMS, Static generators |

### SaaS Application

| Role | Option |
|------|--------|
| **Primary** | Next.js + Supabase |
| **Alternatives** | Laravel full-stack, FastAPI + React, Django full-stack |

### API-First

| Role | Option |
|------|--------|
| **Primary** | FastAPI + PostgreSQL |
| **Alternatives** | Node.js + Express, Laravel API-only, Django REST Framework |

### Real-Time Collaboration

| Role | Option |
|------|--------|
| **Primary** | Next.js + Supabase Realtime |
| **Alternatives** | Node.js + Socket.io + Redis, Phoenix/Elixir, Firebase |

### Data-Heavy Analytics

| Role | Option |
|------|--------|
| **Primary** | FastAPI + PostgreSQL + Pandas |
| **Alternatives** | Django + Celery, Node.js + PostgreSQL |

### Learning CRUD Project

| Role | Option |
|------|--------|
| **Primary** | PHP + MySQL + Simple MVC |
| **Alternatives** | Flask, Express + EJS, Laravel |

### Documentation Site

| Role | Option |
|------|--------|
| **Primary** | Wiki.js (Self-Hosted) |
| **Alternatives** | Next.js + MDX, Docusaurus, GitBook |

---

## Tech Stack Reference

### Frontend Options

| Option | Description | Best For |
|--------|-------------|----------|
| **Next.js** | Modern web apps, SEO, rich interactivity | Full-featured web apps |
| **Vue.js/Nuxt** | Progressive enhancement | Incremental adoption |
| **Plain PHP Templates** | Traditional, server-rendered | Simple sites |
| **Laravel Blade** | Full-stack PHP | PHP ecosystems |

### Backend Options

| Option | Description | Best For |
|--------|-------------|----------|
| **Next.js API Routes** | Integrated with frontend | Full-stack JS |
| **Node.js + Express** | RESTful APIs, real-time | APIs, WebSockets |
| **PHP (Plain or MVC)** | Traditional, shared hosting compatible | Simple backends |
| **Laravel** | Batteries-included, conventions | Rapid development |
| **FastAPI** | API-first, data-heavy, ML integration | Python ecosystems |
| **Django** | Full-featured, admin panels | Data-heavy apps |

### Database Options

| Option | Description | Notes |
|--------|-------------|-------|
| **Supabase (PostgreSQL + BaaS)** | Full stack, pgvector | PREFERRED DEFAULT, $0 marginal |
| **PocketBase (SQLite + BaaS)** | Lightweight alternative | Simple auth, prototypes |
| **PostgreSQL (Standalone)** | Custom backend needs | Full control |
| **MySQL** | Shared hosting compatibility | Traditional |

### Auth Options

| Option | Best For |
|--------|----------|
| **Supabase Auth** | Email/password, OAuth, magic links |
| **NextAuth.js** | Next.js projects, many OAuth providers |
| **JWT (Custom)** | API-first, full control |
| **Laravel Breeze/Jetstream** | Laravel projects |
| **Session-based** | Server-rendered apps, simple auth |

---

## Output Template

```markdown
## PRIMARY RECOMMENDATION: {Stack Name}

{2-3 sentence summary}

### Why This Fits Your Project

- {Reason 1 - capability fit}
- {Reason 2 - addresses key features}
- {Reason 3 - ecosystem/community strength}
- {Reason 4 - scalability path}

### Tech Stack Breakdown

| Layer | Technology |
|-------|------------|
| Frontend | {Framework/library + version} |
| Backend | {Framework/language + version} |
| Database | {Database system + version} |
| Auth | {Authentication approach} |
| Styling | {CSS approach} |
| File Storage | {Storage solution} |
| Testing | {Testing frameworks} |

**Stack Philosophy:** Enterprise / Balanced / Hacker (see below)

---

## ALTERNATIVE 1: {Stack Name}

{Brief description}

**Why Consider This:**
- {Advantage 1}
- {Advantage 2}

**Trade-offs vs Primary:**
- {Disadvantage 1}
- {Disadvantage 2}

**When to Choose This Instead:**
- {Condition 1}
- {Condition 2}

---

## NOT RECOMMENDED: {Stack Name}

**Why Ruled Out:**
- {Specific reason 1}
- {Specific reason 2}

---

## Enterprise vs Hacker Analysis

| Option | Position | Why |
|--------|----------|-----|
| Primary: {name} | {Enterprise / Balanced / Hacker} | {1-sentence rationale} |
| Alt 1: {name} | {Enterprise / Balanced / Hacker} | {1-sentence rationale} |
| Alt 2: {name} | {Enterprise / Balanced / Hacker} | {1-sentence rationale} |

**What This Means For You:**
- If you value {speed/iteration/simplicity}: Consider {Hacker option}
- If you value {maintainability/team-scaling/type-safety}: Consider {Enterprise option}
- The recommended {Primary} balances {specific trade-off}

---

## Decision Rationale

**Chosen:** {stack} because {key reasons}
**Alternatives considered:** {stack} - not selected because {why}
**Reversibility:** Easy / Moderate / Difficult to change

---

## User-Stated Constraints

{If user explicitly mentioned deployment preferences, infrastructure requirements, or hosting constraints, document them here. Otherwise, omit this section.}

---

## Next Steps

**Handoff document created:** .docs/tech-stack-decision.md

1. Review and ask questions about the tech stack recommendation
2. When satisfied -> Invoke **deployment-advisor** skill for hosting decisions
3. Deployment-advisor will recommend hosting strategy based on this tech stack
```

---

*Reference document for tech-stack-advisor skill*
*Created: 2025-12-09*
