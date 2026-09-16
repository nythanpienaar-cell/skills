# Playout — Pipeline Layout

**The requirements document for the Pipeline Setup.**

Playout is the single source of truth for *what a working pipeline installation actually consists of*. It is not a manual, not a tutorial, and not written for a learner to read. It is written for the Pipeline Setup skill to execute against.

**Governing assumption: the user has set up nothing.** No accounts, no tools, no repo, no prior knowledge of what a terminal is. Every component below is specified as though it is absent, and every probe is written to prove presence rather than assume it. Where a step cannot be automated, it is marked `HUMAN` and given click-level instructions.

**Governing goal: maximum AFK.** Everything the setup skill can install, link, or configure itself, it does itself — see §0.4 for the exact line between AFK and `HUMAN`.

---

## 0. Scope and intent

### 0.1 The two-part split

| Part | Skill | Job | Reads |
|---|---|---|---|
| 1 | **Pipeline Setup** | Get a machine and a project into a state where every pipeline skill works. Detect, branch, install, configure, verify. | **Playout (this file)** |
| 2 | **Tutorial** (`/tutorial`) | Coach the user through the pipeline once it works. | `docs/Pipeline.md`, written by Part 1 in the format of [PIPELINE-FORMAT.md](PIPELINE-FORMAT.md) |

Part 2 is out of scope for this document. Playout describes Part 1 only. The handoff between them is `docs/Pipeline.md`: Part 1 writes it only once everything needed is installed, and Part 2 relies on it.

### 0.1a Run arguments

| Invocation | Runs |
|---|---|
| `/pipeline-setup` | Everything in L0–L8 that the project shape (Branch C) needs. **L9 is locked**: Branch E is never asked and nothing in §11 L9 is offered or installed. L9 is not probed either — **except** when an earlier automations run recorded a selection, which is re-probed read-only so `docs/Pipeline.md` stays truthful (§3 Branch E). |
| `/pipeline-setup automations` | Same as above, **plus** Branch E and L9 (Ralph, Docker, Claude Code token, Sandcastle). Anything missing from L0–L8 is completed first — automation sits on top of a finished core, never beside a broken one. |

A beginner's first run never meets Docker or unattended loops. They are unlocked deliberately, when the user is ready.

### 0.2 The core claim this document exists to make

> Installing the skills is roughly a third of the job.
>
> The Matt Pocock skills — `to-tickets`, `implement`, `code-review`, `triage`, `wayfinder`, `to-spec` — are *thin*. They do not carry their own storage, their own vocabulary, or their own credentials. They read per-repo configuration files, and they shell out to `gh`. A machine with every skill installed and none of that configuration is a machine where the pipeline **silently produces nothing usable**: `/to-tickets` writes tickets into a void, `/triage` applies labels that do not exist, `/wayfinder` cannot build a map, `/code-review` skips its Spec axis without saying why.

The setup skill's remit is therefore: **skills, plus everything the skills reach for.**

### 0.3 Design rules for the setup skill

1. **Detect before install.** Every component carries a `DETECT` probe. Nothing is installed without first failing its probe.
2. **Probe, don't assume.** Where a capability is version- or account-dependent, the setup skill makes a real call against the real target and branches on the result. It never assumes a feature exists because the docs say so.
3. **Never silently skip.** A component skipped — by branch, by user choice, or by failure — is recorded with a reason, and its downstream consequence (`DEGRADES`) is stated to the user at the time.
4. **Idempotent.** Re-running on a fully configured machine/project changes nothing and reports "already complete."
5. **Resumable.** State lives in `.pipeline/setup-state.md` (§13). Every completed component is written there before the next one starts. Re-running in the same project, or a new one, is always safe and always the answer to "I need something that wasn't installed" (§3 Branch C, *Not needed*).
6. **Least privilege.** Never request an auth scope broader than the component needs. Never write a secret into a file the setup skill has not first confirmed is gitignored.
7. **The setup skill runs commands; the user does what only the user can do.** See §0.4.
8. **One question at a time**, leading with the recommended answer so it can be accepted in a word.
9. **No jargon without a gloss.** First use of terminal, repo, commit, push, remote, scope, label, token, environment variable gets a half-sentence plain-language gloss. The user has never met these words.
10. **Never blame the user for a failed probe.** A missing tool is data for the install list, not a mistake to report.
11. **Prefer machine-readable output to human prose.** Parse `--json`, exit codes, and structured fields rather than matching English strings. This skill ships to other people, whose machines may be set to any language — a probe keyed on the words "Logged in to" fails on a non-English install of a tool that is working perfectly. Where a tool offers no structured output, key on the exit code and treat the text as a hint only.
12. **Third-party identifiers are volatile — fail loudly, never guess.** See §0.7.

### 0.4 What the setup skill can and cannot do itself — AFK vs HUMAN

This boundary shapes the whole skill and is easy to get wrong.

**AFK** — the setup skill performs it. This includes anything that only needs the user to **approve**: a Claude Code tool-permission prompt, or a click-only OS elevation dialog (Windows UAC "Yes"). Approving is not doing.

**`HUMAN` (HITL)** — only the user can do it: creating an account, signing in through a browser (OAuth), typing a password, answering a real-terminal interactive prompt, typing a slash command or a user-invoked skill, and anything involving money.

**The setup skill CAN**, directly: run shell commands (`node --version`, `gh label create`, `npm install`, `winget install`), read and write files in the project, call the GitHub API through `gh`, install plugins and MCP servers through the Claude Code CLI's shell subcommands (below), inspect its own session's skill listing.

**Shell equivalents of slash commands — AFK.** Current Claude Code CLIs expose these as ordinary shell commands. **Probe first** (`claude plugin --help`, `claude mcp --help`); if a subcommand is absent on the user's version, fall back to the slash command as a `HUMAN` step.

| Slash command (`HUMAN`) | Shell equivalent (AFK) |
|---|---|
| `/plugin marketplace add <src>` | `claude plugin marketplace add <src>` |
| `/plugin install <p>@<m>` | `claude plugin install <p>@<m> --scope user --yes` |
| `/skills` (to verify a plugin) | `claude plugin list --json` |
| `/mcp` (to list servers) | `claude mcp list` |
| adding an MCP server | `claude mcp add --scope user ...` |

**Installed is not loaded.** Plugins, skills, and MCP servers installed from the shell register in the *next* Claude Code session. Verify installation from the shell (AFK); when a later component needs the new skill or server *inside* the session, the user must start a new session and re-run `/pipeline-setup` — a `HUMAN` step, made painless by the state file (§13). Batch every install that needs a restart so the user restarts once, not per component.

**The setup skill CANNOT**, and must hand to the user:

| Cannot | Why | Handling |
|---|---|---|
| Type a slash command with no shell equivalent (`/reload-plugins`, MCP sign-in via `/mcp` → Authenticate) or invoke a user-invoked skill (`/setup-matt-pocock-skills`) | These are interpreted by the Claude Code front-end. A skill cannot issue them into its own session, and user-invoked skills are unreachable by the agent. | Give the exact command, say **which window** to type it in, wait, then verify the *effect* by another means |
| Answer an interactive prompt that reads from a real TTY (`gh auth login`'s menus, `npm init playwright`, `netlify init`, `supabase link`'s password prompt, `sandcastle init`) | The setup skill's shell is non-interactive | Hand to the user with the exact answers to give at each prompt |
| Enter an OS password (Homebrew install, Docker install) | No TTY, and the setup skill must never handle passwords | HUMAN step |
| Click in a browser (account signup, OAuth approval, Supabase project creation) | No browser control in scope | HUMAN step with click-level directions |
| See a token value | Secrets must not pass through the transcript | Instruct the user to paste it directly into the target file |

**The bootstrap paradox, stated plainly.** The setup skill is a Claude Code skill, so Claude Code must already exist for the setup skill to run at all. `CC-DESKTOP` / `CC-CLI` in §6 are therefore partly retrospective — the setup skill verifies and completes an install that has already partly happened, and its real job there is to establish *which surface the user is currently in* and whether the *other* surface also exists.

### 0.5 Preflight — what the setup skill says before it starts

A beginner should never be surprised by a password prompt or a bill. Before the first component, state:

- Roughly how long this takes, and that it is **safely stoppable at any point** — progress is saved and resuming picks up where it left off.
- That several steps open a **browser sign-in** (GitHub, and optionally Supabase and Netlify), and a couple may ask for the **computer's own password**.
- Which accounts will be created, and that **every one of them has a free tier that is sufficient** — with the single exception of Anthropic (§6), which needs a paid plan or API credit and is the only thing here that costs money.
- That the setup skill will make **small, reversible test writes** to the GitHub repo (a throwaway label, one or two throwaway issues) to prove permissions actually work, and will clean them up.
- That it will **only install what this project needs**, will say plainly what it left out and why, and that running `/pipeline-setup` again later adds anything that becomes needed.
- **Windows only:** that some installs pop up a **"Do you want to allow this app to make changes?"** box — clicking **Yes** is expected — and that installing from `winget` accepts each package's licence terms. **Ask once for consent to accept those terms before the first `winget install`.**
- **macOS and Linux only:** that a few steps must be run in their own Terminal because they ask for the computer's password, and that the password stays invisible while typed.

### 0.6 Terminal literacy — the one-time orientation

Delivered once, on first `HUMAN` step requiring a terminal, only in `full` mode.

- **Opening it.** Windows: Start key → type `PowerShell` → open *Windows PowerShell*. macOS: `Cmd+Space` → type `Terminal` → Enter. Linux: `Ctrl+Alt+T` on most desktops.
- **Pasting.** Windows PowerShell and macOS Terminal: `Ctrl+V` / `Cmd+V`. Many Linux terminals need `Ctrl+Shift+V`.
- **Password prompts show nothing while you type.** No dots, no stars. That is normal and not a freeze.
- **A command that ends with no output usually succeeded.** Silence is success in this world.
- **The Claude Code CLI is a different program from the Claude Code desktop app**, even though both are called Claude Code. Typing `claude` in a terminal starts the CLI. Some steps only work in one or the other, and each step below says which.

**Git Bash (Windows only)** — delivered the first time a wizard script must run (§0.8), in any mode, because the user has almost certainly never heard of it. Say it in roughly these words:

- *What it is:* "Git Bash is a second kind of terminal window. Windows' normal terminal (PowerShell) speaks one command language; Git Bash speaks the one most coding tools and scripts are written in. Think of it as the same kind of window, speaking a different language."
- *Where it came from:* "It was installed automatically along with Git earlier — you don't need to download anything."
- *Opening it:* Start key → type `Git Bash` → open it. A black window with a green line of text appears.
- *Pasting:* **right-click → Paste**, or `Shift+Insert`. `Ctrl+V` often does **not** work in Git Bash — the most common "nothing happened" moment.
- *Things to watch out for:*
  - PowerShell and Git Bash are **not interchangeable**. Every command the setup skill gives names the window it goes in. A Git Bash command pasted into PowerShell fails with confusing red text.
  - Windows also has a different program called plain `bash` (part of WSL). If Start search shows both, pick **Git Bash**.
  - Paths look different: `C:\Users\you\project` in PowerShell is `/c/Users/you/project` in Git Bash. The setup skill gives the exact line to paste, already converted.
  - To reach the project folder, paste the `cd` line the setup skill provides before running the script.

