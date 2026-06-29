# my-skills — repository guide

This is my personal **skills repository**, shared across Claude Code, Codex, and
Cursor. It is the source of truth for my reusable skills and my shared Odoo agent
instructions; `setup.sh` symlinks them into each tool's global locations.

## What working in this repo means

When you're in this repo, the task is almost always to **author, edit, review, or
improve the skills and their supporting files** — not to perform the workflows the
skills describe.

- **Do not invoke these skills as workflows just because a request seems to match
  their description.** Here, each `SKILL.md` is an artifact to maintain, not a
  procedure to run. (If I explicitly type `/<skill-name>`, that's different —
  honor it.)
- Treat the skills' own instructions (e.g. the Odoo `[ticket-XXXX]` commit format,
  the staging-branch steps) as content you maintain, not rules you must follow
  while editing this repo.

## Layout

- `odoo-dev-skills/<skill-name>/SKILL.md` — one folder per skill.
- `odoo-agent.md` — shared Odoo agent instructions. `setup.sh` installs this as
  the global `~/.claude/CLAUDE.md` and `~/.codex/AGENTS.md`, so edits here change
  my global agent behavior in other projects.
- `setup.sh` — installs skills + agent instructions (symlinks). Re-run after
  adding a skill.
- `README.md` — human-facing overview.

## Editing skills

- Keep the `SKILL.md` frontmatter intact: `name`, `description`, `version`.
- Bump `version` when you change a skill's behavior.
- Match the existing tone and structure of the other skills, and keep each
  `description` specific enough that the right skill stays discoverable.
- Keep changes focused (KISS/YAGNI) — don't refactor unrelated skills.

## Commits in this repo

This meta-repo is not an Odoo project, so the Odoo rules from my global agent
instructions (per-change ticket identifier, always-plan-mode, pre-commit) do
**not** apply to changes made here. Use plain, descriptive commit messages in the
style of the existing history, ending with the `Co-Authored-By: Claude ...`
trailer. Only commit or push when I ask.
