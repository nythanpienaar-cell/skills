# Changelog

## 0.3.1

- `establish-architecture` reads the spec from the issue tracker (`/to-spec` publishes it as an issue, not a file) and can cite issues as sources.
- `design-system` reads `CONTEXT.md` and the spec issue for context instead of `docs/context.md` and `docs/PRD.md`, and never edits either.

## 0.3.0

- Add `design-system` (modified from BuilderOS, MIT); its closing line now points at `/to-tickets`.
- `pipeline-setup` installs this plugin from `nythanpienaar-cell/skills` (replacing the placeholder), and its docs now match current `establish-architecture`, `audit-architecture`, and Matt Pocock's skills 1.2.3: `/implement` reviews and commits but doesn't push, the stale `qa` skill is gone, and `/wizard` and `/resolving-merge-conflicts` are listed as tools.
- `pipeline-setup` ends by pointing the user at `/tutorial`.

## 0.2.0

- Add `establish-architecture` and `audit-architecture`.

## 0.1.0

- First release: `pipeline-setup` and `tutorial`.
