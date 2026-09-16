# Skills

Agent skills for setting up an AI coding pipeline, learning to use it, and keeping the code's architecture in shape. They're built for anyone, on any machine, starting from nothing.

## Before you start

You need **Claude Code**, installed and signed in with a plan that includes it: the desktop app ([claude.ai/download](https://claude.ai/download)) or the terminal version ([setup guide](https://code.claude.com/docs/en/setup)). That's all. `/pipeline-setup` installs everything else the pipeline needs, including [Matt Pocock's skills](https://github.com/mattpocock/skills).

## Install

Pick **one** of these routes. Installing two gives you every skill twice. Whichever you pick, **restart Claude Code afterwards**: new skills load in the next session.

<details open>
<summary><strong>Claude Code desktop app (easiest)</strong></summary>

Open the **Code** tab and send Claude this message:

```
Install the nythan-skills plugin for me: run `claude plugin marketplace add nythanpienaar-cell/skills`, then `claude plugin install nythan-skills@nythan --scope user`. If the `claude` command isn't found, install the Claude Code command-line tool first, following Anthropic's setup guide.
```

Don't type `/plugin …` into the desktop chat box: slash commands like that only work in the terminal version, and in the app nothing happens.

</details>

<details>
<summary><strong>Claude Code in a terminal</strong></summary>

In a normal terminal:

```bash
claude plugin marketplace add nythanpienaar-cell/skills
claude plugin install nythan-skills@nythan --scope user
```

Or from inside a running `claude` session:

```
/plugin marketplace add nythanpienaar-cell/skills
/plugin install nythan-skills@nythan
```

You get every skill as one bundle, and updates arrive when a new version ships.

</details>

<details>
<summary><strong>Other agents, or if you want to edit the skills</strong></summary>

```bash
npx skills@latest add nythanpienaar-cell/skills
```

This copies the skills into your project as ordinary files you own and can change. Pick which skills to take. To get just one:

```bash
npx skills@latest add nythanpienaar-cell/skills --skill=tutorial
```

Pull later changes with `npx skills update`.

</details>

## How to use them

1. Open your project folder in Claude Code (a new, empty folder is fine) and run `/pipeline-setup`. It checks your computer, installs what's missing, and writes `docs/Pipeline.md`.
2. Run `/tutorial` whenever you're not sure what to do next. It coaches you through the pipeline one step at a time.
3. Run `/establish-architecture` once the project has a plan, to set the structure new code builds to. Run `/audit-architecture` from time to time to find code that has drifted from it.
4. If the project has screens, run `/design-system` with a screenshot or mockup to set its visual rules.

## Skills

**User-invoked:** these only run when you type them.

- **[pipeline-setup](./skills/pipeline-setup/SKILL.md):** Checks this computer and project, installs everything the AI coding pipeline needs, and writes `docs/Pipeline.md`.
- **[establish-architecture](./skills/establish-architecture/SKILL.md):** Designs the project's four-layer architecture foundation (modular monolith, vertical slices, hexagonal ports, deep modules) from its existing plan, so implementation work builds on it.
- **[audit-architecture](./skills/audit-architecture/SKILL.md):** Audits existing code against that foundation and writes evidence-backed findings, each ready to fix directly or hand to spec, ticket, and ADR skills.

The architecture skills use vocabulary and follow-up commands from [Matt Pocock's skills](https://github.com/mattpocock/skills) (`/codebase-design`, `/grill-with-docs`, `/to-spec`, `/to-tickets`). `/pipeline-setup` installs those for you; if you skip it, install them yourself for the full flow.

**Model-invoked:** you can type these, or the agent can reach for them when the moment fits.

- **[tutorial](./skills/tutorial/SKILL.md):** Plain-English coaching for the next step of the pipeline. Type `/tutorial`, and once a project has started coaching, it picks up again by itself after each stage.
- **[design-system](./skills/design-system/SKILL.md):** Turns screenshots, mockups, Figma links, or live websites into `docs/design.md` (for the agent) and `docs/design.html` (for you). A modified version of [BuilderOS](https://github.com/BuildGreatProducts/builder-os)'s skill, trimmed to fit this pipeline.

## License

[MIT](./LICENSE). `design-system` is adapted from BuilderOS's MIT-licensed skill; its original author credit is kept in the skill's frontmatter.
