# git-workflow-skills

Claude Code skills for Git release workflows — create PRs and trigger Android release builds without leaving the terminal.

## Skills

### `/pr` — Create Pull Request

Automates the full PR creation flow:

- Merges the latest target branch into your current branch
- Extracts ticket numbers (`XXX-NNNN`) from the commit log
- Detects PR templates at the repo level or org level via the GitHub API and fills them in automatically
- Previews the PR for confirmation before pushing

**Requires:** [GitHub CLI (`gh`)](https://cli.github.com) installed and authenticated

### `/release` — Android Release Build

Automates the Android release packaging flow:

- Merges the latest main branch
- Determines the next `VERSION_NUMBER` from existing `releases-*` tags
- Extracts ticket numbers from commits since the last release tag and writes them to `ci/release_notes.txt`
- Bumps `VERSION_NUMBER` in `app/build.gradle`
- Commits, tags, and pushes to trigger CI

Has two confirmation checkpoints — before writing files and before pushing — since the push triggers an irreversible CI build.

## Installation

```bash
npx skills add laxy-me/git-workflow-skills
```

## Usage

```
/pr                        # PR to default target branch (v{VERSION_NAME})
/pr <target-branch>        # PR to a specific branch
/release                   # Run the Android release workflow
```

## Requirements

| Skill | Requirement |
|-------|-------------|
| `/pr` | `gh` CLI installed and authenticated (`gh auth login`) |
| `/release` | Android project with `app/build.gradle` and `ci/release_notes.txt` |
