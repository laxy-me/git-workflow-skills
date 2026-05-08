---
name: release
description: Android release workflow — bump version, update release notes, commit and push a tag to trigger CI build
---

# Android Release Workflow

Execute the following steps in order to publish a release. Steps marked **[Confirm]** require user confirmation before continuing.

## Step 1: Fetch latest commits and tags

// turbo
```bash
git fetch origin
```

## Step 2: Determine version and merge main branch

1. Read `VERSION_NAME` and the current `VERSION_NUMBER` from `app/build.gradle`
2. Main branch name = `v{VERSION_NAME}` (e.g. if VERSION_NAME is `5.5.00`, main branch is `v5.5.00`)
3. Merge main branch:

```bash
git merge origin/v{VERSION_NAME}
```

If there are merge conflicts, **abort** and prompt the user to resolve conflicts manually, then re-run `/release`.

## Step 3: Determine new version number

1. Find the highest `releases-XXXX` number from remote tags:

// turbo
```bash
git tag -l 'releases-*' | sed 's/releases-//' | sort -n | tail -1
```

2. New VERSION_NUMBER = highest number + 1

## Step 4: Extract release notes

1. Find the most recent `releases-*` tag in the current branch's commit history:

// turbo
```bash
git describe --tags --match 'releases-*' --abbrev=0
```

2. Extract ticket numbers (`XXX-NNNN` format) from the git log between that tag and HEAD:

// turbo
```bash
git log {previous_tag}..HEAD --oneline | grep -oE '[A-Z]+-[0-9]+' | sort -u
```

3. Join the extracted ticket numbers with `, ` as the release notes content

## Step 5: [Confirm 1] Confirm version and release notes

Display the following for user confirmation:
- Current VERSION_NAME: `{VERSION_NAME}`
- New VERSION_NUMBER: `{new_version}`
- Release notes: `{ticket_list}`
- Commit message will be: `build: {VERSION_NAME}({new_version})`
- Tag will be: `releases-{new_version}`

**Wait for user confirmation or edits before continuing.**

## Step 6: Bump version

1. Update `VERSION_NUMBER` in `app/build.gradle` to the new version number (only the `def VERSION_NUMBER = "XXXX"` line)
2. Overwrite `ci/release_notes.txt` with the confirmed ticket list

## Step 7: Commit and tag

```bash
git add app/build.gradle ci/release_notes.txt
git commit -m "build: {VERSION_NAME}({new_version})"
git tag releases-{new_version}
```

Only stage these two files to avoid accidentally committing other local changes.

## Step 8: [Confirm 2] Confirm push

Display the following for user confirmation:
- Push branch: `origin/{current_branch}`
- Push tag: `releases-{new_version}`

**Wait for user confirmation before continuing.** The push will trigger a CI build and cannot be undone.

## Step 9: Push to remote

```bash
git push origin {current_branch}
git push origin releases-{new_version}
```

Once pushed, CI will automatically start the build.
