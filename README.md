# Skills

Agent skills for setting up an AI coding pipeline and learning to use it. They're built for anyone, on any machine, starting from nothing.

## Install

Pick **one** of these. Installing both gives you every skill twice.

<details>
<summary><strong>Claude Code (recommended)</strong></summary>

Run these two commands inside a Claude Code session:

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

1. Run `/pipeline-setup` in your project. It checks your computer, installs what's missing, and writes `docs/Pipeline.md`.
2. Run `/tutorial` whenever you're not sure what to do next. It coaches you through the pipeline one step at a time.

## Skills

**User-invoked:** these only run when you type them.

- **[pipeline-setup](./skills/pipeline-setup/SKILL.md):** Checks this computer and project, installs everything the AI coding pipeline needs, and writes `docs/Pipeline.md`.

**Model-invoked:** you can type these, or the agent can reach for them when the moment fits.

- **[tutorial](./skills/tutorial/SKILL.md):** Plain-English coaching for the next step of the pipeline. Type `/tutorial`, and once a project has started coaching, it picks up again by itself after each stage.

## License

[MIT](./LICENSE)
