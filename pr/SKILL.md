---
name: pr
description: Create a GitHub PR from the current branch to the main branch (default v{VERSION_NAME})
---

# Create Pull Request

Use GitHub CLI (`gh`) to create a PR from the current branch. The default target branch is `v{VERSION_NAME}`. The user may specify a different target branch.

## Prerequisites

Verify `gh` is installed and authenticated:

// turbo
```bash
gh auth status
```

If not authenticated, prompt the user to run `gh auth login` and re-run `/pr` afterward.

## Step 1: Determine target branch

// turbo
1. Get the current branch name:
```bash
git branch --show-current
```

2. Read `VERSION_NAME` from `app/build.gradle` to derive the default target branch `v{VERSION_NAME}` (e.g. `v5.5.00`)

3. If the user specified a target branch when invoking `/pr`, use that instead of the default

## Step 2: Merge from target branch

1. Fetch latest from remote:

// turbo
```bash
git fetch origin
```

2. Merge the target branch into the current branch:

// turbo
```bash
git merge origin/{target_branch}
```

If there are merge conflicts, **abort** and prompt the user to resolve conflicts manually, then re-run `/pr`.

## Step 3: Extract ticket numbers

Extract ticket numbers (`XXX-NNNN` format) from commits between the current branch and the target branch:

// turbo
```bash
git log origin/{target_branch}..HEAD --oneline | grep -oE '[A-Z]+-[0-9]+' | sort -u
```

## Step 4: Detect PR template and compose description

Get the repository owner and name:

// turbo
```bash
gh repo view --json owner,name --jq '"\(.owner.login)/\(.name)"'
```

Look up the PR template via the GitHub API in order of priority:

1. Repo-level template:
// turbo
```bash
gh api repos/{owner}/{repo}/contents/.github/pull_request_template.md --jq '.content' | base64 -d
```

2. If not found (API returns 404), fall back to the org-level template:
// turbo
```bash
gh api repos/{owner}/.github/contents/.github/pull_request_template.md --jq '.content' | base64 -d
```

3. If the org uses multiple templates (`.github/PULL_REQUEST_TEMPLATE/` directory), list them:
// turbo
```bash
gh api repos/{owner}/.github/contents/.github/PULL_REQUEST_TEMPLATE --jq '.[].name'
```
Use `ask_user_question` to let the user select one, then fetch its content via the API.

**Case A — No template found at either level:**
Use the ticket list directly as the PR description body.

**Case B — Template found:**
Inject the ticket list into the appropriate section of the template (e.g. `## Description`, `## Changes`, or the first available placeholder). Use the filled template as the PR description body.

## Step 5: [Confirm] Preview PR

Display the following for user confirmation:
- **Source branch**: `{current_branch}`
- **Target branch**: `{target_branch}`
- **PR title**: `{current_branch_name}`
- **PR description**: full composed body (with template filled in if applicable)

Use the `ask_user_question` tool to present button options:
- **Confirm**: proceed to Step 6
- **Edit title**: let the user input a new title, then continue
- **Edit description**: let the user edit the description, then continue
- **Cancel**: abort the workflow

## Step 6: Push and create PR

1. Push the current branch to remote:

// turbo
```bash
git push origin {current_branch}
```

2. Create the PR:

// turbo
```bash
gh pr create --base {target_branch} --head {current_branch} --title "{pr_title}" --body "{pr_description}"
```

Display the PR link after successful creation.
