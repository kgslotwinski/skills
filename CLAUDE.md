# Public Skills Repository

This repository contains public skills distributed via `npx skills`.

## Skill Maintenance Rules

### Version Management

**CRITICAL:** When modifying any skill's SKILL.md or reference files, you MUST update the frontmatter:

1. **Bump version** following semver:
   - **Major (X.0.0)**: Breaking changes to triggers, allowed-tools, or workflow
   - **Minor (1.X.0)**: New capabilities, significant additions, new reference files
   - **Patch (1.2.X)**: Bug fixes, corrections, typo fixes, non-breaking refactors

2. **Update date**: Set `updated: YYYY-MM-DD` to today's date

### Example Frontmatter Update

```yaml
---
name: skill-name
version: 1.2.1  # ← BUMP THIS
updated: 2026-03-07  # ← UPDATE THIS
---
```

### Workflow

Before committing skill changes:
1. Update `version` field in SKILL.md frontmatter
2. Update `updated` field to today's date
3. Commit with message format: `skill-name: v1.2.1 - brief description`

## Skill Structure

Each skill should follow this structure:

```
skills/
  skill-name/
    SKILL.md           # Main skill file (keep under 200 lines)
    references/        # Optional: detailed API documentation
      *.md
```

## Style Guidelines

- No emojis in content unless explicitly requested
- Keep SKILL.md concise and actionable
- Extract verbose API documentation to references/
- Use plain markdown, avoid excessive formatting
