# Skills

Custom Claude Code skills collection.

## Skills

| Skill | Description |
|-------|-------------|
| [weekly-report](./skills/weekly-report) | Generate English weekly reports from Git/SVN commit history |
| [session-memory](./skills/session-memory) | Extract safe, reusable memories from the current conversation |

## Installation

### From GitHub

```
/plugin marketplace add <your-github-user>/skills
```

Then install:

```
/plugin install weekly-report@tufzh-skills
/plugin install session-memory@tufzh-skills
```

### From Local

```
/plugin marketplace add /path/to/skills
```

## Usage

```
/weekly-report /path/to/repo1 /path/to/repo2
/session-memory Extract reusable memories from this conversation
```

## Creating a Skill

Each skill is a folder under `skills/` containing a `SKILL.md` file with YAML frontmatter:

```markdown
---
name: my-skill
description: What this skill does and when to use it
---

# My Skill

Instructions here...
```

## License

MIT
