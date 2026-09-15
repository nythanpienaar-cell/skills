Every skill is a folder under `skills/` holding a `SKILL.md`. Everything a skill needs lives inside its own folder and is linked by relative path. These skills run on strangers' machines (Windows, macOS, Linux), so never assume a local path, an OS, or anything already installed.

Anything inside `skills/` ships to users through both install routes. Keep drafts out of `skills/`.

Every `SKILL.md` is either user-invoked (`disable-model-invocation: true`, with a short human-facing `description`) or model-invoked (no flag, with a `description` that carries "Use when…" trigger phrasing).

When you add, rename, or remove a skill, or change what one does:

- Update its entry in `README.md` under **User-invoked** or **Model-invoked**, with the skill name linked to its `SKILL.md`.
- Bump `version` in `.claude-plugin/plugin.json` and add an entry to `CHANGELOG.md`. Installed users only see an update when the version changes.
- Run `claude plugin validate . --strict` (the marketplace) and `claude plugin validate .claude-plugin/plugin.json --strict` (the plugin). Keep this file in `.claude/`: a `CLAUDE.md` at the repo root fails the plugin check.

Test local changes before pushing with `claude --plugin-dir <path-to-this-repo>`.
