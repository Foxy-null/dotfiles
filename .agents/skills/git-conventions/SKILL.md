---
name: git-conventions
description: >-
  Apply Gitmoji commit messages and repository-specific pull request title rules
  when drafting commit messages, committing changes, creating pull requests, or
  editing PR titles. Choose the PR prefix from the change's purpose, never codex.
---

# Commit and pull request conventions

## Commit messages

- Format the subject as `<Gitmoji> <message>`: one actual Unicode Gitmoji emoji,
  one ASCII space, then a message describing the change. Do not include literal
  brackets or use shortcodes such as `:sparkles:`.
- Choose the Gitmoji from the actual changes: for example, `✨` for a feature,
  `🐛` for a fix, or `📝` for documentation.
- Example: `🐛 空の入力で発生するエラーを修正`.
- An optional commit body follows the subject; this format applies to the subject.

## Pull request titles

- Inspect the target repository's `.github/workflows/` and any title-validation
  scripts or configuration they reference before drafting the title. Follow the
  required format, allowed prefixes, scopes, and other title constraints.
- Choose an allowed prefix that describes the PR's actual changes. Never use
  `codex`, `codex:`, or `[codex]` as the title prefix. A branch name starting with
  `codex/` does not determine the PR title.
- If workflows do not define a title format, follow any title rules in the target
  repository's `AGENTS.md` or contribution guide. If none exist, use
  `<type>: <summary>` with a suitable Conventional Commits type such as `feat`,
  `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, or `revert`.
- Example fallback: `docs: コミットとPRの命名規則を追加`.
- Commit subjects and PR titles have separate formats. Add a Gitmoji to a PR title
  only when the repository's title rules require it.
- Verify the title against the discovered rules before creating or updating the
  PR. Run an existing title validator when it can be run locally, and explicitly
  pass the chosen title to the PR tool or command.
