## 9. Getting Started

The previous sections described the workflows and structure I use. This section shows how to build toward that setup — whether you're working solo or introducing it to a team.

### 9.1 For Individual Developers

#### Take time to build intuition

Don't expect immediate productivity gains. The first weeks are an investment in learning how Claude Code behaves with your specific codebase, understanding its strengths and limitations, and developing a sense for which tasks it handles well autonomously versus which need close supervision. 

At the same time, I believe it can be introduced safely without decreasing the current performance just from the first days.

#### Start with the foundation

Before diving into complex workflows, set up the basic structure described in Section 8:

- Create your CLAUDE.md files with project-specific context
- Configure basic code review subagents

#### Identify patterns and create skills upfront

Before starting real tasks, analyze your codebase for recurring patterns. It's more effective to create skills for your most common patterns upfront:

- Review your typical feature implementation: what components do you usually create?
- Identify the common operations: adding entities, creating API endpoints, implementing forms, etc.
- Create a skill for each pattern with examples from your existing code

#### Begin with simple tasks

Once your skills are in place, start with straightforward, well-defined tasks:

- Simple bug fixes with clear reproduction steps
- Adding a new field to an existing entity
- Implementing a feature similar to existing ones

Run one task at a time. Watch how Claude Code approaches the problem, what context it gathers, where it succeeds, and where it struggles. In case of failure, try to understand why it happened, tune Claude Code configuration (create a doc, update CLAUDE.md, etc), and start the process from the beginning.

#### Run tasks in parallel only when you're confident in your current workflows

Start tasks in parallel only if you are confident enough that your workflows are stable. 

#### Iterate continuously

Your setup will never be "done." As you work:

- Add skills when you notice repeated patterns
- Update documentation when Claude Code makes incorrect assumptions
- Refine workflows based on what actually works for your project

The goal is continuous improvement, not perfection from day one.

### 9.2 For Teams

#### Choose a Champion

Designate one person to lead the introduction. This champion should:

- Have time to experiment and iterate on the setup
- Be able to document patterns and create initial skills

The champion builds the foundation that the rest of the team will use.

#### Centralize the configuration

Teams benefit significantly from a centralized approach:

- Shared CLAUDE.md files ensure consistent context across all team members
- Common skills library prevents duplicate effort
- Standardized workflows make code reviews predictable
- Unified code review subagents enforce team standards

#### Spread across team members

Once the foundation is solid:

1. Start with developers who are enthusiastic about the tooling
2. Pair them with the champion for initial onboarding
3. Gather feedback and refine the setup
4. Gradually expand to the rest of the team

Don't force adoption.

#### Spread across the organization

For multi-team organizations:

- Share successful patterns between teams via plugins
- Create organization-wide skills for common infrastructure and patterns (auth, logging, etc.)
- Establish cross-team standards for Claude Code configuration

### 9.3 Bonus: A Skill for Creating Skills

When building your initial skills library, having a meta-skill that creates other skills can speed up the process significantly. Here's the actual skill I use:

````markdown
---
name: implementing-skill
description: Creates concise Claude Code skills from source documents, existing source code, code examples, or prompts. Extracts only project-specific patterns. Target 50-100 lines.
---

# Implementing Concise Claude Code Skills

Creates focused skills containing only project-specific patterns, conventions, and gotchas.

## When to Use

- User wants to create skill from documentation, code examples, source code or description
- Codifying project-specific patterns into reusable skill

## Core Philosophy

**Assume Claude is an expert programmer.** Only include what's unique to this project:

✅ **Include:**
- Project-specific naming/structure conventions
- Unique file locations and organization
- Critical gotchas (e.g., `@psalm-suppress TooManyArguments`)
- Project-specific validation commands
- Non-obvious patterns from codebase

❌ **Exclude:**
- Standard language/framework knowledge Claude already knows
- Generic programming patterns
- Verbose explanations of common concepts
- Information fully covered in referenced example files

**Target: 50-100 lines** (150 max)

## Instructions

### 1. Analyze All Available Context

Extract patterns from whatever user provides:

**Source types:**
- Documentation files (e.g., `backend/doc/repository-creation.md`)
- Code examples (e.g., "based on TeamDomainRepository.php")
- User prompt describing workflow
- UML diagrams or technical specs
- Mix of above

**Read and identify:**
- What's project-specific vs generic?
- What patterns repeat across examples?
- What gotchas or special annotations appear?
- What validation/workflow steps are unique?

### 2. Check for Conflicts & Better Solutions

Before creating skill, verify:

