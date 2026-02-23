# Skills Repository

A collection of AI agent skills for development automation, compatible with Claude Code and other AI coding assistants.

## Structure

```
skills/
├── config.json         # Global configuration
└── skills/            # Individual skills
    └── easy-admin-bundle/
        └── SKILL.md    # Skill definition
```

## Available Skills

- **easy-admin-bundle** - Symfony EasyAdminBundle automation for rapid admin panel development

## Installation

### Using NPX (recommended)
```bash
# Install all skills
npx skills add kgslotwinski/skills

# Install specific skill
npx skills add kgslotwinski/skills --skill easy-admin-bundle
```

### Manual Installation

Copy skills to your project or global directory:

```bash
# Project-specific (recommended)
cp -r skills/easy-admin-bundle .claude/skills/

# Global
cp -r skills/easy-admin-bundle ~/.claude/skills/
```

## Adding New Skills

1. Create a new directory under `skills/`
2. Add `SKILL.md` with frontmatter and documentation
3. Skills are automatically discovered

### SKILL.md Format

```markdown
---
name: skill-name
description: "Detailed description with capabilities, use cases, and triggers"
allowed-tools: Bash(command *), Read, Write, Edit, Glob, Grep
---

# Skill Title

Complete documentation with examples, code snippets, and workflows.
```

## SKILL.md Components

- **name**: Unique skill identifier (kebab-case)
- **description**: Rich description including:
  - Core capabilities
  - Use cases
  - Trigger keywords (for AI to recognize when to use the skill)
- **allowed-tools**: Tools the skill can use (Bash commands, Read, Write, etc.)
- **Content**: Complete markdown documentation with:
  - Quick start guide
  - Full code examples
  - Common patterns
  - Troubleshooting
  - Configuration examples

## Requirements

- Claude Code or compatible AI coding assistant
