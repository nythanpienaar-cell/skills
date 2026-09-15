---
name: pipeline-setup
description: Checks this computer and project against Playout, installs everything the AI coding pipeline needs, and writes docs/Pipeline.md.
disable-model-invocation: true
argument-hint: "(optional) automations"
---

# Pipeline Setup

Get this computer and this project to where every pipeline skill works, then hand off through `docs/Pipeline.md`.

[Playout.md](Playout.md) is the **checklist** — every component, its probe, its install, its failure handling. This file is the process; Playout is what the process runs against. Read Playout in full before step 1 (it spans more than one Read); every `§` below points into it. Where Playout and the machine disagree, the machine is right (§0.7).

**AFK first.** Everything you can install, link, or configure, you do yourself — including anything that only needs the user to approve a permission prompt or click Yes on an admin dialog. The user gets only what is truly theirs: accounts, browser sign-ins, typed passwords, slash commands. The line is Playout §0.4; how to guide those steps is §0.8.

**Argument.** None: everything L0–L8 the project needs; L9 locked — nothing offered or installed, though an automation set up by an earlier run is still re-checked. `automations`: also Branch E and L9. See §0.1a.

## 0. Resume and preflight

Look for `.pipeline/setup-state.md` in the project root.

- **Found:** lead with anything under Blocked and reuse recorded answers, except: Branch C is always re-asked; Branch D is re-offered when recorded as `local` or `other`; Branch E is asked whenever the argument is `automations` (§3).
- **Not found:** give the §0.5 preflight, only the lines for this OS.

Done when the user knows what is about to happen, and you know whether this run is fresh or a resume.

## 1. Check the device

Probe only; install nothing.

1. Detect the OS (Branch A) and run every environment-hazard probe (Branch G).
2. Run `DETECT` on every L0–L4 component whose `GATE` does not depend on a Branch C, D, or E answer.
3. Interview: Branch C, D, and — only under `automations` — E. One question at a time, plain language, recommended answer first.
4. Run `DETECT` on every remaining component the answers gate in.
5. Settle the mode (Branch B) from all of the above.

Key every probe on exit codes and `--json`, never on English output (§0.3 rule 11). An inconclusive probe, or one Playout marks as deferred (`GH-ACCOUNT`), means asking about that one item and recording the answer as asserted. Write the state file as soon as there is anything to record (§13).

Done when every component in Playout has exactly one result — present, missing, blocked, asserted, or **not needed** with the answer that ruled it out — and the branches are recorded.

## 2. Build the checklist

List every missing needed component in dependency order (`NEEDS`, plus the §2 hard ordering constraints), each tagged **AFK** or **HUMAN**. Show the user, in plain language:

- what is already there, and the probe that proved it;
- what you will install yourself;
- what you will need them for;
- what is not needed for this project, and why.

If nothing is missing, go to step 5.

Done when the user has seen the checklist and every missing component is on it with its tag.

## 3. Install

Work the checklist top to bottom.

- **AFK:** run `INSTALL`, then `VERIFY` — restarting the shell and retrying once before calling it failed (§4). Ask once for consent before the first `winget` licence acceptance.
- **HUMAN:** baby steps in chat, or a wizard script for steps that capture values (§0.8). The first terminal moment gets the §0.6 orientation; the first Git Bash moment gets its explanation.
- **Restart batch:** group installs that load only in a new Claude Code session; when the next item needs them loaded, save state and hand over the restart (§0.4).
- **Failures:** the §14 table.

Write each outcome to the state file (§13) before starting the next item.

Done when every item on this pass's checklist is verified, Blocked with its exact error, or the user has paused.

## 4. Recheck

Re-run `DETECT` and `VERIFY` for **every** needed component from step 1 — not only this pass's installs, since an install can break a neighbour. Increment `Recheck passes`.

- **Clear** — every needed component passes: go to step 5.
- **Not clear** — rebuild the checklist from what failed, show it as in step 2, and go back to step 3. Loop until clear; §14.1 covers what counts, pausing, and Blocked items.

Done when the recheck is clear, or the run is paused — by the user, or on its own because everything still outstanding is Blocked on something outside this session (§14.1). A paused run gives the pause report (§14) and stops without writing `docs/Pipeline.md`.

## 5. Write docs/Pipeline.md

Create `docs/` if it does not exist. Read [PIPELINE-FORMAT.md](PIPELINE-FORMAT.md) and write `docs/Pipeline.md` in exactly that format. Then give the closing report (§12), ending with one line: type `/tutorial` to be coached through the first step.

Done when `docs/Pipeline.md` exists; it opens with "Everything is installed."; every not-needed component appears under *Not installed for this project*; every stage and tool it lists is installed; and no `{placeholder}` survives.
