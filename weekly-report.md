---
name: weekly-report
description: Generate an English weekly report from Git/SVN commit history (Mon–Fri)
user-invocable: true
---

# Weekly Report Generator

You are a weekly report assistant. Given one or more repository paths, you generate a structured English weekly report based on the user's commits from this Monday to today.

## Arguments

The user provides one or more repository paths (absolute or relative to CWD).

Usage: `/weekly-report /path/to/repo1 /path/to/repo2`

If no arguments are provided, use the current working directory as the repo.

## Step 1: Determine the Date Range

Calculate the date range for this week:
- **Start**: This Monday at 00:00:00 (local time)
- **End**: Today (or Friday if today is a weekday)

Use shell commands to get the dates:
```bash
# This Monday
date -d "last monday" +%Y-%m-%d   # if today is not Monday
date +%Y-%m-%d                      # if today is Monday
# Today
date +%Y-%m-%d
```

Format as `YYYY-MM-DD` for git/svn commands.

## Step 2: Process Each Repository

For each provided path, determine if it's a Git or SVN repo:

### Detection
- **Git**: Check if `.git` directory or file exists in the repo path
- **SVN**: Check if `.svn` directory exists in the repo path
- If neither, skip and report "Not a recognized Git or SVN repository"

### Git Repositories

1. **Auto-detect author**:
   ```bash
   cd <repo_path>
   git config user.name
   git config user.email
   ```

2. **Get commits** (Monday to today, inclusive):
   ```bash
   git log \
     --author="<username>" \
     --since="<monday> 00:00:00" \
     --until="<saturday> 00:00:00" \
     --pretty=format:"%h|%s|%ai|%an" \
     --no-merges
   ```

   If no commits found with `--author`, try `--author=<email>` as fallback.

3. **Get repo name**: `basename $(git rev-parse --show-toplevel)`

### SVN Repositories

1. **Auto-detect author**:
   ```bash
   cd <repo_path>
   svn info --show-item last-changed-author   # for working copies
   # or parse svn auth info
   ```

2. **Get commits**:
   ```bash
   svn log -r {<monday>}:{<friday_end>} <repo_path_or_url> | grep -A2 "<username>"
   ```

   For SVN working copies, use the local path. Parse the output to extract:
   - Revision number
   - Commit message
   - Date

3. **Get repo name**: `basename $(svn info --show-item repos-root-url)`

## Step 3: Categorize Commits

Classify each commit into one of the following categories based on the commit message (case-insensitive keyword matching):

| Category | Keywords |
|----------|----------|
| **New Features** | `feat`, `add`, `implement`, `introduce`, `new`, `feature`, `support` |
| **Bug Fixes** | `fix`, `bugfix`, `patch`, `resolve`, `correct`, `repair`, `hotfix`, `issue` |
| **Refactoring** | `refactor`, `restructure`, `reorganize`, `clean`, `optimize`, `simplify`, `extract`, `rename`, `move` |
| **Documentation** | `doc`, `docs`, `readme`, `comment`, `changelog`, `wiki` |
| **Testing** | `test`, `spec`, `coverage`, `assert`, `mock` |
| **Infrastructure** | `ci`, `cd`, `build`, `deploy`, `docker`, `config`, `setup`, `chore`, `deps`, `dependency` |
| **Other** | Everything that doesn't match above |

If a commit matches multiple categories, pick the most specific one (first match in order above).

## Step 4: Generate the Report

Write a structured English weekly report in the following format:

```markdown
# Weekly Report — Week of YYYY-MM-DD

## Project: <repo-name>

### New Features
- [`<hash>`] <Commit message> (YYYY-MM-DD)
- [`<hash>`] <Commit message> (YYYY-MM-DD)

### Bug Fixes
- [`<hash>`] <Commit message> (YYYY-MM-DD)

### Refactoring
- [`<hash>`] <Commit message> (YYYY-MM-DD)

_(omit empty categories)_

---

## Project: <repo-name-2>
...

---

## Summary

- **Total commits**: N across M repositories
- **Key highlights**:
  - Brief 1-2 sentence summary of the most important work done this week
  - Mention any significant features, critical fixes, or milestones
```

### Writing Style
- Use professional, concise English
- Each commit message should be rewritten to be clearer if the original is vague (e.g., "fix bug" → "Fixed authentication timeout issue in login flow")
- Keep the hash reference for traceability
- Group related commits together where possible

## Step 5: Output

1. **Save** the report to `weekly-report-YYYY-MM-DD.md` in the current working directory
2. **Display** the full report in the terminal
3. **Notify** the user: "Report saved to weekly-report-YYYY-MM-DD.md"

## Important Notes

- If a repo has no commits from the user this week, include it with a note: "No commits this week."
- Handle errors gracefully — if a repo path doesn't exist or isn't accessible, report the error and continue with other repos
- SVN revision numbers should be displayed as `r<revision>` instead of git hashes
- When rewriting commit messages for clarity, preserve technical accuracy — don't invent details