### 0.7 Volatile identifiers

Playout names identifiers owned by other people, on their release schedules, not ours. This skill ships to strangers and will be run months after it was written, so every one of these is a live staleness risk:

| Identifier | Owner | Used by |
|---|---|---|
| `mattpocock-skills@claude-plugins-official` | plugin marketplace | `SKILLS-MATTPOCOCK` |
| `nythanpienaar-cell/skills` (marketplace `nythan`), `nythan-skills@nythan` | the skill's author | `SKILLS-OWN` |
| `ChromeDevTools/chrome-devtools-mcp`, `chrome-devtools-mcp@chrome-devtools-plugins` | Chrome DevTools team | `MCP-CHROME-DEVTOOLS` |
| `https://mcp.supabase.com/mcp` | Supabase | `MCP-SUPABASE` |
| `figma@claude-plugins-official`, `https://mcp.figma.com/mcp` | Figma | `MCP-FIGMA` |
| `https://mcp.context7.com/mcp` | Upstash (Context7) | `MCP-CONTEXT7` |
| `BuildGreatProducts/builder-os/skills/launch-checklist` | BuilderOS | `SKILLS-BUILDEROS` |
| winget ids `OpenJS.NodeJS.LTS`, `Git.Git`, `GitHub.cli`, `Google.Chrome` | winget community repo | §5, §8.3 |
| Claude Code CLI subcommands (`claude plugin`, `claude mcp`) | Anthropic | §0.4 |
| `@ai-hero/sandcastle` | AI Hero | `SANDCASTLE-INIT` |
| Node's version floor, the `gh` default scope set, GitHub's sub-issue and dependency endpoints | upstream vendors | §5, §8.3, §8.6 |

**Rule.** When an install command fails with *not found* rather than a permission or network error, the identifier has probably moved. **Stop that component.** Do not retry variations, do not guess a replacement name, and do not silently substitute a similar-looking package — installing the wrong package from a public registry is a supply-chain risk, not a convenience. Record it as Blocked with the exact command and error, tell the user the name appears to have changed, and point them at the upstream source to confirm the current one. Then carry on with everything that does not depend on it.

This is also why §0.3 rule 2 exists: **the setup skill probes live capability rather than trusting this document.** Where Playout and the machine disagree, the machine is right and Playout is stale.

**Placeholder rule.** While any identifier in Playout still reads `<YOUR-SKILLS-REPO>` or another `<...>` placeholder, the component that uses it is **Blocked** with the reason "this copy of the setup skill has not been configured by its author" — never guessed, never searched for.

### 0.8 Guiding `HUMAN` steps

Two formats, chosen per step:

| Format | Use when | How |
|---|---|---|
| **Baby steps in chat** *(default)* | Accounts, browser sign-ins, clicking through a website, typing a slash command, restarting Claude Code | One action per message: where to go (which window or site), exactly what to click or type, and what they should see when it worked. Wait for the user to say done. Then **probe the effect** before the next action — never take "done" as proof when a probe exists. |
| **Wizard script** | A step that captures one or more values into a file or a GitHub secret — especially anything secret (`ENV-FILE`, `CC-TOKEN` into `.sandcastle/.env`) | Invoke the `wizard` skill from `SKILLS-MATTPOCOCK` to generate the script into `.pipeline/scripts/`. Secrets are typed into the script's hidden prompt, so they never pass through the chat transcript. Hand over with: which window (**Git Bash** on Windows — §0.6 — Terminal on macOS/Linux), the exact `cd` line, the exact run line. Probe the result files afterwards. |

A wizard script requires `SKILLS-MATTPOCOCK` (for the `wizard` skill) and, on Windows, `GIT-BASH`. Before those exist, only baby steps are available.

**Writing to a file other than `.env`.** The `wizard` template's `write_env` writes to `$ENV_FILE`, which defaults to `.env` and is set in the library section the template says never to edit. Leave the library alone and set the target on the run line instead: `ENV_FILE=.sandcastle/.env bash .pipeline/scripts/<script>.sh`. Hand the user that exact line. Afterwards, probe that the key exists in the target file without printing its value (e.g. `grep -c '^CLAUDE_CODE_OAUTH_TOKEN=' .sandcastle/.env`).

---

## 1. Scope model

| Scope | Meaning | Repeated per project? |
|---|---|---|
| `MACHINE` | Installed once per computer, ever. | No |
| `ACCOUNT` | An external account plus its auth on this machine. | No — but re-verified per project |
| `PROJECT` | Lives inside the repo; travels with the repo. | **Yes** |
| `REMOTE` | State on a third-party service tied to one repo (GitHub labels, Supabase project, Netlify site). | **Yes** |

The most common failure in practice is treating a `REMOTE` or `PROJECT` item as `MACHINE`. "I already set up GitHub" is true at `ACCOUNT` scope and false at `REMOTE` scope — *this* repo still has no labels.

---

## 2. Layer model and dependency order

```
L0  Foundations      Node, Git (+Git Bash), Git identity+defaults, pkg manager, Chrome (§5)
L1  Agent runtime    Anthropic account+billing, Claude Code desktop + CLI        (§6)
L2  Workspace        Sane project folder, git init, .gitignore, secret scan      (§7)
L3  GitHub chain     Account, gh, auth, scopes, credential helper, repo, Issues  (§8)
L4  Skill install    mattpocock-skills bundle, own skills, BuilderOS, dedupe     (§9)
L5  Repo capability  Labels, sub-issues probe, dependencies probe                (§8.5-8.6)
L6  Project config   docs/agents/*, CLAUDE.md block, CONTEXT.md, standards       (§10)
L7  Glue             Context7 MCP (always); Chrome DevTools, Figma, Supabase MCPs,
                     Playwright, Netlify CLI, Supabase CLI (by project shape)    (§11)
L8  Services         Supabase project + link, Netlify site + link, .env          (§11)
L9  Automation       Docker, Claude Code OAuth token, Sandcastle init
                     — LOCKED unless run as `/pipeline-setup automations`       (§11)
```

**Hard ordering constraints.** Violating these produces confusing failures, not clean errors.

- `L0` before everything — `gh`, `npm`, and the CLIs all need Node and a package manager.
- `L2` before `L3` — `gh repo create --source=.` needs a git repo with at least one commit.
- **`.gitignore` before any secret is written.** Non-negotiable; see `WS-GITIGNORE`.
- `L3` before `L5` — labels need a repo to live in.
- `L4` before `L6` — `setup-matt-pocock-skills` is what writes `docs/agents/*`, and it branches on whether `triage` is installed.
- `L5` before the first `/to-tickets` or `/triage` run — applying a label that does not exist fails the entire `gh issue edit` call, not just the label.
- `L6` before `L9` — Sandcastle's unattended agents read the same `docs/agents/issue-tracker.md`.

---

## 3. Branch points

Resolved **before** touching any component. Each branch prunes the component list.

### Branch A — Operating system
Detected, never asked.

| Value | Consequences |
|---|---|
| `windows` | Package manager = `winget`. Skip Homebrew. Terminal = PowerShell. Additional: long-path config, `core.autocrlf`, OneDrive-redirection check. |
| `macos` | Package manager = Homebrew → **a required L0 component**, including its post-install PATH `echo` lines. Terminal = Terminal.app. Watch for npm global `EACCES`. |
| `linux` | Distro package manager or Homebrew-on-Linux. `gh` install path differs. Docker Engine suffices instead of Docker Desktop. Paste is often `Ctrl+Shift+V`. Watch for npm global `EACCES`. |

### Branch B — Run mode
Settled **last in step 1**, after the interview and after the branch-gated probes it unlocks — the `full`/`project` split depends on L3, which is gated on Branch D and cannot be probed before that answer exists. Inputs: every step-1 probe result, plus the Branch C/D answers (a changed answer can turn `verify` into `repair`). Recording the mode before the interview writes a mode the answers may contradict. The mode labels the run; steps 2–4 are always driven by the checklist's contents, never by the label.

| Mode | Trigger | Runs |
|---|---|---|
| `full` | L0, L1 or L3 fails | Everything needed, L0 → L8 (L9 only when unlocked, §0.1a). Includes §0.6 orientation. |
| `project` | L0–L4 pass, no `docs/agents/` here | L2, L3 (repo only), L5–L8 (L9 only when unlocked) |
| `repair` | L0–L4 pass, `docs/agents/` exists, a later probe fails — or a Branch C answer changed to yes, or Branch D changed | Only failing or newly-needed components and their dependents |
| `verify` | Everything needed passes | No installs. Write or refresh `docs/Pipeline.md` (whiteboard: "if everything is there, skip 3–4, do 5"), then report. |

Branch C is re-asked in **every** mode, including `verify` — a project's needs change, and the interview is how a re-run discovers it.

### Branch C — Project shape
A short interview, once, near the top. Answers gate whole layers.

| Question | If yes | If no |
|---|---|---|
| C1 — Does this project have a user interface? | List the `design-system` stage; enable Playwright, Chrome DevTools MCP, and the Chrome browser requirement; ask C1a | Skip L7 browser tooling and C1a; leave the `design-system` stage out of `docs/Pipeline.md` (the skill still arrives with `SKILLS-OWN`) |
| C1a — *(only if C1 = yes)* Ask in these words: "Figma is a website designers use to draw what an app's screens will look like before it's built. Do you have designs in Figma, or plan to make some? If you've never heard of it, the answer is **no** — screenshots and websites you like work just as well." | Enable `MCP-FIGMA` | Skip `MCP-FIGMA` — design-system still works from screenshots and websites |
| C2 — Does it need to store data between visits? | Enable Supabase: `SUPABASE-CLI`, `MCP-SUPABASE`, L8 | Skip Supabase entirely |
| C3 — Does it need to be reachable on the public internet? | Enable Netlify (L8) and `launch-checklist` | Skip Netlify and `launch-checklist` |
| C4 — Is this a monorepo? *(auto-probe first: `pnpm-workspace.yaml`, `workspaces` in `package.json`, populated `packages/*/src`)* | `domain.md` = multi-context (`CONTEXT-MAP.md` plus per-context `CONTEXT.md`) | Single-context (root `CONTEXT.md` plus `docs/adr/`) |
| C5 — Will anyone other than you open issues here? | Consider flipping the *PRs as a request surface* flag; keep triage labels canonical | Leave the flag off (its default) |
| C6 — Should the GitHub repo be private or public? | See `GH-REPO`; **default private** | — |

> C4 defaults to **single-context**. Do not ask unless a monorepo signal was detected. C1–C3 should be asked in plain language ("will people look at this in a browser?"), never in jargon.

