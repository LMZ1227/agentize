---
name: git-commit
description: Create a git commit with meaningful commit messages following project standards
---

# Git Commit Command

Execute the git-commit skill to commit staged changes with meaningful commit messages.

Use the `/git-commit` skill defined in `claude/skills/git-commit/` to create commits that follow the project's commit message standards.

This command will:
1. Analyze staged changes using `git diff --staged`
2. Review the commit message tag standards in `docs/git-msg-tags.md`
3. Create an appropriate commit message following the format defined in the git-commit skill
4. Execute the commit without bypassing pre-commit hooks

Invoke the skill: /git-commit
