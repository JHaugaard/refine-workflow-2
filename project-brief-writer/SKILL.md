---
name: project-brief-writer
description: "Transform rough project ideas into problem-focused briefs that preserve learning opportunities. Outputs structured .docs/brief.json for downstream skills."
version: 1.2.0
author: john
tags:
  - workflow
  - discovery
  - brief
  - advisory
---

# project-brief-writer

<purpose>
Transform rough project ideas into problem-focused briefs that preserve learning opportunities. Produces technology-neutral requirements that can be consumed by downstream advisory skills.
</purpose>

<output>
- .docs/PROJECT-MODE.md (workflow mode declaration)
- .docs/brief.json (structured handoff for downstream skills)

The JSON handoff captures problem, goals, requirements, and deployment intent WITHOUT technology stack, architecture, deployment platform specifics, or implementation details. This brief must remain NEUTRAL so downstream skills (tech-stack-advisor, deployment-advisor) can perform their advisory roles without bias.
</output>

---

<workflow>

<phase id="0" name="initialize">
<action>Load environment context and ensure .docs/ directory exists.</action>

<environment-loading>
1. Attempt to read ~/.claude/environment.json
2. If not found: Note and proceed (graceful degradation)
3. If found: Check skill_data_access["project-brief-writer"]
4. This skill has EMPTY access [] - no environment data needed
5. Proceed without loading environment context

Note: project-brief-writer intentionally has no environment access to remain neutral about infrastructure decisions.
</environment-loading>

<docs-directory>
1. Check if .docs/ directory exists in current working directory
2. If not, create it
3. Proceed to mode selection
</docs-directory>
</phase>

<phase id="1" name="create-project-mode">
<action>Create .docs/PROJECT-MODE.md declaring learning intent before collecting any project information.</action>

<prompt-to-user>
Before we start, I need to understand your learning intent for this project.

**Which mode best fits your project?**

**LEARNING Mode** (Recommended for skill development)
- Want to learn about technology choices and trade-offs
- Willing to explore options and understand alternatives
- Timeline flexible, learning is primary goal

**DELIVERY Mode** (For time-constrained projects)
- Need to ship quickly with minimal learning overhead
- Already know technology stack or constraints
- Timeline tight, speed is critical

**BALANCED Mode** (Flexible approach)
- Want both learning AND reasonable delivery speed
- Willing to explore but pragmatic about time

**Your choice:** [LEARNING / DELIVERY / BALANCED]
</prompt-to-user>

<file-template name=".docs/PROJECT-MODE.md">
# PROJECT-MODE.md
## Workflow Declaration

**Mode:** [USER_CHOICE]
**Decision Date:** [TODAY]

### What This Means

[If LEARNING:]
- Prioritizing understanding technology trade-offs
- Subsequent skills include detailed exploration phases
- Willing to spend time understanding alternatives

[If DELIVERY:]
- Prioritizing speed and efficiency
- Streamlined workflows with quick decisions
- Minimal checkpoints

[If BALANCED:]
- Want both learning and reasonable speed
- Flexible pathways with optional detailed phases

### Workflow Commitments

- Using PROJECT-MODE.md to inform all subsequent decisions
- Following appropriate checkpoint level for this mode
- Mode can be changed by updating this file

### Anti-Bypass Protections

Prevents "Over-Specification Problem" (detailed brief that bypasses learning).
- Each skill checks this file
- Checkpoint strictness based on mode
- Global skipping of ALL checkpoints not allowed in LEARNING/BALANCED modes
</file-template>

