# Boost Skills

Custom [Laravel Boost](https://laravel.com/docs/boost) skills for team development standards and conventions.

## Installation

```bash
php artisan boost:add-skill corgab/boost-skills
```

### Options

```bash
# List available skills
php artisan boost:add-skill corgab/boost-skills --list

# Install all skills
php artisan boost:add-skill corgab/boost-skills --all

# Install specific skill(s)
php artisan boost:add-skill corgab/boost-skills --skill=skill-name

# Force update existing skills
php artisan boost:add-skill corgab/boost-skills --force
```

## Creating a new skill

1. Copy the `_template` folder and rename it:

```bash
cp -r _template my-new-skill
mv my-new-skill/SKILL.md.example my-new-skill/SKILL.md
```

2. Edit `my-new-skill/SKILL.md`:
   - Update the frontmatter (`name`, `description`)
   - Write the skill content

3. Commit and push.

## Skill structure

```
my-skill/
└── SKILL.md          # Required - skill definition with YAML frontmatter
```

Each skill folder must contain a `SKILL.md` (or `SKILL.blade.php` for dynamic content) with YAML frontmatter:

```yaml
---
name: my-skill
description: "When to use this skill"
metadata:
  author: corgab
---
```
