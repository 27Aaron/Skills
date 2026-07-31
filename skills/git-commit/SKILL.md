---
name: git-commit
description: Create or draft Conventional Commit messages from repository changes. Use for committing changes, writing or rewriting commit messages, choosing type, scope, or breaking-change markers, and fixing commitlint message errors. Stage or commit only when explicitly requested.
---

# Git Commit with Conventional Commits

Analyze the actual changes and create a clear, focused commit using [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/). Follow repository-specific instructions and commitlint rules when present.

## Decide the Action

- Treat requests to write, generate, rewrite, suggest, or review a commit message as message-only. Inspect read-only state when useful, then return the proposed message without staging or committing.
- Execute a commit only when the user explicitly asks to commit, create the commit, or otherwise save the changes in Git.
- Default ambiguous requests to message-only rather than mutating the repository.

## Format

```text
<type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

## Types

The specification defines `feat`, `fix`, and breaking-change semantics. The other types below are common conventions; repository rules take precedence.

| Type       | Use for                                    |
| ---------- | ------------------------------------------ |
| `feat`     | New feature                                |
| `fix`      | Bug fix                                    |
| `docs`     | Documentation only                         |
| `style`    | Formatting with no logic change            |
| `refactor` | Restructuring without a feature or bug fix |
| `perf`     | Performance improvement                    |
| `test`     | Adding or updating tests                   |
| `build`    | Build system or dependencies               |
| `ci`       | CI configuration or workflows              |
| `chore`    | Maintenance not covered by another type    |
| `revert`   | Reverting changes when tooling supports it  |

## Workflow

### 1. Analyze Changes

```bash
git status --short
git diff --staged
git diff
```

Always inspect both staged and unstaged state. Treat partially staged files as separate snapshots.

- When staged changes exist, use the staged diff as the commit source of truth while accounting for any unstaged changes that will remain.
- When nothing is staged, use the working-tree diff to plan a logical commit.
- Check applicable repository instructions, commitlint configuration, and recent commit subjects when needed to match established language, scopes, and style.

### 2. Stage a Logical Change

Stage files or hunks only for an explicit commit request. Keep one logical change per commit and avoid unrelated files.

```bash
git add path/to/file
git add -p
git diff --staged --check
git diff --staged
```

- Preserve an existing staged snapshot; do not add unstaged changes unless the user's request includes them.
- When the user explicitly asks to commit all current changes, inspect them first, then stage all intended files.
- Never stage secrets or private keys. Inspect credential and environment files carefully without exposing sensitive values.
- After staging, run the staged diff checks above and base the message only on that exact snapshot.

### 3. Generate the Message

- Choose the type from the user-visible outcome, not merely from the kinds of files changed.
- Prefer a specific type over `chore`; use `chore` only when no more precise type applies.
- Use a scope only when the affected component is clear.
- Write a specific, imperative description such as `add`, `fix`, or `remove`.
- Follow repository language and rules; otherwise match the user's language.
- Keep the header concise and avoid a trailing period. Prefer at most 72 characters when the repository defines no limit.
- Add a body only when the reason, context, or impact is not clear from the header; do not repeat a file list.
- Do not invent issue references, reviewers, co-authors, or sign-offs.

For breaking changes, add `!` before `:` and/or a `BREAKING CHANGE:` footer. When using only `!`, describe what breaks in the header.

```text
feat(api)!: remove the legacy search endpoint

BREAKING CHANGE: use the v2 search endpoint instead
```

### 4. Execute the Commit

Run `git commit` only when the user explicitly asks to commit. Pass the complete message through stdin with `git commit --file=-` or another argv-safe interface; do not interpolate generated text into a shell command.

Allow hooks to run; do not bypass them automatically. If a hook fails:

- Inspect the hook output and `git status --short`.
- Fix and retry only when the cause is within the intended change.
- Reinspect the staged diff when a hook modifies files; never restage blindly.
- Stop and report failures involving unrelated files, expanded scope, or missing authority.

Verify the result with:

```bash
git status --short
git log -1 --format='%H%n%s'
```

Report the commit hash and subject, plus any changes still left staged or unstaged.

## Safety

- Never change Git configuration.
- Never use `--no-verify`, destructive commands, or history rewriting unless explicitly requested.
- Never force-push `main` or `master`; do not force-push another branch unless explicitly requested.
- Never push merely because the user asked to commit.
