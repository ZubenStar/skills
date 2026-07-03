---
name: weekly-report
description: Generate an English weekly report from Git/SVN commit history. Use when the user asks to create a weekly report, summarize their weekly work, or review commits from this week.
---

# Weekly Report Generator

Generate a structured English weekly report based on the user's commits from this Monday to today.

## Arguments

The user provides one or more repository paths as arguments. If no arguments are provided, use the current working directory.

Usage examples:
- `/weekly-report /path/to/repo1 /path/to/repo2`
- `/weekly-report` (use CWD)

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

For each provided path, detect the repo type and collect commits.

### Detection
- **Git**: `.git` directory or file exists
- **SVN**: `.svn` directory exists
- Otherwise, skip with a warning

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

- **Total commits**: N across M repositories
- **Key highlights**:
  - 1-2 sentence summary of the most important work this week
```

## Step 5: Output

1. Save to `weekly-report-YYYY-MM-DD.md` in the current working directory
2. Display the full report in the terminal

## Notes

- Repos with no commits from the user: include with "No commits this week."
- Handle errors gracefully — report and continue
- SVN revisions display as `r<revision>`
- Don't invent details when rewriting messages
