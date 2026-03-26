# Git Workflow Skills

> [한국어](README.ko.md)

A collection of safe, opinionated git workflow skills for Claude Code.

## Skills

| Skill | Slash Command | Description |
|-------|---------------|-------------|
| git-commit | `/git-commit` | Safe staging and committing with Conventional Commits |
| git-pr | `/git-pr` | Push branch and create a GitHub PR |
| git-review | `/git-review` | Analyze PR diff and submit a code review |
| git-merge-pr | `/git-merge-pr` | Squash-merge a PR safely |
| git-pr-done | `/git-pr-done` | Full PR lifecycle: commit → PR → review → merge → cleanup |
| git-cleanup | `/git-cleanup` | Remove worktree, delete local branches, prune remotes |

## Safety Philosophy

All skills enforce:

- Mandatory user confirmation before destructive or irreversible operations
- No `git add -A` or `git add .` (prevents accidental sensitive file inclusion)
- No force push unless explicitly requested by the user
- No `--force`, `--admin`, or `--auto` flags on `gh pr merge`
- Automatic exclusion of sensitive files (`.env`, `*.pem`, `*.key`, `credentials.json`, etc.)
- Confirmation steps cannot be bypassed by urgency statements

## Skill Relationships

`git-pr-done` is an orchestrator that delegates to the other five skills in sequence:

```
git-pr-done
  ├─ Step 1: git-commit    (if uncommitted changes exist)
  ├─ Step 2: git-pr        (if no open PR exists)
  ├─ Step 3: git-review
  ├─ Step 4: git-merge-pr  (always requires confirmation)
  └─ Step 5: git-cleanup   (always requires confirmation)
```

Steps 4 and 5 always require explicit confirmation regardless of auto mode.

## Project-Specific Customization

Skills automatically adapt to project conventions:

- **git-commit** — reads `git log` history to match the project's existing commit style
- **git-pr** — detects the repository's default branch via `gh repo view` instead of hardcoding `main`
- **git-review** — detects languages in the PR diff, loads matching language-specific reviewer skills, and falls back to general best practices when no reviewer skill is available
- **git-merge-pr** — detects the default branch dynamically
- **git-cleanup** — preserves Claude Code local settings (`.claude/settings.local.json`) during worktree removal

## Installation

```bash
claude plugin marketplace add https://github.com/ppzxc/engineering-guidelines.git
```