**Check existing skills:**
- Glob `.claude/skills/*/SKILL.md` and `*/config.json`
- Name/description overlaps? → Propose extending/merging
- Keyword conflicts? → Suggest different keywords
- Similar trigger scenarios? → Consolidate or differentiate

**Is skill the right solution?**
- Simple one-time pattern? → Add to `CLAUDE.md` instead
- Reactive automation? → Use hook (`.claude/hooks/`)
- User-invoked workflow? → Create slash command (`.claude/commands/`)
- Complex repetitive pattern? → Skill is appropriate

**If conflicts/alternatives found:** Present options to user, get approval before proceeding.

### 3. Extract Project-Specific Patterns Only

From all sources, extract ONLY:
- Unique naming conventions (e.g., `{EntityName}Repository`, `{EntityName}RepositoryImpl`)
- Specific file paths/structure (e.g., `{Module}/Domain/`, `{Module}/Infrastructure/`)
- Critical annotations (e.g., `@psalm-suppress`, `@extends`, specific doc comments)
- Project validation commands (e.g., `docker compose exec php composer check`)
- When to trigger (keywords, scenarios)

### 4. Generate Skill Name & Ask Minimal Questions

**Generate name automatically:**
- Component-specific: `{component}-{action}-{object}`
  - Examples: `backend-implementing-repository`, `frontend-creating-component`
- General: `{action}-{object}`
  - Examples: `creating-feature-directory`, `reviewing-skill`
- Format: lowercase-with-hyphens, gerund form (-ing), max 64 chars

**Ask 1-2 questions max, only if:**
- Skill name is ambiguous (confirm generated name)
- Critical pattern unclear from all sources (rarely needed)

Extract everything else from context automatically.

### 5. Create Concise SKILL.md (50-100 lines)

**Structure:**
```markdown
---
name: {skill-name}
description: {What + when, max 164 chars}
---

# {Title}

{One sentence: what it creates/does}

## When to Use

- {Trigger keyword scenario 1}
- {Trigger keyword scenario 2}

## Instructions

### 1. {First Step - usually context gathering}

Extract from context (docs/UML/prompt/code):
- {What to extract}
- Only ask if critical info missing

### 2. {Main Action Step}

**{File/Component Type 1}:** `{project/specific/path}`
- {Project-specific pattern}
- {Critical gotcha if any}

**{File/Component Type 2}:** `{project/specific/path}`
- {Pattern unique to this project}

**Reference:** `{path/to/example/file}`

### 3. {Project-Specific Patterns Section}

**{Pattern Name}** (only if non-obvious):
\```{language}
{Minimal code showing unique project pattern}
\```

### 4. Validate

\```bash
{project-specific validation command}
\```

### 5. {Optional: Updating Existing}

{Brief: Read → Update → Validate}
```

**Apply ruthless filtering:**
- Each line: "Is this project-specific?" → No? Delete.
- Each example: "Would Claude know this?" → Yes? Just reference file instead.
- Keep critical patterns, skip explanations.

### 6. Create config.json

**File:** `{skill-name}/config.json`
```json
{
  "keywords": ["keyword1", "keyword2"],
  "hint": "☝️ {Brief hint when to use this skill}"
}
```

**Keywords:** Words that trigger hint in user prompts (case-insensitive, whole word match)
**Hint:** Short message suggesting skill usage (shown via hook when keywords match)

### 7. Create Templates (Optional)

Only if template:
- Contains non-obvious project structure
- Saves significant time vs writing from scratch

Otherwise: reference existing example files.

### 8. Validate & Report

1. Check line count: 50-100 ideal, 150 absolute max
2. Remove any generic content that snuck in
3. Invoke `reviewing-skill`
4. Report created files and next steps

## Naming Conventions

- `backend-{action}-{object}` (e.g., `backend-implementing-repository`)
- `frontend-{action}-{object}` (e.g., `frontend-creating-hook`)
- `infrastructure-{action}-{object}` (e.g., `infrastructure-deploying-lambda`)
- General: `{action}-{object}` (e.g., `creating-feature`, `reviewing-skill`)

Always gerund form (-ing) for action.
````

This skill demonstrates several patterns worth noting:

- **Core philosophy section** — sets expectations for what should and shouldn't be included
- **Conflict checking** — verifies the skill doesn't duplicate existing functionality
- **Template structure** — provides a consistent format for all skills
- **Naming conventions** — ensures skills are easy to find and understand

Ironically, this skill doesn't follow its own "50-150 lines" recommendation. This is not a hard rule, but rather a constraint for Claude Code while it implements the skill - it's forced to keep the information short and provide only necessary details.s
