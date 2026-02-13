---
name: commit-push-pr
description: Commit, push, and create a pull request in one workflow. Use when the user invokes /commit-push-pr or asks to commit and create a PR. Separates changes into purpose-based commits with conventional commit messages in English, pushes to remote, and opens a PR to the upstream repository with Korean title and body.
---

# Commit, Push, and Create PR

## Workflow

### 1. Analyze Changes

Run `git status` and `git diff` (staged + unstaged) to understand all pending changes. Group related changes by purpose (feature, bugfix, refactor, docs, chore, etc.).

### 2. Create Purpose-Separated Commits

For each logical group of changes, stage only the relevant files and create a commit.

Commit message format (English):
```
<type>: <concise description>
```

Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `style`, `perf`, `ci`, `build`

Examples:
- `feat: add user authentication endpoint`
- `fix: resolve null pointer in payment processing`
- `docs: update API usage examples`
- `chore: bump dependency versions`

Rules:
- One commit per logical purpose. Do NOT bundle unrelated changes.
- Commit message must be in **English**.
- Use imperative mood ("add", not "added" or "adds").
- Use HEREDOC for commit messages to ensure proper formatting.

### 3. Push to Remote

Push the current branch to the remote with `-u` flag:
```bash
git push -u origin <branch-name>
```

### 4. Create PR (only if upstream exists)

Check remotes:
```bash
git remote -v
```

- If an `upstream` remote exists → create a PR targeting the upstream repository.
- If **no** `upstream` remote exists → **skip PR creation**. Just inform the user that commits have been pushed.

When creating a PR, use `gh pr create`. Title and body must be in **Korean**.

```bash
gh pr create --repo <upstream-owner>/<repo> --title "<Korean title>" --body "$(cat <<'EOF'
## 요약
- <변경사항 요약 bullet points>

## 변경 내용
- <상세 변경 내용>

## 테스트
- <테스트 관련 내용>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

Rules:
- PR title and body must be in **Korean**.
- Use `--repo` flag to target the upstream repository.
- Return the PR URL to the user when done.
- If no upstream remote, do NOT create a PR. Only commit and push.
