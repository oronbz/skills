---
name: gett-push
description: "MUST trigger on: commit, push, commit and push, create PR, ship code, /commit, or any git operation that modifies history or the remote — in ANY repo whose `git remote -v` contains `gtforge`. Check the remote FIRST. If not gtforge, skip this skill. Manages Gett's full git workflow: branch naming, Jira-linked commits, single-commit-per-PR, and auto-PR creation."
---

# Gett Push — Git Workflow for gtforge Repos

## Quick Check

Run `git remote -v`. If the remote does **not** contain `gtforge`, stop here — this skill does not apply. Fall back to normal git behavior.

## The Pipeline

Every commit/push/PR request follows this pipeline. Execute all applicable steps — if the user says "commit", that means commit + push + PR. The goal is to get changes reviewed with minimal friction.

### Step 1: Jira Ticket

Look for a Jira ticket in the conversation. The user may provide:
- A full URL: `https://gett.atlassian.net/browse/GETT-12345`
- Just the ID: `GETT-12345`

If no ticket was mentioned, ask the user for one. They can say "none" — that's fine, proceed without it.

**Fetching the story name:** If the Atlassian MCP tools are available (`getJiraIssue`), use them to fetch the ticket summary. If the MCP is not available, infer a concise story name from the diff and the conversation context, then confirm with the user (e.g., "I'd call this 'No validation error on screenshotDisabled text fields' — sound right?").

**Cleaning the story name:** Jira titles often have cruft. Strip these:
- Platform prefixes: "Rider", "iOS", "Android", "Backend", "Server"
- Leading punctuation: hyphens, dashes, colons, pipes after removing prefixes
- Ticket IDs embedded in the title
- Leading/trailing whitespace

Examples:
- `"Rider iOS - No validation error on screenshotDisabled text fields"` → `"No validation error on screenshotDisabled text fields"`
- `"iOS: Login button unresponsive on iPad"` → `"Login button unresponsive on iPad"`
- `"GETT-12345 - Fix crash on order creation"` → `"Fix crash on order creation"`

### Step 2: Determine Type

Classify the work as one of: **Infra**, **Bug**, **Feature**, **Crash**

Infer from context:
- The Jira ticket type (bug, story, task, etc.)
- The nature of the code changes
- The branch name if one already exists
- What the user said

If it's ambiguous, ask. Use lowercase for branch names (`bug/`, `feature/`, `infra/`, `crash/`) and capitalized for commit messages (`Bug -`, `Feature -`, `Infra -`, `Crash -`).

### Step 3: Branch

Check the current branch with `git branch --show-current`.

**If on `master`:**
Create a new branch from the cleaned Jira story name:
- Format: `[type]/[descriptive_name_in_snake_case]`
- Keep it concise but readable — aim for 3-6 words
- Use underscores, not hyphens, for word separation in the name part

```
git checkout -b bug/no_validation_error_on_screenshot_fields
```

Examples:
- `bug/no_navigation_bar_on_ob_screen`
- `feature/login_redesign`
- `infra/refactor_login_to_swift`
- `crash/nil_pointer_on_order_creation`

**If already on a branch:** Stay on it. Don't rename it even if it doesn't perfectly match the convention.

### Step 4: Commit

**Stage changes:** Stage all modified/new files relevant to the task. Be mindful — don't stage `.env`, credentials, or files clearly unrelated to the work.

**Check commits ahead of master:**
```
git rev-list --count master..HEAD
```

- **0 commits ahead:** Create a new commit
- **1 commit ahead:** Amend the existing commit (`git commit --amend`)
- **2+ commits ahead:** Amend the last commit, but mention to the user that there are multiple commits on this branch (the convention is one commit per PR — they may want to squash)

**Commit message format** — exactly two lines (or one if no Jira ticket):

```
[Type] - [Clean story name]
[Jira URL]
```

Real examples from the repo:
```
Bug - No validation error on screenshotDisabled text fields in add card
https://gett.atlassian.net/browse/GETT-163552
```
```
Bug - additional space in OB screen
https://gett.atlassian.net/browse/GETT-164648
```
```
Infra - Add daily repo status (Agentic Workflow)
```

Use a HEREDOC to pass the message so the newline is preserved:
```bash
git commit -m "$(cat <<'EOF'
Bug - No validation error on screenshotDisabled text fields in add card
https://gett.atlassian.net/browse/GETT-163552
EOF
)"
```

For amends, same format but with `--amend`:
```bash
git commit --amend -m "$(cat <<'EOF'
Bug - No validation error on screenshotDisabled text fields in add card
https://gett.atlassian.net/browse/GETT-163552
EOF
)"
```

### Step 5: Push

Push immediately after committing:

- **New branch / first push:** `git push -u origin [branch-name]`
- **Existing tracking branch:** `git push`

If the push is rejected (typically after an amend), **ask the user** before force pushing:
> "The push was rejected because the remote has diverged (likely from the amend). Want me to force push?"

Only force push with explicit user confirmation: `git push --force-with-lease`

### Step 6: Pull Request

Check if a PR already exists:
```bash
gh pr list --head [branch-name] --state open
```

**If no PR exists**, ask the user if they'd like to create one. If they decline, stop here — just report the push was successful. If they confirm, create one:
- **Title:** The first line of the commit message (e.g., `Bug - No validation error on screenshotDisabled text fields in add card`)
- **Body:** The Jira URL, or the ticket ID linked as URL `https://gett.atlassian.net/browse/GETT-XXXXX`. If no ticket, body can reference the commit description or be left minimal.
- **Base:** `master`
- **Assignee:** The current user (`@me`)
- **Label:** The type in lowercase — `bug`, `feature`, `infra`, `crash`, `design`, etc. Use whichever label matches the work type. If the label doesn't exist on the repo, skip it rather than erroring.

```bash
gh pr create --title "Bug - No validation error on screenshotDisabled text fields in add card" --body "$(cat <<'EOF'
https://gett.atlassian.net/browse/GETT-163552
EOF
)" --base master --assignee @me --label bug
```

**If a PR already exists**, no action needed — the amend + force push updates it automatically.

Report the PR URL to the user when done.

### Step 7: Update Jira Status

If the Atlassian MCP tools are available and a Jira ticket was provided:
- Use `getTransitionsForJiraIssue` to find the transition that moves the ticket to **"In Code Review"** (or the closest matching status)
- Use `transitionJiraIssue` to apply that transition

This happens after the PR is created/updated — the code is now out for review, so the Jira status should reflect that. If the transition fails (e.g., the ticket is already in that status, or the status doesn't exist), just mention it to the user and move on. Don't block the workflow over it.

## Summary of Conventions

| Element | Format | Example |
|---------|--------|---------|
| Branch | `type/snake_case_name` | `bug/no_navigation_bar_on_ob_screen` |
| Commit line 1 | `Type - Clean description` | `Bug - No navigation bar on OB screen` |
| Commit line 2 | Jira URL (optional) | `https://gett.atlassian.net/browse/GETT-164648` |
| PR title | Same as commit line 1 | `Bug - No navigation bar on OB screen` |
| PR body | Jira URL | `https://gett.atlassian.net/browse/GETT-164648` |
| Types | `Infra`, `Bug`, `Feature`, `Crash` | — |