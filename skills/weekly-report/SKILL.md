---
name: weekly-report
description: Generate an English weekly report from Git/SVN commit history. Use when the user asks to create a weekly report, summarize their weekly work, or review commits from this week.
---

# Weekly Report Generator

Generate a structured English weekly report based on the user's commits from this Monday to today.

## Step 0: Ask for Repositories

Ask the user which repositories to include in the report. Use `AskUserQuestion` to ask:

```
Which repositories should I include in the weekly report?
Please provide the paths (comma-separated or one per line).
```

- The user can enter absolute or relative paths
- Resolve relative paths to absolute before proceeding
- If a path doesn't exist or is not a valid repo, warn and skip

## Step 1: Determine the Date Range

Calculate the date range for this week:
- **Start**: This Monday at 00:00:00 (local time)
- **End**: Today

```bash
# Get this Monday
date -d "last monday" +%Y-%m-%d
# Get today
date +%Y-%m-%d
```

## Step 2: Process Each Repository

For each selected path, detect the repo type and collect commits.

### Detection Order

Check in this priority order — first match wins:
1. **Git**: `.git` directory or file exists → treat as Git repository
2. **SVN**: `.svn` directory exists → treat as SVN repository
3. Otherwise: skip with a warning

### Git Repositories

1. **Auto-detect author**:
   ```bash
   cd <repo_path>
   git config user.name
   git config user.email
   ```

2. **Collect commits**:
   ```bash
   git log \
     --author="<username>" \
     --since="<monday> 00:00:00" \
     --until="<saturday> 00:00:00" \
     --pretty=format:"%h|%s|%ai|%an" \
     --no-merges
   ```

   If empty, retry with `--author=<email>`.

3. **Get repo name**: `basename $(git rev-parse --show-toplevel)`

### SVN Repositories

1. **Auto-detect author**:
   ```bash
   cd <repo_path>
   svn info --show-item last-changed-author
   ```

2. **Collect commits**:
   ```bash
   svn log -r {<monday>}:{<friday>} . | grep -B1 -A1 "<username>"
   ```

3. **Get repo name**: `basename $(svn info --show-item repos-root-url)`

## Step 3: Categorize Commits

Classify each commit by keyword matching (case-insensitive) on the commit message:

| Category | Keywords |
|----------|----------|
| **New Features** | `feat`, `add`, `implement`, `introduce`, `new`, `feature`, `support` |
| **Bug Fixes** | `fix`, `bugfix`, `patch`, `resolve`, `correct`, `repair`, `hotfix` |
| **Refactoring** | `refactor`, `restructure`, `reorganize`, `clean`, `optimize`, `simplify`, `extract`, `rename`, `move` |
| **Documentation** | `doc`, `docs`, `readme`, `comment`, `changelog` |
| **Testing** | `test`, `spec`, `coverage`, `assert`, `mock` |
| **Infrastructure** | `ci`, `cd`, `build`, `deploy`, `docker`, `config`, `setup`, `chore`, `deps` |
| **Other** | Everything else |

First match wins.

## Step 4: Generate the Report

Produce a structured English weekly report. Rewrite vague commit messages to be clearer while preserving technical accuracy. Keep hashes for traceability.

```markdown
# Weekly Report — Week of YYYY-MM-DD

## Project: <repo-name>

### New Features
- [`<hash>`] <Clear commit message> (YYYY-MM-DD)

### Bug Fixes
- [`<hash>`] <Clear commit message> (YYYY-MM-DD)

### Refactoring
- [`<hash>`] <Clear commit message> (YYYY-MM-DD)

_(omit empty categories)_

---

## Project: <repo-name-2>
...

---

## Summary

### Project: <repo-name>
- N commits | Category1: X, Category2: Y
- One-sentence summary of the key work done in this repository this week

### Project: <repo-name-2>
- N commits | Category1: X, Category2: Y
- One-sentence summary of the key work done in this repository this week

---

**Total commits**: N across M repositories
```

## Step 5: Output

1. Save to `weekly-report-YYYY-MM-DD.md` in the current working directory
2. Display the full report in the terminal

## Notes

- Repos with no commits from the user: include with "No commits this week."
- Handle errors gracefully — report and continue
- SVN revisions display as `r<revision>`
- Don't invent details when rewriting messages
