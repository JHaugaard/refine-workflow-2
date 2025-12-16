# project-brief-writer Skill

## Overview

Transform rough project ideas into problem-focused, learning-appropriate project briefs that preserve learning opportunities and feed cleanly into the Skills workflow.

**Use when:** You have a new project idea and want to create a non-technical brief that focuses on WHAT to build (not HOW), preventing the PRD Quality Paradox where over-detailed specifications bypass learning opportunities.

**Output:** A polished project brief in narrative, bullet-point structured format that describes the problem, goals, and requirements without specifying technology stack, architecture, or implementation details.

---

## How It Works

When invoked, this skill will:

1. **Create PROJECT-MODE.md** - Declare your learning intent (LEARNING/DELIVERY/BALANCED)
2. **Present a template** - Simple form for your project idea
3. **Analyze responses** - Check for completeness and appropriate detail
4. **Ask clarifying questions** - Fill gaps in batches
5. **Detect over-specification** - Redirect if you specify HOW instead of WHAT
6. **Quarantine tech thoughts** - Preserve technology ideas in separate section
7. **Generate polished brief** - Professional format ready for next skill
8. **Save and confirm** - Store file and show workflow status

---

## Skills Workflow Integration

This skill is the **first step** in the learning-focused project workflow:

```
Rough Idea
    ↓
project-brief-writer ← YOU ARE HERE
    ↓
problem-focused brief.md
    ↓
tech-stack-advisor (explore tech options)
    ↓
deployment-advisor (plan deployment)
    ↓
project-spinup (create foundation)
```

**Key principle:** This brief preserves "gaps" that subsequent skills help explore. Those gaps are where learning happens.

---

## Learning Mode Options

### LEARNING Mode
- Prioritize understanding technology trade-offs
- Detailed exploration phases and checkpoints
- Timeline flexible, learning is primary goal

### DELIVERY Mode
- Ship quickly with minimal learning overhead
- Streamlined workflows with quick decisions
- Timeline tight, speed is critical

### BALANCED Mode
- Both learning AND reasonable delivery speed
- Flexible pathways with optional detailed phases
- Best of both worlds with acknowledged trade-offs

---

## The Over-Specification Problem

This skill prevents the "PRD Quality Paradox" where detailed specifications bypass learning:

**Over-specified (bad):**
- "Build a REST API using Express.js with JWT authentication"
- "Use React with Redux for state management"

**Appropriate (good):**
- "Build an API that allows authenticated users to access their data"
- "Build a user interface with good visual design"

Technology choices belong in tech-stack-advisor, not the brief.

---

## Example Interactions

### User Over-Specifies

**Input:** "Build a REST API using Node.js and Express with MongoDB database"

**Response:** The skill redirects to focus on requirements, captures tech ideas separately, and explains that tech-stack-advisor will help explore whether those are the best options.

### User Too Vague

**Input:** "Build a tool to organize photos"

**Response:** The skill asks clarifying questions about the specific problem, target users, current workflow frustrations, and what "organized" means.

### Good Input

**Input:** "Build a migration tool that transfers images from SmugMug to BackBlaze B2 storage while preserving metadata"

**Response:** The skill acknowledges the well-scoped problem and asks targeted questions about scale, sync frequency, success criteria, and error handling.

---

## Related Skills

- **tech-stack-advisor** - Uses PROJECT-MODE.md to determine checkpoint strictness (next in workflow)
- **deployment-advisor** - Continues MODE-aware workflow guidance
- **project-spinup** - Final skill completing the 3-phase workflow

---

## Background Documentation

- **after-action-report.md** - Over-Specification Problem and SmugMug project lessons
- **lovable-vs-claude-code.md** - Strategic vs tactical learning, Phase 0 meta-skills philosophy
- **done-vs-next.md** - Phase 0 meta-skills positioning and workflow focus

---

## Future Enhancements

After using this skill 2-3 times, consider adding:
- Support for different project types (hardware, content, research)
- Customizable question batches based on project category
- Brief versioning and evolution support
- Brief comparison and diff tools

---

## Version History

### v1.1 (2025-01-11)
**Skills Workflow Refinement - Phase 2**

- Added PROJECT-MODE.md auto-creation with mode selection
- Implemented workflow state visibility
- Added MODE-aware guidance for subsequent skills
- Documented anti-bypass protections

### v1.0 (2025-01-08)
**Initial Release**

- Problem-focused brief template
- Over-specification detection
- Tech thought quarantine
- Clarifying question batches
- Polished brief generation

---

**Version:** 1.1
**Last Updated:** 2025-01-11