#### Not needed — the beginner's safety net

A beginner who answers "no" today does not know that tomorrow's project may need the thing they skipped. Every component switched off by a Branch C answer is recorded as **Not needed** — distinct from *Skipped* (user declined something needed) — and surfaced in three places:

1. **The closing report** (§12) — a plain-language "Not installed, because this project doesn't need it" list.
2. **`docs/Pipeline.md`** — the same list, permanently, under the install report (format: [PIPELINE-FORMAT.md](PIPELINE-FORMAT.md)). This is what lets the Part 2 coaching skill notice later work that needs a skipped tool.
3. **The re-run habit** — both of the above end with the same instruction: *run `/pipeline-setup` at the start of every new project, and again in this one if its needs change.*

Each Not-needed entry carries: the component in plain words, what it is for in one sentence, the question and answer that ruled it out, and that re-running `/pipeline-setup` adds it.

**Changed answers on re-run.** Compare every Branch C answer with the one recorded in the state file. *no → yes*: the newly-needed components join this run's checklist. *yes → no*: **uninstall nothing** — record the component as Not needed, note it is still installed, and move on.

**Sub-questions never reached.** A component gated by a sub-question that was never asked (C1a when C1 = no) is Not needed because of the **parent** question: report it with the parent's question and answer ("Left out because you answered *no* to 'Will people look at this on a screen?'").

**C6 changed on re-run.** *private → public*: `WS-SECRET-SCAN` must pass first; then confirm with the user in plain words that anyone on the internet will be able to read the code, and change visibility with `gh repo edit --visibility public` (with its current consequence-acknowledgement flag). *public → private*: change it and record — nothing else depends on it. Either change puts the run in `repair` mode.

### Branch D — Issue tracker
**The highest-consequence branch in this document.** All of L5 and half of L6 depend on it.

| Value | Chosen when | L5 work | CLI |
|---|---|---|---|
| `github` *(default)* | `git remote -v` points at github.com, or no preference stated | Full §8.5–8.6 provisioning | `gh` |
| `gitlab` | Remote points at gitlab.com or self-hosted GitLab | Labels via `glab`; **no sub-issue or dependency parity — §8.7** | `glab` |
| `local` | No remote, or explicitly solo/offline | Create `.scratch/`. No labels, no auth, no API probes. L3 becomes optional. | none |
| `other` | Jira, Linear, etc. | Setup cannot provision. Record a one-paragraph workflow description verbatim. Warn that `wayfinder` blocking and `triage` labelling become manual. | n/a |

> **Setup rule:** propose the value implied by `git remote -v` and let the user accept in one word. For a true beginner with no remote yet, the correct default is `github` — propose creating the repo rather than falling through to `local`, since `local` quietly forfeits `wayfinder` and the entire automation path.

> **"Not right now" is `local`.** A user who declines to create a GitHub account is choosing `local` for this run: state the §8.7 cost, record it, and carry on. It is not a skip of a needed item.

**Changed on re-run.** Branch D is re-offered on every run only when the recorded value is `local` or `other` — "Last time you kept issues on this computer. Keep it that way, or set up GitHub now?" A change to `github` adds the §8 chain and L5 to this run's checklist. A change *away* from `github` deletes nothing remote; record the new value and state the §8.7 cost.

### Branch E — Automation appetite
**Locked behind `/pipeline-setup automations` (§0.1a).** The argument, never resume status, decides whether Branch E is asked. On a plain `/pipeline-setup` run, Branch E is never asked and nothing in L9 is offered or installed. The recorded value is **preserved**: if it is unset or `locked`, record `locked` and leave L9 unprobed; if it is already `none`, `ralph`, or `sandcastle` from an earlier automations run, keep it and re-probe that L9 selection read-only, so `docs/Pipeline.md` stays truthful. An L9 failure found on a plain run does not hold the §14.1 loop — report it, show it in Pipeline.md as needing repair, and point at `/pipeline-setup automations`. L9 is not listed as Not needed either — it is not part of a beginner's pipeline, so it stays out of their report entirely. When unlocked, asked last, after everything else works.

| Value | L9 components |
|---|---|
| `none` | Skip L9. |
| `ralph` | Nothing to install. Verify only that L0–L6 are complete, since the loop's entire safety model is that progress lives in commits and tickets rather than in conversation memory. |
| `sandcastle` | Docker, `claude setup-token`, `npx @ai-hero/sandcastle init`, token into `.sandcastle/.env`, gitignore check. |

### Branch F — Pre-existing conflicting install
Probed during L4.

| Condition | Action |
|---|---|
| Skills appear twice (`mattpocock-skills:to-tickets` **and** a bare `to-tickets`) | An older manual copy exists, typically under `~/.claude/skills/` or a symlink farm such as `~/.agents/skills/`. Both work; the ambiguity is the problem. Offer to identify and remove the older copy. **Never delete without explicit confirmation** — it may be a deliberate local fork. |
| A skill directory exists but is a broken symlink | Repair or remove, and report. |

### Branch G — Environment hazards
Probed silently during L0/L2. Each is a real, common beginner trap that produces a baffling failure much later.

