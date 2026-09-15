# Pipeline.md format

The exact shape of `docs/Pipeline.md`, written in step 5. Its reader is the Part 2 coaching skill, which relies on these headings — keep every heading, in this order, spelled exactly. Its second reader is a beginner: plain language, no Playout component IDs.

## Rules

- **Written only on a clear recheck** (Playout §14.1), so the top line is always true.
- **Regenerated whole** on every clear run — it is a generated file, and says so.
- **Include only what this project has.** A stage or tool gated off by the project shape is left out of *The pipeline* and appears under *Not installed for this project* instead. Stage 11 and the automation tools appear only when the recorded automation value is `ralph` or `sandcastle`.
- **Fill Reads and Writes from the installed skills themselves.** Open each included skill's `SKILL.md` and record what it actually reads and writes — never from memory, since skills change between versions. Stages with no skill behind them (0 Fog check, 11 Automation) take theirs from Playout instead — for stage 11, its §11 L9 components.
- Replace every `{placeholder}`; none survives.

## Template

````markdown
# Pipeline

> **Everything is installed.** Checked by `/pipeline-setup` on {YYYY-MM-DD}. This file is generated — run `/pipeline-setup` again rather than editing it.

## Install report

| Area | Status | Details |
|---|---|---|
| Computer basics | Ready | Node {version}, Git {version}{, Git Bash — Windows only} |
| Claude Code | Ready | CLI {version}{; desktop app} |
| GitHub | Ready | Signed in as {login} · repo {owner/name} ({private/public}) · labels ready |
| Pipeline skills | Ready | Matt Pocock bundle · Nythan bundle (tutorial, establish-architecture, audit-architecture{, design-system}){ · launch-checklist} |
| MCP servers | Ready | Context7{ · Chrome DevTools}{ · Figma}{ · Supabase} |
| Project setup | Ready | `docs/agents/` · agent skills block in `{CLAUDE.md/AGENTS.md}` |
| Online services | {Ready/Not used} | {Supabase project linked}{ · Netlify site linked} |
| Automation | {Locked/Ready/Needs repair/Not used} | {Locked: run `/pipeline-setup automations` when you're ready for it / Ralph Loop{ · Sandcastle} / what failed — run `/pipeline-setup automations` to repair / You chose not to use automation} |

## Not installed for this project

{One bullet per Not-needed component, or exactly: `Nothing — this project needed every tool.`}

- **{Plain name}** — {what it's for, one sentence}. Left out because you answered *{answer}* to "{question}". {It is still installed from an earlier project. / To add it, run `/pipeline-setup` and answer *yes*.}

Run `/pipeline-setup` at the start of every new project, and again in this one if what it needs changes.

## Project shape

- Has screens people look at: {yes/no}
- Designs in Figma: {yes/no/—}
- Remembers information (database): {yes/no}
- Public on the internet: {yes/no}
- Issue tracker: {GitHub/GitLab/local/other}
- Sub-issues: {native/task list} · Blocking between tickets: {native/"Blocked by" line}

## The pipeline

### Stages

| # | Stage | Command | Who starts it | Reads | Writes | Use it when |
|---|---|---|---|---|---|---|
{one row per included stage, from the stage list below}

### Tools the stages lean on

| Tool | What it's for | Used in |
|---|---|---|
{one row per included tool, from the tool list below}

### Ground rules

{one bullet per rule below that applies to this project}
````

## Stage list

Source for the *Stages* table, in pipeline order. **Who starts it**: *you type it* for a skill with `disable-model-invocation: true`, otherwise *you or Claude* — check each installed `SKILL.md`.

| # | Stage | Command | Gate | Use it when |
|---|---|---|---|---|
| 0 | Fog check | — (a question, not a tool) | always | Every project: can one conversation produce a complete plan? Yes → 2. No → 1. |
| 1 | Wayfinder | `/wayfinder` | tracker supports it | Big or unclear ideas only. Resolves open decisions one at a time, then goes straight to 3 — it replaces the grill session. |
| 2 | Grill session | `/grill-with-docs` | always | Every project that didn't use Wayfinder. Interviews you until nothing about the idea is ambiguous. |
| 3 | Spec | `/to-spec` | always | Every project. Turns the shared understanding into checkable requirements. |
| 4 | Architecture foundation | `/establish-architecture` | always | Once per project, right after the spec and before any code: it designs the structure every feature is built to and writes it to `docs/architecture.md`. It changes no code. |
| 5 | Design system | `/design-system` | has screens | After the architecture, before tickets, so every screen follows the same visual rules. |
| 6 | Tickets | `/to-tickets` | always | Every project. Slices the spec into small tickets that each work end to end. |
| 7 | Build each ticket | `/implement` | always | Every ticket: builds test-first with `/tdd`, runs the tests, reviews its own work with `/code-review`, then commits to the current branch. It does not push. |
| 8 | Review | `/code-review` | always | Before merging a branch: review everything since the branch started, against the project's standards and the ticket's spec. |
| 9 | Triage | `/triage` | `triage` installed | Whenever new issues arrive. |
| 10 | Launch | `/launch-checklist` | public on the internet | Once, at the end, before going live. |
| 11 | Automation | Ralph Loop{ · Sandcastle} | automation is `ralph` or `sandcastle` | Running stage 7 unattended — only after doing it by hand. |

## Tool list

| Tool | What it's for | Used in | Gate |
|---|---|---|---|
| GitHub + `gh` | Where tickets, labels, and code backups live | 1, 6, 7, 8, 9 | tracker = GitHub |
| Git{ + Git Bash} | Saves snapshots of the work; Git Bash runs setup scripts on Windows | 7 | always |
| Context7 MCP | Current library documentation, so code matches today's APIs | 7 | always |
| Chrome DevTools MCP | Lets Claude see and click through a real browser | 5, 7, 8 | has screens |
| Playwright | Runs the browser tests `/tdd` writes | 7 | has screens |
| Figma MCP | Lets `/design-system` read Figma designs | 5 | designs in Figma |
| Supabase CLI + MCP | The database: local testing, migrations, schema, logs | 7 | remembers information |
| Netlify CLI | Puts the project online | 10 | public on the internet |
| `/diagnosing-bugs` | Structured hunt for a bug's real cause | 7 | always |
| `/prototype` | A throwaway build to answer a design question | 1, 5 | always |
| `/research` | Reading official sources and saving the findings | 1, 2 | always |
| `/domain-modeling`, `/codebase-design` | The project's shared vocabulary and module design | 2, 4, 7 | always |
| `/audit-architecture` | Finds code that has drifted from `docs/architecture.md` and writes findings you can fix or turn into tickets | after 7, every few tickets | stage 4 has run |
| `/improve-codebase-architecture` | Surveys the code for modules worth deepening and shows them as a visual report | after 7, every few days | always |
| `/wizard` | Turns steps only you can do (dashboards, secrets, sign-ups) into a guided script | 7, 10 | always |
| `/resolving-merge-conflicts` | Works through a merge conflict by what each side meant | 7, 8 | always |
| Docker | Runs Sandcastle's agents inside sealed-off containers, so they can't touch the rest of the computer | 11 | automation is `sandcastle` |
| Sandcastle | Runs unattended agents on tickets, each inside its own container | 11 | automation is `sandcastle` |
| Claude Code token | Lets Sandcastle's agents use your Claude plan; stored only in `.sandcastle/.env` | 11 | automation is `sandcastle` |

## Ground rules

| Rule | Applies when |
|---|---|
| Push after every ticket `/implement` commits. | auto-commit was accepted |
| `docs/design.md` is the source of truth for every screen; change it there before building against a new value. | has screens |
| Build every feature in the slice shape `docs/architecture.md` describes. Where it leaves a choice open, raise it rather than decide silently; a decision worth keeping becomes an ADR. | after stage 4 has run |
| Start each new feature with the slice script `/establish-architecture` wrote. | a slice script was written in stage 4 |
| Never put keys or passwords in the repo — they go in `.env`, which Git ignores. | remembers information or public |