<confirmation>
Created .docs/PROJECT-MODE.md with MODE: [user's choice]

This file will guide the entire Skills workflow.
</confirmation>
</phase>

<phase id="2" name="present-template">
<action>Present template for user to fill in.</action>

<template name="project-brief-input">
# Project Brief Template

## 1. Problem Statement
[What problem are you trying to solve? Why does it need solving?]

## 2. Project Goals
[What do you want to achieve? What does success look like?]

## 3. High-Level Features
[What should the project do? List capabilities, not implementation details]
- Feature 1:
- Feature 2:
- Feature 3:

## 4. Constraints
[What limitations exist? Timeline, budget, infrastructure, etc.]

## 5. Success Criteria
[How will you know the project succeeded?]

## 6. Use Cases
[Describe 2-3 scenarios of how this will be used]

## 7. Known Edge Cases
[What unusual situations should be handled?]

## 8. Explicitly Out of Scope
[What will NOT be included in initial version?]

## 9. Deployment Intent

[Where do you expect this to run? Choose one - DO NOT specify platforms or hosting providers]

- [ ] Localhost only (runs on my machine)
- [ ] Public (accessible to others online)
- [ ] TBD (undecided)

## 10. Learning Goals (Optional)

[What do you want to learn from this project?]

## 11. User Stated Preferences (Optional - Reference Only)

[If you have specific technology or platform preferences, note them here. These will be recorded as stated preferences for downstream skills to consider, but will NOT constrain their recommendations.]
</template>

<instruction-to-user>
Please fill in this template with your project idea. Focus on WHAT you want to build and WHY, not HOW to build it. Leave sections blank if you haven't thought about them yet - I'll ask follow-up questions.

**Important:** Keep your responses technology-neutral. If you have preferences for specific technologies or platforms, note them in the "User Stated Preferences" section - they'll be forwarded to downstream skills for consideration but won't lock in decisions.
</instruction-to-user>
</phase>

<phase id="3" name="analyze-input">
<action>Analyze user responses for completeness, over-specification, and appropriate detail level.</action>

<checks>
- Completeness: Which sections are empty or vague?
- Over-specification: Are they describing implementation instead of requirements?
- Tech mentions: Did they mention specific technologies in requirements sections?
- Appropriate detail: Too vague (needs expansion) or too detailed (bypasses learning)?
- Deployment intent: Is it clear where this will run?
</checks>

<detection-rules name="over-specification">
<indicators>
- Specific programming languages, frameworks, or libraries in requirements
- Architecture descriptions (microservices, MVC, etc.)
- Implementation patterns or design patterns
- Specific APIs or third-party services (unless core to problem)
- Database schemas or data models
- Deployment platforms or hosting providers (AWS, Vercel, fly.io, Cloudflare, etc.)
- Infrastructure specifics (Docker, Kubernetes, serverless, etc.)
- CI/CD tools or automation platforms
</indicators>

<examples>
<bad>Build a REST API using Express.js with JWT authentication</bad>
<good>Build an API that allows authenticated users to access their data</good>

<bad>Use React with Redux for state management and Material-UI components</bad>
<good>Build a user interface with good visual design and responsive layout</good>

<bad>Deploy to Vercel with a PostgreSQL database on Supabase</bad>
<good>Needs to be publicly accessible with persistent data storage</good>

<bad>Host on my VPS with Docker and Caddy</bad>
<good>Will run on my own server (public deployment)</good>
</examples>
</detection-rules>

<response-if-overspecified>
I notice you're describing HOW to implement (e.g., 'using Express.js'). For this brief, let's focus on WHAT you need (e.g., 'an API that handles user requests'). The tech-stack-advisor skill will help you explore implementation options later. Would you like to rephrase this as a requirement without specifying the technology?
</response-if-overspecified>

<response-if-tech-in-requirements>
I see you've mentioned [specific technology]. I'll record that as a stated preference in the handoff document. The requirements will stay technology-neutral so downstream skills can provide unbiased recommendations - but your preference will be visible for them to consider. Does that work?
</response-if-tech-in-requirements>

<response-if-deployment-specified>
I see you've mentioned a specific deployment platform/infrastructure [e.g., 'Vercel', 'Docker', 'AWS']. For this brief, I'll capture your deployment intent (localhost vs. public) and record your platform preference separately. The deployment-advisor skill will help evaluate whether that's the best fit for your needs. Does that work?
</response-if-deployment-specified>
</phase>

<phase id="4" name="clarifying-questions">
<action>Ask 2-3 batches of questions based on gaps. Don't ask everything at once.</action>

<batch id="1" name="problem-clarity" trigger="Problem Statement is vague">
- Who experiences this problem?
- What's the current workaround or manual process?
- What's the impact of NOT solving this problem?
- Is this a new problem or improving an existing solution?
</batch>

<batch id="2" name="scope-features" trigger="Features are unclear">
- What's the minimum set of features for this to be useful?
- Are there features that are "nice to have" vs "must have"?
- What should users be able to do that they can't do now?
- Are there features you've explicitly decided NOT to include?
</batch>

<batch id="3" name="users-stakeholders" trigger="Use Cases are weak">
- Who is the primary user?
- Are there different types of users with different needs?
- What are the most common scenarios where this will be used?
- What are the edge cases or unusual situations to consider?
</batch>

<batch id="4" name="success-constraints" trigger="Success Criteria missing">
- How will you measure success?
- What timeline are you working with?
- Are there budget constraints?
- What existing infrastructure or tools must this work with?
</batch>

<batch id="5" name="deployment-intent" trigger="Deployment Intent unclear">
- Will this run only on your computer (localhost)?
- Does this need to be accessible to others online?
- Is this primarily for learning, or do you need it in production?
</batch>

<batch id="6" name="learning-goals" trigger="Learning mode selected or learning goals unclear">
- Is this primarily a learning project or a delivery project?
- What specific technologies or concepts do you want to learn?
- How much time can you dedicate to learning vs building?
</batch>

<strategy>
- Ask one batch at a time
- Wait for responses before next batch
- Skip batches if already answered
- Use conversational tone
</strategy>
</phase>

<phase id="5" name="generate-brief">
<action>Generate structured JSON handoff after gathering sufficient information.</action>

<json-schema>
{
  "document_type": "brief",
  "version": "1.0",
  "created": "[YYYY-MM-DD]",
  "project": "[project-name]",
  "mode": "[LEARNING/DELIVERY/BALANCED]",

  "rationale": "[1-3 sentence human-readable summary of the project and its goals]",

  "problem": {
    "statement": "[Clear description of problem being solved]",
    "impact": ["[Impact point 1]", "[Impact point 2]", "[Impact point 3]"]
  },

  "goals": {
    "primary": "[Main objective in one sentence]",
    "secondary": ["[Goal 1]", "[Goal 2]", "[Goal 3]"]
  },

  "features": [
    {
      "name": "[Feature name]",
      "description": "[What it does and why]",
      "priority": "must-have | nice-to-have"
    }
  ],

  "success_criteria": [
    {
      "metric": "[Metric name]",
      "measurement": "[How to measure]"
    }
  ],

  "use_cases": [
    {
      "name": "[Use case name]",
      "scenario": "[Describe situation]",
      "expected_behavior": ["[Step 1]", "[Step 2]"]
    }
  ],

  "edge_cases": [
    {
      "category": "[Category name]",
      "problem": "[What unusual situation might occur]",
      "solution": "[How system should handle it]"
    }
  ],

  "constraints": {
    "timeline": "[If specified, otherwise null]",
    "budget": "[If specified, otherwise null]",
    "infrastructure": "[Existing systems to work with, otherwise null]"
  },

  "assumptions": ["[Assumption 1]", "[Assumption 2]"],

  "out_of_scope": ["[Feature not included]", "[Feature not included]"],

  "deployment_intent": "localhost | public | tbd",

  "learning_goals": {
    "topics": ["[Learning goal 1]", "[Learning goal 2]"],
    "skills": ["[Skill 1]", "[Skill 2]"]
  },

  "user_stated_preferences": {
    "disclaimer": "These are user-stated starting points, NOT requirements. Downstream skills should provide unbiased recommendations.",
    "technology": "[User's tech preferences if stated, otherwise null]",
    "platform": "[User's platform preferences if stated, otherwise null]"
  }
}
</json-schema>

<generation-rules>
- Populate all fields based on gathered information
- Use null for optional fields user didn't specify
- Keep rationale concise (1-3 sentences)
- Ensure deployment_intent is one of: localhost, public, tbd
- Mode must match PROJECT-MODE.md
- NO technology specifications in any field except user_stated_preferences
- NO platform specifics except in user_stated_preferences
</generation-rules>
</phase>

<phase id="6" name="save-brief">
<action>Save the JSON handoff to .docs/ subdirectory.</action>

<save-location>.docs/brief.json</save-location>

<prompt-to-user>
I'll save this brief to `.docs/brief.json`.

[Show formatted JSON preview]

Does this capture your project accurately?
</prompt-to-user>
</phase>

<phase id="7" name="confirm-completion">
<action>Confirm save and summarize outputs.</action>

<confirmation-template>
**Project Brief Complete**

Saved:
- `.docs/brief.json` — structured project requirements
- `.docs/PROJECT-MODE.md` — workflow mode declaration

Summary:
- Mode: [LEARNING/DELIVERY/BALANCED]
- Deployment intent: [localhost/public/tbd]
- User preferences: [captured/none stated]

The brief is ready for downstream skills to consume.
</confirmation-template>
</phase>

</workflow>

---

<guardrails>

<primary-directive>
This skill produces a NEUTRAL handoff document. The brief must NOT influence or constrain decisions made by downstream skills (tech-stack-advisor, deployment-advisor). Any user preferences are recorded separately and clearly labeled as non-binding.
</primary-directive>

<must-do>
- Create .docs/ directory if it doesn't exist
- Keep brief focused on problem space (WHAT, not HOW)
- Redirect over-specification attempts
- Record user preferences in user_stated_preferences object ONLY
- Include disclaimer that preferences are non-binding
- Preserve learning opportunities (appropriate detail level)
- Generate valid JSON output matching the schema
- Include deployment_intent as: localhost, public, or tbd
- Save to .docs/brief.json
- Guide user to next skill in workflow
</must-do>

<must-not-do>
- Specify technologies, frameworks, or libraries in any JSON field except user_stated_preferences
- Include architecture or design patterns in requirements
- Name deployment platforms or hosting providers except in user_stated_preferences
- Include infrastructure specifics (Docker, serverless, etc.) in requirements
- Make implementation or deployment decisions
- Endorse or validate user's tech/platform preferences (remain neutral)
- Skip clarifying questions when gaps exist
- Allow hyper-detailed specifications that bypass learning
- Save files outside .docs/ subdirectory
- Generate invalid JSON
</must-not-do>

<neutrality-rules>
When users state technology or platform preferences:
1. Acknowledge the preference without endorsing it
2. Record it verbatim in "User Stated Preferences" section
3. Include the explicit disclaimer that downstream skills will provide unbiased recommendations
4. Do NOT rephrase preferences in a way that makes them sound like requirements
5. Do NOT elaborate on why their preference might be good/bad
</neutrality-rules>

<quality-checks>
Before generating brief, verify:
1. Problem clarity: Clear what problem is being solved and why?
2. Feature completeness: Core features described at appropriate level?
3. Success criteria: Can success be measured?
4. Use cases: Realistic scenarios described?
5. Boundaries: Scope clear (in-scope and out-of-scope)?
6. Tech neutrality: Requirements sections free of ALL implementation details?
7. Deployment neutrality: Deployment intent is category-only (no platform names)?
8. Preference isolation: User preferences appear ONLY in dedicated section with disclaimer?

If any check fails, ask more questions or revise before generating.
</quality-checks>

</guardrails>

---

<examples>

<example name="user-over-specifies">
<user-input>Build a REST API using Node.js and Express with MongoDB database</user-input>
<response>
I notice you're specifying technologies (Node.js, Express, MongoDB). For this brief, let's describe what the API needs to do without specifying the technology.

Try rephrasing as: 'Build an API that stores and retrieves data'

I'll record 'Node.js, Express, MongoDB' in the User Stated Preferences section. The tech-stack-advisor skill will consider these alongside other options for your specific needs. Does that work?
</response>
</example>

<example name="user-too-vague">
<user-input>Build a tool to organize photos</user-input>
<response>
Great starting point! Let me ask some clarifying questions:

**Problem Clarity**
- What specific problem with photo organization are you trying to solve?
- Who will use this tool?
- What's your current workflow, and what's frustrating about it?
- What would 'organized' mean for your photos?
</response>
</example>

<example name="localhost-project">
<user-input>Build a personal task manager that runs on my Mac</user-input>
<response>
I see this is a localhost-only project for personal use. I'll mark the deployment intent as "Localhost."

Let me gather a few more details about your task manager requirements...
</response>
</example>

<example name="user-specifies-platform">
<user-input>I want to build a blog platform and deploy it on Vercel with a Supabase database</user-input>
<response>
I'll capture "Vercel" and "Supabase" as your stated preferences - they'll be recorded in the handoff document for downstream skills to consider.

For the brief itself, I'll describe what the blog platform needs to do (features, requirements) and mark deployment intent as "Public." The deployment-advisor skill will evaluate your Vercel/Supabase preferences alongside other options.

Let me ask some clarifying questions about what the blog should do...
</response>
</example>

<example name="good-user-input">
<user-input>Build a migration tool that transfers images from SmugMug to BackBlaze B2 storage while preserving metadata like dates, titles, and albums.</user-input>
<response>
Excellent! This is a well-scoped problem. Let me ask a few questions:

**Scope & Success**
- Approximately how many images will you be migrating?
- Do you need this to run once, or repeatedly for ongoing sync?
- How will you know the migration was successful?
- What should happen if the migration is interrupted?

**Deployment Intent**
- Will this run only on your computer, or does it need to be accessible remotely?
</response>
</example>

</examples>

---

<integration-notes>

<outputs>
Produces:
- `.docs/PROJECT-MODE.md` — workflow mode declaration (LEARNING/DELIVERY/BALANCED)
- `.docs/brief.json` — structured handoff for downstream skills

These files are designed to be consumed by advisory skills (tech-stack-advisor, deployment-advisor, etc.) without requiring this skill to specify which comes next.
</outputs>

<deployment-intent>
Deployment intent is captured as a simple category only:
- localhost: Project runs locally
- public: Project needs online access
- tbd: Decision deferred

This enables downstream routing decisions without this skill prescribing them.
</deployment-intent>

</integration-notes>