| Hazard | Probe | Response |
|---|---|---|
| Project folder inside a cloud-synced directory (OneDrive, iCloud Drive, Dropbox, Google Drive) | Compare the project path against known sync roots. On Windows, **Desktop and Documents are frequently redirected into OneDrive** — resolve the real path, do not trust the visible one. | Warn: sync engines fight with `node_modules` and `.git`, causing file-lock errors, corrupted checkouts, and enormous sync churn. Recommend relocating to a non-synced path. Do not move anything without consent. |
| Project folder is a system or home root (`C:\`, `~`, `/`, Downloads) | Path comparison | Refuse to `git init` there. Offer to create a proper subfolder. |
| Path contains spaces or non-ASCII characters | Path inspection | Low severity. Note it; some tooling quotes paths badly. Do not block. |
| Windows long-path limit | `git config --get core.longpaths` | If unset on Windows, offer `git config --global core.longpaths true`. Deep `node_modules` trees exceed 260 characters routinely. |
| No write permission in the folder | Attempt a temp-file write | Stop. Ask for a different folder. |
| Offline / proxied network | A single reachability check before the first download | Report plainly; nearly every component needs the network. |
| Managed or corporate machine | `winget` disabled, admin rights refused, or a policy error on install | Do not fight it. Record as Blocked with the exact error. From the first policy failure on, **treat every remaining package-manager install as presumptively Blocked** — do not attempt each one and collect a failure per item. Give the user **one** IT request that lists every remaining component needing an install on this machine, so IT is asked once, not once per run. Re-probe them next pass as usual. |

---

## 4. Component schema

| Field | Meaning |
|---|---|
| `ID` | Stable identifier, used in the state file |
| `SCOPE` | MACHINE / ACCOUNT / PROJECT / REMOTE |
| `NEEDS` | Component IDs that must complete first |
| `GATE` | Branch conditions under which this component applies at all |
| `DETECT` | Read-only probe. Never installs, never mutates. |
| `INSTALL` | The action, OS-branched where relevant |
| `HUMAN` | The part only the user can do |
| `VERIFY` | The check proving it worked, where that differs from DETECT |
| `DEGRADES` | **What breaks downstream if skipped.** Stated to the user at skip time. |

**Universal rule for every `VERIFY` in this document:** if a verify fails immediately after a successful install, **restart the shell and retry once before declaring failure.** A newly installed tool is very often absent from the current shell's `PATH` and present in the next one. This single rule prevents the most common false failure in the entire setup run.

---

## 5. L0 — Machine foundations

Nothing here is pipeline-specific. All of it is absent on a fresh machine.

**`OS-PKGMGR`** — a way to install command-line software
- `SCOPE` MACHINE · `GATE` always
- `DETECT` — windows: `winget --version` · macos: `brew --version` · linux: `apt`/`dnf`/`pacman` present
- `INSTALL` —
  - **windows**: `winget` ships with Windows 10/11 via App Installer. If missing, install *App Installer* from the Microsoft Store. `HUMAN`.
  - **macos**: Homebrew — **`HUMAN`**, because it asks for the Mac password and the setup skill's shell has no terminal to type it into. Two separate baby steps:
    1. In **Terminal** (not the chat box), paste `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`, press Enter, and type the Mac password when asked (nothing appears while typing). Wait for it to finish.
    2. **Its own step:** scroll to the "Next steps" section it printed and run each of the two or three lines listed there, one at a time. These put Homebrew on the PATH. Skipping them leaves Homebrew installed but unfindable in every future terminal, which presents as "the install silently did nothing."
  - **linux**: already present.
- `VERIFY` — close and reopen the terminal, then re-run the detect. **macOS:** if `brew --version` still fails but `/opt/homebrew/bin/brew` (Apple Silicon) or `/usr/local/bin/brew` (Intel) exists, step 2's PATH lines were missed — send the user back to that step, not to reinstall.
- `DEGRADES` — `gh` cannot be installed on macOS; several installs have no path

**`NODE`** — the JavaScript runtime
- `SCOPE` MACHINE · `GATE` always
- `DETECT` — `node --version` prints a version **and that version meets the current floor**
- **Version floor.** Claude Code, Playwright, and the Supabase/Netlify CLIs all require a modern Node. Treat **Node 18 as an absolute floor and current LTS as the target**, and check the tools' own published requirement rather than hardcoding a number that will age. **A stale Node (14, 16) is worse than no Node** — it passes a naive "does `node --version` print something" probe and then fails obscurely deep inside an install. The probe must compare, not merely observe.
- `INSTALL` — **AFK**: windows `winget install --id OpenJS.NodeJS.LTS -e --accept-package-agreements --accept-source-agreements` (UAC click) · macos `brew install node` · linux distro package or a version manager. `HUMAN` fallback only when the package manager is unavailable: the LTS installer from nodejs.org with defaults. If an old Node exists, upgrading in place is preferable to a second parallel install.
- `VERIFY` — `node --version` and `npm --version` both print, in a fresh shell
- `DEGRADES` — nothing in L4, L7, L8, or L9 can install

**`NPM-GLOBAL-WRITABLE`** — global installs work without `sudo`
- `SCOPE` MACHINE · `NEEDS` NODE · `GATE` macos/linux, and only when a global install is planned (`NETLIFY-CLI`)
- On macOS and Linux, a system-installed Node routinely makes `npm install -g` fail with `EACCES`. A beginner's instinct is `sudo npm install -g`, which creates root-owned files that break every later install.
- `DETECT` — `npm config get prefix`, and test writability of that directory
- `INSTALL` — repoint the prefix to a user-owned directory, or install Node through a version manager. **Never instruct the user to run `sudo npm`.**
- `DEGRADES` — `NETLIFY-CLI` fails, or "succeeds" into a root-owned state that breaks later

**`GIT`** — version control
- `SCOPE` MACHINE · `GATE` always
- `DETECT` — `git --version` prints a version. **macOS: run `xcode-select -p` first.** If it fails, record Git as missing and do **not** run `git --version` — on a Mac without Command Line Tools, `git` is a stub that pops up a system installer dialog, turning a read-only probe into an unannounced install.
- `INSTALL` — **AFK**: windows `winget install --id Git.Git -e --accept-package-agreements --accept-source-agreements` (UAC click; includes Git Bash) · macos `brew install git` (`xcode-select --install` opens a system dialog — `HUMAN` click-through) · linux distro package
- `DEGRADES` — no commits, no GitHub, no pipeline

**`GIT-BASH`** — a bash terminal for wizard scripts
- `SCOPE` MACHINE · `NEEDS` GIT · `GATE` windows
- Ships inside Git for Windows; nothing separate to install. macOS and Linux already have a bash-compatible shell.
- `DETECT` — resolve Git's install root from `git --exec-path` and confirm `bin/bash.exe` exists there. **Do not probe with a bare `bash`**: on Windows that can resolve to WSL's `bash.exe` in `System32`, a different program.
- `INSTALL` — none of its own; if absent, `GIT` was installed without it — reinstall `GIT` via winget.
- `HUMAN` — the §0.6 Git Bash orientation, the first time a wizard script is handed over
- `DEGRADES` — wizard scripts (§0.8) cannot run; value-capture steps fall back to baby steps with the user pasting values into files by hand

**`GIT-IDENTITY`** — Git knows who is committing
- `SCOPE` MACHINE · `NEEDS` GIT
- `DETECT` — `git config --global user.name` and `user.email` both non-empty
- `INSTALL` — set both
- **Critical beginner trap — the email must match GitHub.** If the user later enables GitHub's *Keep my email address private* setting, pushes from a mismatched address are **rejected outright** with an opaque error. Two safe options: use an email verified on the GitHub account, or use the account's `ID+username@users.noreply.github.com` address. The setup skill should resolve the correct value after `GH-AUTH` and reconcile it, rather than guessing here.
- **Ordering note.** Identity is not enforced at install time — it fails silently at the *first commit*, which under automation may be hours later and inside an unattended loop.
- `DEGRADES` — every commit fails, or commits succeed but are not attributed to the user's GitHub account

**`GIT-DEFAULTS`** — sane global Git configuration
- `SCOPE` MACHINE · `NEEDS` GIT · `GATE` always, offered as a batch

| Setting | Why | Applies |
|---|---|---|
| `init.defaultBranch main` | GitHub, Netlify, and most tooling assume `main`. A repo initialised as `master` produces mismatched-branch confusion at the first push. | all |
| `core.autocrlf true` | Prevents whole-file line-ending churn in diffs, which makes `code-review` useless by burying the real change | windows |
| `core.longpaths true` | `node_modules` routinely exceeds the 260-character path limit | windows |
| `pull.rebase false` (or an explicit choice) | Git refuses to pull without a configured strategy and emits a wall of text a beginner cannot parse | all |

- `DEGRADES` — a series of confusing, unrelated-looking errors at unpredictable moments

**`CHROME`** — a Chrome-family browser
- `SCOPE` MACHINE · `GATE` C1=yes
- The Chrome DevTools MCP drives Chrome specifically. A user with only Safari or Edge has nothing for it to attach to.
- `DETECT` — look for a Chrome (or Chromium) installation in the OS's standard location
- `INSTALL` — **AFK**: windows `winget install --id Google.Chrome -e --accept-package-agreements --accept-source-agreements` · macos `brew install --cask google-chrome` · linux: `HUMAN`, from google.com/chrome (distro packaging varies too much to automate safely)
- `DEGRADES` — `MCP-CHROME-DEVTOOLS` installs but cannot open a browser; UI bugs are diagnosed by guessing from source

---

## 6. L1 — Anthropic account and the agent runtime

**`ANTHROPIC-ACCOUNT`** — an account with a plan that permits Claude Code
- `SCOPE` ACCOUNT · `GATE` always
- **This is the only component in Playout that costs money**, and it is the one most likely to be assumed rather than checked. Claude Code requires either a subscription that includes it or API billing with credit available. A user on neither gets a sign-in that appears to work and then a refusal at first use.
- `DETECT` — indirect: the setup skill is running inside Claude Code, so *some* valid access exists. What cannot be assumed is that it covers `claude setup-token` (see `CC-TOKEN`) or that credit will not run out mid-run.
- `HUMAN` — sign in at claude.ai; confirm the plan or API credit
- `DEGRADES` — everything stops, including the setup skill itself

**`CC-DESKTOP`** — the Claude Code desktop app
- `SCOPE` MACHINE · `NEEDS` ANTHROPIC-ACCOUNT
- `DETECT` — if the setup skill is running inside the desktop app, present by definition. Otherwise check the OS application list.
- `INSTALL` — `HUMAN`, from claude.ai/download. Windows: run the installer with defaults. macOS: drag to Applications.
- `VERIFY` — the app opens, shows a sidebar, and the **Code** tab has a chat box
- `DEGRADES` — the day-to-day surface for the pipeline is missing

**`CC-CLI`** — the Claude Code command-line interface
- `SCOPE` MACHINE · `NEEDS` ANTHROPIC-ACCOUNT
- **The desktop app and the CLI are separate programs sharing one name.** The CLI is also what gives the setup skill its AFK install path: `claude plugin install` and `claude mcp add` (§0.4) are subcommands of the CLI binary, and work from the setup skill's shell even when the user is in the desktop app. `/plugin` slash commands, by contrast, cannot be typed into the desktop chat box — the most common reason a beginner's manual skill install "does nothing."
- `DETECT` — `claude --version` in a shell
- `INSTALL` — per current Claude Code install instructions for the OS
- `VERIFY` — typing `claude` in a terminal opens an interactive session
- `DEGRADES` — no plugin or MCP server can be installed, AFK or otherwise

**`CC-SURFACE`** — which surface is the setup skill currently in?
- `SCOPE` — n/a, a determination rather than an install · `NEEDS` CC-DESKTOP, CC-CLI
- Not optional bookkeeping: every `HUMAN` instruction involving a slash command must name the correct window, and that name depends on where the user is now. Determine it, record it, and phrase instructions accordingly ("switch to the Terminal window running `claude`" vs "you're already there").
- `DEGRADES` — the user types `/plugin install` into the wrong window, sees nothing happen, and concludes the setup skill is broken

---

## 7. L2 — Workspace and repo hygiene

Everything here precedes GitHub and exists to prevent damage that is expensive to undo.

**`WS-FOLDER`** — a sane project directory
- `SCOPE` PROJECT · `GATE` always
- `DETECT` — apply every **Branch G** hazard probe to the current directory
- `INSTALL` — where the folder is unsuitable, propose a specific alternative path and offer to create it. **Never relocate an existing folder without explicit consent.**
- `DEGRADES` — cloud-sync corruption, permission failures, or a `git init` over the user's entire home directory

**`WS-GITREPO`** — a local git repository with at least one commit
- `SCOPE` PROJECT · `NEEDS` GIT, GIT-IDENTITY, WS-FOLDER
- `DETECT` — `git rev-parse --is-inside-work-tree` is true, and `git rev-list -n1 --all` returns something
- **A repo with zero commits has no default branch.** `gh repo create --source=. --push`, Netlify's repo linking, and several `gh` calls all behave strangely against it. Make the first commit part of this component, not a later afterthought.
- `INSTALL` — `git init`, then `WS-GITIGNORE`, then an initial commit
- `DEGRADES` — `GH-REPO` cannot complete

**`WS-GITIGNORE`** — a `.gitignore` exists and covers the dangerous paths
- `SCOPE` PROJECT · `NEEDS` WS-GITREPO · `GATE` always
- **Runs before any secret, key, or token is written anywhere in the project.** This ordering is the single most important safety constraint in Playout. A secret committed to a repo — especially one about to be made public — is not fixed by deleting the file; it persists in history and must be treated as compromised and rotated.
- `DETECT` — `.gitignore` exists and contains, at minimum: `node_modules/`, `.env`, `.env.*`, `.sandcastle/.env`, build output (`dist/`, `build/`, `.next/`), OS cruft (`.DS_Store`, `Thumbs.db`)
- `INSTALL` — create or extend it. Extending must be additive; never rewrite a user's existing file.
- `DEGRADES` — secrets leak; `node_modules` is committed, bloating the repo and making every diff unreviewable

**`WS-SECRET-SCAN`** — nothing sensitive is already tracked
- `SCOPE` PROJECT · `NEEDS` WS-GITREPO · `GATE` always, and **mandatory before `GH-REPO` creates a public repo**
- `DETECT` — list tracked files (`git ls-files`) for `.env`, `*.pem`, `*.key`, credential JSON, and `.sandcastle/.env`; scan tracked content for obvious key shapes (`sk-`, `ghp_`, `gho_`, `service_role`, long base64 blobs in config files)
- On a hit: **stop and tell the user plainly** — the value must be treated as compromised and rotated at its source, and removal from history is a separate, deliberate operation. Do not silently `git rm --cached` and proceed as though it is fixed.
- `DEGRADES` — a published repo containing live credentials

**`WS-README`** — a minimal README
- `SCOPE` PROJECT · `GATE` optional, low priority
- Gives `gh repo create` something to show and the first commit something to contain. Skip without ceremony if the user has content already.

---

## 8. L3/L5 — GitHub, the requirements chain

This section is the reason Playout exists. Everything here is invisible in a "did the skills install?" check and fatal in practice.

### 8.1 Why GitHub is not optional under Branch D = `github`

| Skill | What it does to GitHub | Fails how, without setup |
|---|---|---|
| `to-tickets` | Creates one issue per vertical slice, in dependency order, applying `ready-for-agent` | Creation fails outright, or issues are created **unlabelled** and no agent can find them |
| `triage` | Reads open issues, applies one of five canonical labels | `gh issue edit --add-label` errors on a non-existent label; the issue is left untriaged |
| `wayfinder` | Creates a map issue, child sub-issues, blocking edges, claims by assignee | Cannot create the map (no `wayfinder:map` label); falls back to task-list text, losing the live blocked/unblocked gate |
| `code-review` | Reads the originating issue for its Spec axis | Silently drops to a Standards-only review and reports "no spec available" |
| `implement` | Commits and pushes the work | Push rejected, or commits stack up locally unnoticed |

### 8.2 The chain, in order

```
GH-ACCOUNT → GH-CLI → GH-AUTH → GH-ACCOUNT-SELECT → GH-SCOPES → GH-CREDENTIAL-HELPER
                                                                          ↓
                                              GH-EMAIL-MATCH → GH-REPO → GH-ISSUES-ON
                                                                          ↓
                                        GH-LABELS-TRIAGE + GH-LABELS-WAYFINDER
                                                                          ↓
                                        GH-SUBISSUES-PROBE + GH-DEPENDENCIES-PROBE
```

### 8.3 Account, CLI, and authentication

**`GH-ACCOUNT`** — a GitHub account exists
- `SCOPE` ACCOUNT · `GATE` D=github
- `DETECT` — deferred; only provable via `GH-AUTH`
- `HUMAN` — sign up free at github.com, note the username, and **verify the email address GitHub sends** (an unverified account cannot push)
- `DEGRADES` — nothing downstream works

**`GH-CLI`** — the `gh` command-line tool
- `SCOPE` MACHINE · `NEEDS` OS-PKGMGR · `GATE` D=github
- `DETECT` — `gh --version` prints a version
- `INSTALL` — **AFK**: windows `winget install --id GitHub.cli -e --accept-package-agreements --accept-source-agreements` · macos `brew install gh` · linux distro package or Homebrew
- `DEGRADES` — every GitHub operation in every skill

**`GH-AUTH`** — this machine is signed in
- `SCOPE` ACCOUNT · `NEEDS` GH-CLI, GH-ACCOUNT · `GATE` D=github
- `DETECT` — `gh auth status` **exits zero** for github.com. Key on the exit code, not on the words in the output (§0.3 rule 11); use `gh api user --jq .login` to get the account name itself rather than scraping it from prose.
- `INSTALL` — `gh auth login`. **Interactive — hand to the user with the exact answers:** GitHub.com → **HTTPS** → yes to authenticating Git with GitHub credentials → login with a web browser → copy the one-time code → approve in the browser.
- **Choose HTTPS, not SSH.** SSH requires generating and registering a key pair, which is a beginner cliff with no benefit here.
- `HUMAN` — the browser approval and the one-time code
- `DEGRADES` — as `GH-CLI`

**`GH-ACCOUNT-SELECT`** — the right account is active
- `SCOPE` ACCOUNT · `NEEDS` GH-AUTH · `GATE` D=github, and only when more than one account is present
- `gh` supports several accounts at once, and `gh auth status` marks one **Active account: true**. A user with personal and work accounts can be authenticated as the wrong one and get 404s on their own repo — which reads as "the repo does not exist" rather than "you are the wrong person."
- `DETECT` — count accounts in `gh auth status`. If more than one: pass when the active account matches the selection recorded in the state file; otherwise **ask the user** which account this project belongs to (`HUMAN` question, leading with the active account) — never pick silently — and record the answer, so rechecks and later runs don't ask again
- `INSTALL` — `gh auth switch`
- `DEGRADES` — unexplained 404s on a repo that plainly exists

**`GH-SCOPES`** — the token can actually do what the skills need
- `SCOPE` ACCOUNT · `NEEDS` GH-AUTH · `GATE` D=github
- **The component most commonly missing on a machine whose owner believes GitHub is "set up."** A token can be valid, print a cheerful `gh auth status`, and still be unable to create a label.
- **Reference baseline.** The `gh auth login` browser flow currently grants `gist`, `read:org`, `repo`, `workflow` — which covers everything the pipeline needs. A token *narrower* than this baseline usually means a hand-made PAT rather than the standard flow, and is the case to watch for.

| Capability needed | Used by | Classic OAuth scope | Fine-grained PAT permission |
|---|---|---|---|
| Read/write issues, comments, labels | `to-tickets`, `triage`, `wayfinder`, `code-review` | `repo` | Issues: read & write |
| Read/write repo contents, push | `implement` | `repo` | Contents: read & write |
| Read repo metadata | all | `repo` | Metadata: read |
| Create a repo | `GH-REPO` | `repo` | *(fine-grained PATs cannot create repos — needs the OAuth flow)* |
| Read org membership, for org-owned repos | `GH-REPO` | `read:org` | Organization permissions as required |
| Touch workflow files, if the project gains CI | `implement` | `workflow` | Workflows: read & write |

- `DETECT` — `gh auth status` prints the token's scope list; confirm `repo` is present
- `INSTALL` — `gh auth refresh -h github.com -s repo,read:org,workflow`
- `VERIFY` — **do not trust the scope list alone.** Perform one real write against the target repo and roll it back: create a throwaway label, read it back, delete it. This catches org policy restrictions, SAML-unauthorised tokens, and repos where the caller has read-only access — none of which appear in the scope list.
- `DEGRADES` — skills fail mid-run with opaque `HTTP 403` errors, typically after writing half a plan

**`GH-CREDENTIAL-HELPER`** — `git push` can authenticate
- `SCOPE` MACHINE · `NEEDS` GH-AUTH · `GATE` D=github
- **A distinct failure from `GH-AUTH`, and a severe one.** `gh` being authenticated does not by itself let plain `git push` authenticate. GitHub no longer accepts account passwords over HTTPS, so without a credential helper the first push prompts for a password, rejects whatever is typed, and offers no useful explanation. Under automation it simply hangs.
- `DETECT` — **`git config credential.helper`, with no scope flag.** Checking `--global` alone gives a false negative: Git for Windows sets `credential.helper=manager` in the **system** config, so `--global` returns nothing while the helper is in fact configured and working. A setup skill checking only `--global` will "fix" something that is not broken.
- `INSTALL` — `gh auth setup-git`
- `VERIFY` — the real proof is a successful push, which happens at `GH-REPO`. Treat a push failure there as this component, not that one.
- `DEGRADES` — every push fails; `implement`'s commit step and all automation break

**`GH-EMAIL-MATCH`** — the commit email is acceptable to GitHub
- `SCOPE` MACHINE · `NEEDS` GH-AUTH, GIT-IDENTITY · `GATE` D=github
- `DETECT` — compare `git config --global user.email` against the emails on the authenticated account
- **Two failure shapes.** With *Keep my email address private* enabled, a mismatched address causes an **outright push rejection**. Without it, the push succeeds but commits are not linked to the user's GitHub profile — invisible now, confusing forever.
- `INSTALL` — set `user.email` to a verified account email, or to `ID+username@users.noreply.github.com`
- `DEGRADES` — pushes rejected, or a permanently unattributed commit history

### 8.4 Repository

**`GH-REPO`** — this project is a real repo with a real remote
- `SCOPE` REMOTE · `NEEDS` GH-AUTH, GH-ACCOUNT-SELECT (when gated in), GH-SCOPES, GH-CREDENTIAL-HELPER, WS-GITREPO, WS-SECRET-SCAN · `GATE` D=github
- `DETECT` — inside a work tree, `git remote -v` shows a github.com origin, and `gh repo view` succeeds
- **Visibility (Branch C6).** Ask, do not assume. Default **private**: private repos are free and unlimited, and a beginner's first project is the likeliest place for an accidentally committed key. Public can be chosen later; a leak cannot be un-leaked. **Never create a public repo without `WS-SECRET-SCAN` passing first.**
- `INSTALL`, branched:

| Condition | Action |
|---|---|
| Not a git repo | Handled upstream by `WS-GITREPO`, then `gh repo create --source=. --private --push` |
| Git repo, no remote | `gh repo create --source=. --private --push` |
| Remote exists, points elsewhere (GitLab, Bitbucket) | Stop. Re-run Branch D — this is not a GitHub project. |
| Remote points at GitHub but `gh repo view` 404s | Either the token lacks access or the wrong account is active. Check `GH-ACCOUNT-SELECT` **before** concluding the repo is gone. Never auto-create a replacement. |
| Repo name collides with an existing one | Ask for a different name; do not append a number silently |

- `VERIFY` — `gh repo view` resolves **and** an actual `git push` succeeds. The push is the real test of `GH-CREDENTIAL-HELPER` and `GH-EMAIL-MATCH` together.
- `DEGRADES` — nothing at REMOTE scope can be provisioned

**`GH-PERMISSION-TIER`** — what the user is allowed to do *in this repo*
- `SCOPE` REMOTE · `NEEDS` GH-REPO · `GATE` D=github
- **Do not assume the user owns the repo.** A repo the setup skill creates is owned by the user; a repo they were invited to is not. GitHub grades access, and the pipeline needs different tiers for different components:

| Tier | Can | Cannot |
|---|---|---|
| `read` | nothing the pipeline needs | everything below |
| `write` (push) | create labels, create/edit/close issues, push commits | change repo settings |
| `admin` | all of the above, plus enable Issues and edit repo settings | — |

- `DETECT` — `gh repo view --json viewerPermission`
- **Branching.** At `write`, everything except `GH-ISSUES-ON` works — proceed, and if Issues are already on, nothing is lost. At `read`, stop the entire GitHub branch: nothing at REMOTE scope can be provisioned, and the honest options are to ask the repo's owner for write access, or to fork it and work in the fork. Say that plainly rather than failing component by component.
- **Why this exists.** The author of this skill owns the repos they tested it on. Someone using it inside an organisation, a team, or a repo they were added to may hold neither `admin` nor `write`, and every downstream failure would otherwise present as a mystery 403.
- `DEGRADES` — a cascade of 403s with no single clear cause

**`GH-ISSUES-ON`** — the Issues tab is actually enabled
- `SCOPE` REMOTE · `NEEDS` GH-REPO, GH-PERMISSION-TIER · `GATE` D=github
- Issues can be disabled per repo, and are off by default on some forks and templates. Every ticket-producing skill assumes they are on.
- `DETECT` — `gh repo view --json hasIssuesEnabled`
- `INSTALL` — `gh repo edit --enable-issues`. **Requires `admin`.** At `write` tier this call fails; if Issues are already enabled the component still passes, and if they are not, the user must ask an owner to enable them.
- `DEGRADES` — `to-tickets` and `wayfinder` produce nothing at all

### 8.5 Label provisioning

Labels are **REMOTE scope and per-repo**. A user with a decade of GitHub history has none of these in a repo created this morning. Creating them is cheap, idempotent, and the highest-value thing the setup skill does.

**`GH-LABELS-TRIAGE`** — the five canonical triage roles
- `SCOPE` REMOTE · `NEEDS` GH-ISSUES-ON, GH-SCOPES · `GATE` D=github **and** `triage` installed

| Label | Meaning |
|---|---|
| `needs-triage` | Maintainer needs to evaluate this issue |
| `needs-info` | Waiting on the reporter for more information |
| `ready-for-agent` | Fully specified, ready for an unattended agent |
| `ready-for-human` | Requires human implementation |
| `wontfix` | Will not be actioned |

- `DETECT` — `gh label list --json name`, compared against the five
- `INSTALL` — `gh label create <name> --description "..." --color <hex>` for each missing one; `--force` keeps re-runs idempotent
- **Branch — the repo already uses different names.** If the repo has e.g. `bug:triage` or `status/blocked`, do **not** create duplicates. Ask the user to map each canonical role onto an existing label and record it in the right-hand column of `docs/agents/triage-labels.md`. The skills speak canonical roles; that file is the translation layer.
- **Note** — a new GitHub repo ships with default labels (`bug`, `enhancement`, `documentation`, …). None of them are these. Their presence is not evidence of setup.
- `DEGRADES` — `/triage` cannot label anything; `/to-tickets` cannot mark tickets agent-ready, so Ralph Loop and Sandcastle have no queue to read

**`GH-LABELS-WAYFINDER`** — the wayfinding vocabulary
- `SCOPE` REMOTE · `NEEDS` GH-ISSUES-ON, GH-SCOPES · `GATE` D=github **and** `wayfinder` installed

| Label | Applied to |
|---|---|
| `wayfinder:map` | The single map issue holding Notes / Decisions-so-far / Fog |
| `wayfinder:research` | A child ticket resolved by reading |
| `wayfinder:prototype` | A child ticket resolved by a throwaway build |
| `wayfinder:grilling` | A child ticket resolved by an interview |
| `wayfinder:task` | A child ticket that is a blocking chore |

- Same detect/install/idempotency pattern as the triage set.
- `DEGRADES` — `/wayfinder` cannot create or find its map; the whole fog-resolution path is unavailable

### 8.6 Capability probes — features that may or may not exist

**Not installs.** Questions the setup skill asks GitHub about this specific repo, whose answers change how `wayfinder` and `to-tickets` must behave. Answers are written into `docs/agents/issue-tracker.md` so the skills do not re-discover them at runtime.

**`GH-SUBISSUES-PROBE`** — are native sub-issues available here?
- `SCOPE` REMOTE · `NEEDS` GH-REPO · `GATE` D=github
- `DETECT` — call the sub-issues REST endpoint for a throwaway issue; observe success, 404, or 403. Roll back anything created.
- **Outcome A — available:** record `sub-issues: native`. `wayfinder` links children to the map through the real parent/child relationship.
- **Outcome B — unavailable:** record `sub-issues: task-list fallback`. Children are tracked as a markdown task list in the map body, each child body opening with `Part of #<map>`.
- `DEGRADES` — without a recorded answer, `wayfinder` guesses, and a failed guess costs a half-built map

**`GH-DEPENDENCIES-PROBE`** — are native issue dependencies available here?
- `SCOPE` REMOTE · `NEEDS` GH-REPO · `GATE` D=github
- `DETECT` — add and then remove a `blocked_by` edge between two throwaway issues. The endpoint takes the blocker's **numeric database id** (`gh api repos/<owner>/<repo>/issues/<n> --jq .id`), *not* the `#number` and *not* the `node_id` — passing the wrong identifier makes the setup skill misreport the capability as absent.
- **Outcome A — available:** record `blocking: native dependencies`. The frontier query uses `issue_dependencies_summary.blocked_by > 0` as a live, UI-visible gate.
- **Outcome B — unavailable:** record `blocking: Blocked by line`. Each child body carries `Blocked by: #<n>, #<n>` at the top; a ticket is unblocked when every listed blocker is closed.
- `DEGRADES` — `to-tickets` cannot express dependency order reliably, so an unattended agent can pick up a ticket whose prerequisite does not exist yet

> **Cleanup obligation.** Both probes create throwaway objects. The setup skill must delete or close them, prefer closing where deletion is unavailable, and **report explicitly if cleanup failed** rather than leaving debris in a real repo. Announce the probes before running them — a beginner watching their brand-new repo sprout mystery issues will reasonably think something has gone wrong.

### 8.7 Non-GitHub tracker parity

| Branch D | Triage labels | Sub-issues | Dependencies |
|---|---|---|---|
| `github` | Provisioned by setup | Probed | Probed |
| `gitlab` | Via `glab label create` | No direct equivalent — task list in the map | GitLab blocking relationship if the tier allows, else a `Blocked by` line |
| `local` | Not applicable — status is a field in the markdown file under `.scratch/<feature>/` | Directory nesting | `Blocked by` line |
| `other` | Manual — record the described workflow verbatim | Manual | Manual |

State this table to the user at the moment they pick a non-GitHub tracker. It is the honest cost of that choice.

---

## 9. L4 — Skill installation

**`SKILLS-MATTPOCOCK`** — the pipeline bundle
- `SCOPE` MACHINE · `NEEDS` CC-CLI
- `DETECT` — `claude plugin list --json` shows `mattpocock-skills` installed and enabled, **or** the session's own skill listing contains the core names below, with or without a `mattpocock-skills:` prefix. Core names: `setup-matt-pocock-skills`, `grilling`, `grill-with-docs`, `to-spec`, `to-tickets`, `tdd`, `implement`, `wayfinder`, `code-review`, `triage`, `domain-modeling`, `codebase-design`, `improve-codebase-architecture`, `wizard`.
- `INSTALL` — **AFK**: `claude plugin install mattpocock-skills@claude-plugins-official --scope user --yes`. If the `claude-plugins-official` marketplace is missing (`claude plugin marketplace list`), that is a volatile-identifier case (§0.7) — stop, do not guess a source. `HUMAN` fallback, only when the subcommand is absent: `/plugin install mattpocock-skills@claude-plugins-official` in the CLI, then `/reload-plugins`.
- `VERIFY` — `claude plugin list --json`. Then the **restart batch** (§0.4): the skills load in the next session, and `CFG-SETUP-RUN` and wizard scripts (§0.8) need them loaded.
- Then apply **Branch F** for duplicates.
- `DEGRADES` — there is no pipeline

**`SKILLS-OWN`** — the author's own and modified skills
- `SCOPE` MACHINE · `NEEDS` CC-CLI · `GATE` always
- One plugin, `nythan-skills`, from the author's marketplace `nythan` (repo `nythanpienaar-cell/skills`). It bundles this setup skill and the four skills below, so they install together; the project-shape answers decide only which of them `docs/Pipeline.md` lists.
  - **`tutorial`** — Part 2 in the table at the top: the coaching skill that reads `docs/Pipeline.md`. **The setup skill installs it and never runs it**; the closing report ends by telling the user to type `/tutorial` next.
  - **`establish-architecture`** — user-invoked. Designs the project's architecture **foundation**: how four layers (modular monolith, vertical slices, hexagonal ports, deep modules) take shape in *this* project, written to `docs/architecture.md` with a pointer section appended to `CLAUDE.md`/`AGENTS.md`. It changes no code, and only writes a slice-generator script if the user asks for one. **The setup skill installs it and never runs it.** Its place in the pipeline, for `docs/Pipeline.md`: once per project, after `/to-spec` and before `design-system` and the first `/to-tickets` → `/implement`. It reads the plan from files (`CONTEXT.md`, ADRs, READMEs) and reads the spec issue from the tracker named in `docs/agents/issue-tracker.md`. It uses `codebase-design`'s vocabulary from `SKILLS-MATTPOCOCK`.
  - **`audit-architecture`** — user-invoked. Audits code against the foundation `establish-architecture` wrote and writes evidence-backed findings to `docs/architecture-audits/<date>.md`, each routed to a fix, `/to-spec` → `/to-tickets`, or `/grill-with-docs`. It changes no code. Stops if no foundation exists. For `docs/Pipeline.md`: after tickets have been built, every few tickets.
  - **`design-system`** — the modified BuilderOS skill (`version: 1.1-nb`, MIT, BuilderOS credited). Turns screenshots, mockups, Figma links (via `MCP-FIGMA`) and live websites (via `MCP-CHROME-DEVTOOLS`) into `docs/design.md` (for agents) and `docs/design.html` (for the human). **Install this copy, never upstream BuilderOS's** — upstream recommends BuilderOS planning skills that conflict with this pipeline.
- `DETECT` — `claude plugin list --json` shows `nythan-skills` installed and enabled, **or** the session's skill listing contains `tutorial`, `establish-architecture`, `audit-architecture`, and `design-system`, with or without a `nythan-skills:` prefix, and `design-system`'s frontmatter shows `1.1-nb` rather than upstream.
- `INSTALL` — **AFK**: `claude plugin marketplace add nythanpienaar-cell/skills` (skip if `claude plugin marketplace list` already shows `nythan`), then `claude plugin install nythan-skills@nythan --scope user --yes`. If the marketplace add fails because the repo or name has moved, that is a volatile-identifier case (§0.7) — stop, do not guess a source. `HUMAN` fallback, only when the subcommand is absent: `/plugin marketplace add nythanpienaar-cell/skills`, then `/plugin install nythan-skills@nythan` in the CLI, then `/reload-plugins`.
- `VERIFY` — `claude plugin list --json`, then the **restart batch** (§0.4), shared with `SKILLS-MATTPOCOCK`.
- **Branch F applies**: an older manual copy of any of these four (for example under `~/.claude/skills/`), or an upstream BuilderOS `design-system`, is a duplicate or conflicting install — offer to remove it, never delete without confirmation.
- `DEGRADES` — no `/tutorial` coaching; no architecture foundation for `/implement` to build into or audit against; no shared design rules for UI tickets

**`SKILLS-BUILDEROS`** — launch-checklist (unmodified upstream)
- `SCOPE` PROJECT · `NEEDS` NODE · `GATE` C3=yes
- `INSTALL` — **AFK**: `npx skills add BuildGreatProducts/builder-os/skills/launch-checklist`, with the CLI's non-interactive flags. Probe `npx skills add --help` for them; if the command can only run interactively, hand it over as `HUMAN` with the exact answers.
- `DEGRADES` — no go-live audit

**`SKILLS-TRIAGE-CHECK`** — is `triage` present?
- A gate, not an install. Its answer decides whether `GH-LABELS-TRIAGE` runs at all and whether `/setup-matt-pocock-skills` writes `triage-labels.md`. **Resolve before L5.**

---

## 10. L6 — Project configuration the skills read

The skills hardcode none of this. They read files. Absent files mean the skills either stop and ask for setup, or degrade quietly — and quiet degradation is the dangerous one.

### 10.1 Files produced

| File | Written by | Read by | Contains |
|---|---|---|---|
| `docs/agents/issue-tracker.md` | `/setup-matt-pocock-skills` | `to-tickets`, `triage`, `to-spec`, `code-review`, `wayfinder` | Which tracker, exact `gh`/`glab` conventions, the PRs-as-request-surface flag, **plus the §8.6 probe results** |
| `docs/agents/triage-labels.md` | `/setup-matt-pocock-skills` | `triage`, `to-tickets` | Canonical role → actual label string mapping |
| `docs/agents/domain.md` | `/setup-matt-pocock-skills` | domain-modeling, codebase-design | Single- vs multi-context layout and consumer rules |
| `CLAUDE.md` → `## Agent skills` | `/setup-matt-pocock-skills` | every session, automatically | Three one-line summaries pointing at the files above |
| `CONTEXT.md` | domain-modeling, over time | `code-review`, `implement` | Ubiquitous language for the project |
| `docs/adr/` | domain-modeling, over time | `code-review` | Architectural decision records |
| `CODING_STANDARDS.md` / `CONTRIBUTING.md` | the user, optionally | `code-review` Standards axis | Documented standards, which **override** the built-in Fowler smell baseline |

### 10.2 Components

**`CFG-SETUP-RUN`** — run `/setup-matt-pocock-skills` in this project
- `SCOPE` PROJECT · `NEEDS` SKILLS-MATTPOCOCK, and GH-REPO when D=github or WS-GITREPO for any other tracker · `GATE` always
- `DETECT` — `docs/agents/issue-tracker.md` exists
- `INSTALL` — `HUMAN`: `setup-matt-pocock-skills` is user-invoked, so the agent cannot run it. Hand it over as a baby step **with the answers pre-supplied** from Branch D and C4, so the user is not re-asked what the setup skill already determined. Its three questions: which tracker, which labels, where domain docs live.
- `DEGRADES` — `to-tickets`, `triage`, `code-review` all stop and demand it; `wayfinder` has no tracker conventions

**`CFG-CLAUDE-MD`** — the `## Agent skills` block is in the right file
- `SCOPE` PROJECT · `NEEDS` CFG-SETUP-RUN
- **Selection rule, in order:** if `CLAUDE.md` exists, edit it. Else if `AGENTS.md` exists, edit that. If neither exists, **ask which to create — do not pick.** Never create `AGENTS.md` alongside an existing `CLAUDE.md`, or the reverse.
- An existing `## Agent skills` block is updated in place. Never append a second one; never overwrite surrounding sections.
- `DEGRADES` — the pointer files exist but nothing tells a fresh session to read them

**`CFG-PROBE-RESULTS`** — §8.6 answers recorded into `issue-tracker.md`
- `SCOPE` PROJECT · `NEEDS` CFG-SETUP-RUN, GH-SUBISSUES-PROBE, GH-DEPENDENCIES-PROBE · `GATE` D=github
- The stock template assumes native sub-issues and native dependencies. Where a probe came back negative, the setup skill **edits the template** so the fallback is stated as the convention, rather than leaving `wayfinder` to discover it mid-run.
- `DEGRADES` — a half-built map and a confusing failure in the middle of planning

**`CFG-AUTOCOMMIT`** — standing commit-and-push instruction
- `SCOPE` PROJECT · `NEEDS` CFG-CLAUDE-MD · `GATE` offered always, applied on consent
- Add to `CLAUDE.md`: after each ticket is completed and its tests pass, commit and push automatically without being asked. With no remote (D = `local`), write "commit" only — a push instruction with nowhere to push fails every ticket.
- A convenience at E=`none`; close to a **requirement** at E=`ralph`/`sandcastle`, where no human is present to say "commit now" and uncommitted work is simply lost at cycle end.
- `DEGRADES` — under automation, silent loss of completed work

**`CFG-STANDARDS`** — a documented standards source
- `SCOPE` PROJECT · `GATE` optional, offer once
- `DETECT` — `CODING_STANDARDS.md` or `CONTRIBUTING.md` at the repo root
- `code-review` always applies its built-in Fowler smell baseline, so absence is not fatal — but a documented repo standard **overrides** the baseline, and without one every review is judged against generic heuristics alone.
- `DEGRADES` — reviews flag house-style choices as smells, repeatedly

---

## 11. L7–L9 — Glue, services, automation

Everything here except `MCP-CONTEXT7` is gated by project shape (Branch C) or by the `automations` argument. A component gated off is *Not needed* (§3), and its absence never breaks the core planning-and-building loop.

### L7 — Glue

Every MCP server below installs **AFK** and joins the §0.4 restart batch. Servers that need a sign-in then get one `HUMAN` baby step: in a new Claude Code session, type `/mcp` → pick the server → **Authenticate** → approve in the browser. Verify with `claude mcp list` (connected, not "needs authentication"); if that subcommand is unavailable, ask the user to run `/mcp` and report back rather than guessing.

| ID | Scope | Gate | Install | Verify | Degrades |
|---|---|---|---|---|---|
| `MCP-CONTEXT7` | MACHINE | **always**, needs `CC-CLI` | **AFK**: `claude mcp add --scope user --transport http context7 https://mcp.context7.com/mcp`. No account or key needed. A free API key (context7.com dashboard) only raises usage limits — offer it as an optional `HUMAN` step if the user hits rate limits, added as an `Authorization: Bearer` header and never echoed. Gives the agent current, version-specific library documentation so it writes code for today's APIs rather than remembered ones | `claude mcp list` shows `context7` | The agent writes code from memory; renamed or removed library functions surface as errors during `/implement` |
| `MCP-CHROME-DEVTOOLS` | MACHINE | C1=yes, needs `CHROME` | **AFK**: `claude plugin marketplace add ChromeDevTools/chrome-devtools-mcp`, then `claude plugin install chrome-devtools-mcp@chrome-devtools-plugins --scope user --yes`. No sign-in | `claude plugin list --json` shows it; `claude mcp list` lists `chrome-devtools` | The agent cannot see a real browser; UI bugs are diagnosed by guessing from source; `design-system` cannot screenshot live websites |
| `MCP-FIGMA` | MACHINE | C1a=yes | **AFK**: `claude plugin install figma@claude-plugins-official --scope user --yes` (fallback: `claude mcp add --scope user --transport http figma https://mcp.figma.com/mcp`). Then `HUMAN`: a Figma account if they lack one (check Figma's current plan requirements for MCP access rather than promising free), and the `/mcp` sign-in | `claude mcp list` shows `figma` connected | `design-system` cannot read Figma links; the user pastes screenshots instead |
| `MCP-SUPABASE` | MACHINE | C2=yes, needs `SUPABASE-PROJECT` | **AFK**: `claude mcp add --scope user --transport http supabase https://mcp.supabase.com/mcp`. Then `HUMAN`: the `/mcp` sign-in (OAuth — no token to create or paste). Supabase's docs show `--scope project`; user scope is chosen here so it is added once per machine | `claude mcp list` shows `supabase` connected | The agent cannot inspect the schema, run migrations, read logs, or generate types; the user relays database errors by hand |
| `PLAYWRIGHT` | PROJECT | C1=yes | `npm init playwright@latest` — **interactive**, hand to the user; defaults are fine. Downloads browser binaries (hundreds of MB) — say so first | `npx playwright test` passes the example tests | `/tdd` writes tests nothing can execute |
| `NETLIFY-CLI` | MACHINE | C3=yes, needs `NPM-GLOBAL-WRITABLE` | `npm install -g netlify-cli` | `netlify --version` | No local run of the full site before deploying |
| `SUPABASE-CLI` | PROJECT | C2=yes | `npm install -D supabase` (project-local by design; invoked as `npx supabase`) | `npx supabase --version` | No local database to test against |

### L8 — Services

| ID | Scope | Gate | Notes |
|---|---|---|---|
| `SUPABASE-PROJECT` | REMOTE | C2=yes | `HUMAN` — sign up, create the project, choose a nearby region, **set and safely record the database password**. It is shown once and needed again at link time; a lost password means a reset. Provisioning takes a minute or two — the dashboard appearing is the confirmation. |
| `SUPABASE-LINK` | PROJECT | C2=yes | `npx supabase login` (browser), `npx supabase init`, then `npx supabase link --project-ref <ref>` — **interactive, prompts for the database password**. Reference ID: dashboard → Project Settings → General. Verify `npx supabase status`. |
| `NETLIFY-SITE` | REMOTE | C3=yes | `HUMAN` — sign up. Then `netlify login` (browser), `netlify init` — **interactive**; choose create-and-configure and point at the repo from `GH-REPO`. **Netlify will ask GitHub for repo authorisation** — a second, separate OAuth grant from `GH-AUTH`, and users often expect to be already connected. Verify `netlify status`. |
| `ENV-FILE` | PROJECT | C2 or C3 = yes | Create `.env`. **`WS-GITIGNORE` must already cover `.env` before a single value is written.** Also explain the distinction a beginner will otherwise get wrong: a Supabase *anon/publishable* key is designed to be public, while a *service-role* key grants full database access and must never reach the browser or the repo. |

### L9 — Automation

**Locked.** Everything in this layer runs only under `/pipeline-setup automations` (§0.1a). Beginners do not need Docker or unattended loops, so a plain run neither installs nor mentions them.

| ID | Scope | Gate | Notes |
|---|---|---|---|
| `DOCKER` | MACHINE | E=sandcastle | **windows / macos:** Docker Desktop (`HUMAN`, may need admin rights and a restart); **leave it running** — Sandcastle needs it active. On Windows this may require enabling virtualisation features. **linux:** Docker Engine is enough — `HUMAN` in Terminal (sudo password): install from the distro or Docker's official repository per Docker's current docs, then `sudo usermod -aG docker $USER`, then **log out and back in** (or reboot) so the group change applies. **Verify on every OS with `docker info`**, run without sudo — `docker --version` only proves the client exists and passes while the daemon is stopped or unreachable. On Linux, a permission error from `docker info` means the log-out step was missed. If it fails on a managed machine, record as Blocked with the exact error. |
| `CC-TOKEN` | ACCOUNT | E=sandcastle | `claude setup-token` in the system terminal (`HUMAN`, opens a browser). **Secret — treat as a password.** The setup skill never echoes it, never logs it, never stores it outside `.sandcastle/.env`. Cannot be probed; ask directly whether one already exists. Requires a plan that permits it — check `ANTHROPIC-ACCOUNT`. |
| `SANDCASTLE-INIT` | PROJECT | E=sandcastle | **`NEEDS` DOCKER (verified with `docker info`), CC-TOKEN, CFG-SETUP-RUN.** `npx @ai-hero/sandcastle init` in the project folder — **interactive**: sandbox provider Docker, agent Claude Code, issue tracker matching Branch D; accept the container image build. Then capture the token with a **wizard script** (§0.8), writing `CLAUDE_CODE_OAUTH_TOKEN` to `.sandcastle/.env` — the token is typed into the script's hidden prompt, never into the chat. **Confirm `.sandcastle/.env` is gitignored before the script runs**, and run `mkdir -p .sandcastle` yourself (AFK) before handing over the run line — the template's `write_env` cannot create a missing directory and the script would die before capturing the token. |
| `RALPH-READY` | PROJECT | E=ralph or sandcastle | **`NEEDS` CFG-AUTOCOMMIT's offer, L5, L6, and SANDCASTLE-INIT when E=sandcastle.** No install. A **gate check**: L5 labels present, L6 config written, `CFG-AUTOCOMMIT` applied — if auto-commit was declined on an earlier run, re-offer it here, because it is required rather than optional under automation. An unattended loop starts each cycle with empty memory by design — everything it needs must already live in files, tickets, and commits. If any of the three is missing, refuse to declare automation ready. |

> **Concurrency warning, surfaced at E=sandcastle.** Two loops both told to pick the next unblocked ticket can claim the same one simultaneously. Run multiple loops only with one specific named ticket each, and only for tickets with no blocking relationship between them. This is why §8.6's dependency probe matters under automation.

> **Cost warning, surfaced at E≠none.** Unattended loops consume usage with nobody watching. Before the first unattended run, state plainly that a loop left running overnight bills for every cycle, and recommend starting with a single bounded run.

---

## 12. Verification matrix

The setup skill's final report. Every line is a real check, not a recollection of having run something.

| # | Check | Command / probe | Pass condition |
|---|---|---|---|
| 1 | Package manager | `winget` / `brew` / distro | version prints |
| 2 | Node | `node --version` | prints **and meets the version floor** |
| 3 | npm | `npm --version` | prints; global prefix writable if L7 planned |
| 4 | Git | `git --version` | version prints |
| 5 | Git identity | `git config --global user.name` / `user.email` | both non-empty |
| 6 | Git defaults | `init.defaultBranch`, plus OS-specific keys | set |
| 7 | Anthropic access | plan or credit confirmed | user-asserted, recorded as asserted |
| 8 | Claude Code CLI | `claude --version` | version prints |
| 9 | Claude Code desktop | app present | user-confirmed |
| 10 | Workspace sanity | Branch G probes | no unresolved hazard |
| 11 | Local repo | `git rev-parse`, `git rev-list -n1 --all` | work tree, at least one commit |
| 12 | `.gitignore` | file contents | covers `node_modules/`, `.env`, `.env.*`, `.sandcastle/.env`, build output |
| 13 | Secret scan | `git ls-files` plus content scan | no tracked secrets |
| 14 | GitHub CLI | `gh --version` | version prints |
| 15 | GitHub auth | `gh auth status` | logged in; **exactly one intended active account** |
| 16 | GitHub scopes | scope list **plus a real label round-trip** | create, read, delete all succeed |
| 17 | Credential helper | `git config credential.helper` *(no scope flag)* | set at any scope |
| 18 | Commit email | compare against account emails | matches a verified address or the noreply form |
| 19 | Repo + remote | `gh repo view` **and a real push** | both succeed |
| 20 | Repo permission tier | `gh repo view --json viewerPermission` | `write` or `admin` |
| 20a | Issues enabled | `gh repo view --json hasIssuesEnabled` | true |
| 21 | Triage labels | `gh label list` | all five roles resolvable, directly or via mapping |
| 22 | Wayfinder labels | `gh label list` | all five `wayfinder:*` present |
| 23 | Sub-issues capability | probe result | a definite yes/no written into `issue-tracker.md` |
| 24 | Dependencies capability | probe result | a definite yes/no written into `issue-tracker.md` |
| 25 | Probe cleanup | issue/label listing | no setup-created debris remains |
| 26 | Pipeline skills | `claude plugin list --json`, or `/skills` *(user-reported)* where the subcommand is absent | `mattpocock-skills` enabled with every core name from `SKILLS-MATTPOCOCK`, no duplicates |
| 27 | Per-project config | file listing | `issue-tracker.md` and `domain.md` exist; `triage-labels.md` exists iff `triage` is installed |
| 28 | Agent skills block | read `CLAUDE.md` / `AGENTS.md` | exactly one block, pointing at files that exist |
| 29 | Chrome *(if C1)* | installation check | present |
| 30 | Glue *(if C1)* | `claude mcp list`; `npx playwright test` | devtools listed; example tests pass |
| 31 | Services *(if C2/C3)* | `npx supabase status`, `netlify status` | linked, not erroring |
| 32 | Secrets hygiene *(if L8/L9)* | read `.gitignore` | `.env` and `.sandcastle/.env` both excluded |
| 33 | Automation gate *(automations run, E≠none)* | `RALPH-READY` | labels, config, auto-commit all confirmed |
| 34 | Git Bash *(windows)* | `GIT-BASH` detect | `bin/bash.exe` under Git's install root |
| 35 | Own skills | `SKILLS-OWN` detect | `nythan-skills` enabled: `tutorial`, `establish-architecture`, `audit-architecture`, modified `design-system` (`1.1-nb`), no duplicates |
| 36 | Context7 MCP | `claude mcp list` | `context7` listed |
| 37 | Figma MCP *(if C1a)* | `claude mcp list` | `figma` connected |
| 38 | Supabase MCP *(if C2)* | `claude mcp list` | `supabase` connected |

**End-to-end smoke test**, offered as the last optional step: create one throwaway issue via `gh`, apply `ready-for-agent`, read it back with `gh issue view --comments`, close it. If that round-trip works, the ticket half of the pipeline works. Clean up afterwards and say so.

**Closing report.** Finish in plain language: what is now working; **what was not installed because this project doesn't need it** (§3 *Not needed*), each with what it's for and that `/pipeline-setup` adds it later; and the re-run habit in one line. A beginner should be able to read it without knowing any of the component IDs. Under the loop-until-clear rule (§14) a closing report is only given on a clear recheck — a paused run gives a pause report instead (§14, *The user wants to stop*).

---

## 13. State file

`.pipeline/setup-state.md`, at the project root. Created on the **first recorded outcome of any kind** — a branch answer, a Completed, Blocked, Asserted, Skipped, or Not-needed entry — so a run where nothing installs (an IT-blocked machine) still resumes where it stopped. **Add `.pipeline/` to `.gitignore`, or accept that it is committed — decide explicitly rather than by accident.**

```md
# Pipeline Setup State

## Branches
- OS: {windows|macos|linux}
- Mode: {full|project|repair|verify}
- Shape: UI={y|n} Figma={y|n|-} Data={y|n} Public={y|n} Monorepo={y|n} External-issues={y|n}
- Repo visibility: {private|public}
- Tracker: {github|gitlab|local|other}
- Automation: {locked|none|ralph|sandcastle}
- Recheck passes: {n}
- Surface in use: {desktop|cli}

## Capabilities
- Sub-issues: {native|fallback}
- Dependencies: {native|fallback}
- Triage labels: {canonical|mapped|absent}

## Completed
- {COMPONENT-ID} — {date} — {one line, only if non-obvious}

## Asserted, not verified
- {COMPONENT-ID} — {what the user said} — {date}

## Not needed
- {COMPONENT-ID} — {question and answer that ruled it out} — {still installed from earlier: y|n}

## Skipped
- {COMPONENT-ID} — {reason} — {the DEGRADES consequence, stated}

## Blocked
- {COMPONENT-ID} — {what is needed from the user} — {exact error, if any}
```

**Rules.** Completed is append-only. Skipped entries must carry the consequence, not just the reason — that is what makes a later `repair` run useful. *Asserted, not verified* exists so a future run knows which "yes" came from a probe and which from a person. Anything under Blocked is what the setup skill leads with on resume. **Never write a secret, token, or database password into this file.**

---

## 14. Failure handling

### 14.1 Loop until clear

The recheck (whiteboard step 4) loops back to install (step 3) until every **needed** component passes. There is no retry cap, and `docs/Pipeline.md` is written only on a clear recheck — its "everything is installed" line must always be true.

- **What counts.** Needed = gated in by branches and not marked *Not needed*. Optional offers (`GIT-DEFAULTS`, `WS-README`, `CFG-AUTOCOMMIT`, `CFG-STANDARDS`) may be declined and logged under *Skipped*; they never hold the loop open.
- **Each pass**, re-offer every outstanding item to the user with its exact next action. The user's choices are: do it now; or **pause** (see *The user wants to stop*). A needed item cannot be skipped — if the user doesn't want a tool, the honest route is changing the Branch C answer that required it, which moves it to *Not needed*. Core items (L0–L6) have no such answer.
- **Blocked items still loop.** An item Blocked on something outside the session (an IT administrator, a placeholder the skill's author hasn't filled) is re-probed every pass. **When every outstanding item is Blocked on something the user cannot resolve in this session, the setup skill pauses on its own** — the same pause report as a user-requested stop — rather than spinning.
- Increment `Recheck passes` in the state file every pass.

| Situation | Setup behaviour |
|---|---|
| A verify fails right after a successful install | **Restart the shell and retry once** before declaring failure (§4). Most often a stale `PATH`. |
| A slash-command effect looks wrong right after install | Restart the Claude Code session (`/exit`, `claude`) and re-check before declaring failure. |
| A probe is inconclusive (command not found, ambiguous output) | Not a pass and not a fail. Ask the user directly about that one item, then take the answer at face value and log it under *Asserted*. |
| A component fails to install | Record under Blocked with the exact error. Continue with everything that does not depend on it; the §14.1 loop brings it back next pass. Never abandon the run over one failure. |
| The user claims something is done that cannot be verified (a saved token, an account) | Ask directly, accept, and record as asserted rather than verified. |
| A write fails with HTTP 403 | Check `GH-PERMISSION-TIER` first — the user may simply not have write access to this repo. If the tier is fine, it is `GH-SCOPES`: org policy, or an unauthorised SAML token. Re-run the §8.3 round-trip verification. |
| An install fails with *not found* (404, `ENOTFOUND`, "no such package") | A **volatile identifier** (§0.7) has probably moved. Stop that component, record the exact command and error, and point the user upstream. Never guess a replacement name or install a similar-looking package. |
| A probe's output is in an unexpected language or format | §0.3 rule 11 — the probe was keyed on prose. Re-probe using `--json` or the exit code. Never conclude a working tool is absent because its output was not in English. |
| A write fails with HTTP 404 on a repo that exists | Check `GH-ACCOUNT-SELECT` before anything else. The wrong active account presents as a missing repo. |
| `git push` asks for a password | `GH-CREDENTIAL-HELPER`, not `GH-AUTH`. GitHub no longer accepts account passwords; no password will ever work. |
| `git push` is rejected over email privacy | `GH-EMAIL-MATCH`. |
| Cleanup of a probe artefact fails | Report explicitly with the issue number. Never leave silent debris in a user's repo. |
| An install needs admin rights the user does not have | Record as Blocked with the exact error and the precise thing to ask an IT administrator for. Do not attempt workarounds. |
| A tracked secret is found | Stop that branch. State that the value must be rotated at its source and that removing the file does not remove it from history. Do not proceed to a public repo. |
| The user wants to stop | Write state, summarise what is done and what is still outstanding with each exact next action, and confirm that typing `/pipeline-setup` again resumes here. Do not write `docs/Pipeline.md`. Never leave a half-written config file behind. |
| A step needs newly installed skills or MCP servers loaded in-session | The §0.4 restart batch: save state, ask the user to start a new Claude Code session and type `/pipeline-setup`, and resume from the state file. |
| Everything passes | `verify` mode: no installs; write or refresh `docs/Pipeline.md`, then give the closing report. |

---

## 15. Deliberately out of scope

- Teaching what any of this means — that is Part 2, which reads `docs/Pipeline.md`.
- Running `establish-architecture` or `design-system` — the setup skill installs them; running them belongs to the pipeline itself.
- Editing or extending any installed skill.
- Installing L9 on a plain `/pipeline-setup` run.
- Choosing a tech stack or framework beyond the Supabase/Netlify defaults.
- Writing any application code, spec, ticket, or test.
- Model selection and cost optimisation beyond the warnings in §11.
- Removing a leaked secret from git history — the setup skill detects and warns; rewriting history is a deliberate, separate operation.
- Uninstalling or rolling back the pipeline.
- Anything requiring a payment method beyond confirming `ANTHROPIC-ACCOUNT`. The setup skill stops at the free tier of every other service.
