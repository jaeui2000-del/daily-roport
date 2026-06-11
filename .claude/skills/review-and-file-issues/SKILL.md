---
name: review-and-file-issues
description: >
  Reviews recent changes in a git repository, finds bugs and improvement
  opportunities, fixes bugs directly (commit + push), and files enhancements
  as GitHub Issues via the REST API. Use this skill whenever the user asks to
  "review and file issues", "find bugs and register issues", "코드 리뷰 후
  이슈 등록", "버그 찾아서 이슈로 등록", or any variation of reviewing
  recent work and logging findings to GitHub. Trigger even if the user says
  things like "방금 작업한 내용 확인해서 이슈 만들어줘" or "검토 후 깃허브에
  올려줘". This skill handles the full end-to-end flow: review → fix → push →
  file issues.
---

# Review & File Issues

End-to-end workflow: review recent changes → fix bugs in-place → push → file improvement ideas as GitHub Issues.

## Overview

1. **Discover** what changed recently (git log + diff + read files)
2. **Classify** findings into bugs (fix now) vs. enhancements (file as issues)
3. **Fix** bugs using the Edit tool, then commit and push
4. **File** enhancement issues via the GitHub REST API
5. **Report** a summary to the user

---

## Step 1 — Understand recent changes

Run these in parallel:

```powershell
git log --oneline -10
git diff HEAD~1 HEAD --stat
git remote -v
```

Then read the files that changed. For HTML/CSS/JS, also scan for:
- Hardcoded content that should be dynamic (e.g., dates, weekday names)
- Missing security attributes on links (`rel="noopener noreferrer"` for `target="_blank"`)
- Broken or placeholder URLs
- Missing referenced files (scripts, stylesheets, etc.)

For any project type, also consider:
- Discrepancy between README/docs and actual code/files
- Missing automation scripts that are mentioned but not present
- Accessibility or UX gaps (no index page, no dark mode, etc.)

---

## Step 2 — Classify findings

Split every finding into one of two buckets:

| Type | Criterion | Action |
|------|-----------|--------|
| **Bug** | Incorrect behavior or data, security flaw, broken link, factual error | Fix immediately, commit, push |
| **Enhancement** | Missing feature, UX improvement, refactor opportunity, docs gap | File as GitHub Issue |

When in doubt, prefer filing an issue over silently skipping it. A borderline finding is better logged than lost.

---

## Step 3 — Fix bugs

Use the **Edit tool** (not PowerShell `Set-Content`) to modify files — `Set-Content` corrupts UTF-8/Korean text on Windows.

After all fixes:

```powershell
git add <changed files>
git commit -m "Fix <concise summary of bugs fixed>"
git push origin <current branch>
```

If there are no bugs, skip this step.

---

## Step 4 — File enhancement issues

### 4a. Get the GitHub repo slug

From `git remote -v` output, extract `owner/repo` from the origin URL.

Example: `https://github.com/alice/my-project.git` → `alice/my-project`

### 4b. Get a GitHub token

```bash
git credential fill <<'EOF'
protocol=https
host=github.com
EOF
```

The `password` field in the output is the token. If this fails (no stored credentials), ask the user to provide a token or run `git push` first to trigger credential storage.

### 4c. Create each issue

Write each issue body to a **temp JSON file** first (so Korean/Unicode characters survive the shell), then post with curl. Do NOT try to inline the JSON in the curl command — encoding breaks.

```powershell
# Write issue to temp file
$issue = @{
    title = "[Enhancement] <issue title>"
    body  = "## Problem`n<description>`n`n## Suggested fix`n<suggestion>"
    labels = @("enhancement")
} | ConvertTo-Json -Depth 3
[System.IO.File]::WriteAllText("$env:TEMP\issue_1.json", $issue, [System.Text.Encoding]::UTF8)
```

Then post with curl (available in Windows 10+):

```powershell
curl -s -X POST "https://api.github.com/repos/$repo/issues" `
  -H "Authorization: token $token" `
  -H "Content-Type: application/json" `
  --data-binary "@$env:TEMP\issue_1.json"
```

Or use the Bash tool with a heredoc:

```bash
curl -s -X POST "https://api.github.com/repos/$REPO/issues" \
  -H "Authorization: token $TOKEN" \
  -H "Content-Type: application/json" \
  --data-binary "@/tmp/issue_1.json"
```

After all issues are posted, delete the temp files.

### 4d. Capture issue numbers and URLs

Parse each curl response for `"number"` and `"html_url"` to include in the final summary.

---

## Step 5 — Report summary

Present a clean summary:

```
## Review complete

### Bugs fixed (committed & pushed)
- [<filename>] <bug description>
- [<filename>] <bug description>

### Issues filed
- #N  [Enhancement] <title> — https://github.com/<owner>/<repo>/issues/N
- #N+1 [Enhancement] <title> — https://github.com/<owner>/<repo>/issues/N+1

### No findings
(If nothing was found, state this clearly rather than omitting the section.)
```

---

## Environment notes (Windows)

- **Shell**: PowerShell is default; Bash tool is also available for POSIX-style commands
- **Python**: may not be available — do not assume it is
- **GitHub CLI (`gh`)**: may not be installed — use `curl` + `git credential fill` instead
- **File encoding**: always use the Edit tool for modifying source files; avoid `Set-Content` / `Out-File` with non-ASCII content
- **Temp files**: use `$env:TEMP` for JSON payloads; clean up after use
