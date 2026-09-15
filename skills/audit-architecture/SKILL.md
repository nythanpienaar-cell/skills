---
name: audit-architecture
description: Audit existing code against the project's architecture foundation and write evidence-backed findings, each ready to fix directly or hand to spec, ticket, and ADR skills.
disable-model-invocation: true
---

# Audit Architecture

**Audit** the code against the project's architecture **foundation** — the standard `/establish-architecture` wrote — together with the ADRs that build on it, and report where the code parts ways with them. Each **finding** names a problem, shows the evidence, and recommends a solution, written so it can be fixed directly or carried into `/grill-with-docs`, `/to-spec`, or `/to-tickets`.

An audit observes and recommends. The one file it writes is its report; code, config, docs, ADRs, and the foundation itself stay exactly as found. What happens to a finding is the user's decision, made after the audit. The foundation is the fixed yardstick: when code suggests a foundation rule itself is wrong, that is a finding for the user to decide on, never a recommendation to edit the foundation.

**Evidence.** Every finding points at `path:line` in the code and at the foundation rule it departs from. A suspicion with no evidence is not a finding.

## 1. Preflight

Find the instructions file (`CLAUDE.md`, else `AGENTS.md`) and its `## Architecture foundation` section, which names the foundation's path. If there is no foundation, tell the user to run `/establish-architecture` first and stop — an audit with no foundation has nothing to measure against.

Read, in full:

- the **foundation** — its vocabulary, the four layers, the slice shape, the existing-code section, and its open questions;
- every **ADR** — noting which ones answer a foundation open question, and which supersede a foundation rule;
- the glossary (`CONTEXT.md` or equivalent);
- the **previous audit report**, if one exists, including the status the user gave each finding.

The **rules in force** are the foundation's rules, with each superseded rule replaced by its ADR's version, plus every answered open question as its ADR decided it.

Done when you can state the rules in force for each of the four layers, each with its source, and list every finding the user previously rejected.

## 2. Scan

Walk the code with the rules in force as the yardstick. For each layer, look for code that departs from the rule *as the foundation and its ADRs state it for this project*:

- **Modular monolith** — code reaching into another module's internals instead of its published interface.
- **Vertical slices** — one feature's logic scattered across layers or folders the slice shape would hold together.
- **Hexagonal ports** — domain logic reaching infrastructure directly. Count every route to the outside world: imported clients, the project's own wrappers around them, and calls that need no import at all (network, storage, environment, framework request objects).
- **Deep modules** — shallow modules whose interface is nearly as complex as their implementation. Apply `/codebase-design`'s deletion test.

Speak the foundation's vocabulary in everything you record, and `/codebase-design`'s terms for the architecture — **module**, **interface**, **depth**, **seam**, **adapter**, **leverage**, **locality**.

Three things are not findings: code that already matches the foundation, a departure an ADR deliberately chose, and a previously rejected finding whose code is unchanged. Record an ADR-sanctioned departure only when the friction is real enough to reopen that ADR, and mark the conflict. Where code touches a foundation open question no ADR has answered yet, report it as that question in action, and leave the question open. An answered question is a rule in force, audited like any other.

Done when every area of the codebase the foundation covers has been visited — list them — and each candidate has its evidence.

## 3. Shape the findings

Write each finding so it stands alone; a reader who sees only that finding can act on it.

- **Title** — the problem in a phrase.
- **Layer** — which of the four.
- **Problem** — what is wrong, with every `path:line` of evidence.
- **Rule** — the rule departed from, quoted from the foundation, or from the ADR that answered or superseded it.
- **Why it matters** — the cost in **locality** and **leverage**, and what it does to testing.
- **Recommended solution** — the target shape in plain language, described in the foundation's terms. Describe the change; leave the code to whoever fixes it.
- **Size and risk** — small, medium, or large; whether behaviour changes; whether tests cover the code today.
- **Confidence** — `Strong`, `Worth exploring`, or `Speculative`.
- **Depends on** — other findings that should land first.
- **Next step** — the route that fits:
  - *Fix directly* — small, no decision needed. Test-first when no test covers the code yet.
  - *`/to-spec` then `/to-tickets`* — large enough to plan and split.
  - *`/grill-with-docs`* — a real decision comes first: the solution needs one, the code has exposed something the foundation does not cover, or the evidence suggests a foundation rule is wrong for this project. That is where the ADR gets written, once the user decides — and the foundation stays as written.
- **Status** — `open`. The user changes it to `accepted` or `rejected: <reason>`.

Done when every candidate from step 2 is a finding with all of the above, or has been dropped for want of evidence.

## 4. Write the report

Write to `docs/architecture-audits/<YYYY-MM-DD>.md`, beside the foundation unless the project keeps such reports elsewhere.

```markdown
# Architecture audit — <date>

Measured against `<foundation path>` and the ADRs that build on it: <ADR numbers, or "none yet">. This report recommends; nothing in the code was changed.

## Summary
| # | Finding | Layer | Size | Confidence | Next step |
<one row per finding, ordered: dependencies first, then highest value for lowest risk>

**Start with:** <the finding to take first, and why>

## Since the last audit
<Findings now resolved · still open · rejected and still standing — or "First audit.">

## Findings
### 1. <Title>
<every field from step 3>

## Open questions in action
<Each still-unanswered foundation open question the code touches · the path:line where it shows · why it is not a finding — or "None touched.">

## Coverage
<Every area visited · anything the audit could not assess, and why>
```

Done when every finding appears in both the summary and the findings section, every open question the code touches is listed, and coverage lists every area from step 2.

## 5. Hand over

Show the user the summary table and the recommended starting point. Tell them each finding's status field is theirs to set, and that a rejected finding with a reason will not be raised again while its code is unchanged.

For a finding they want to act on now, give the exact next command — for example `/to-tickets docs/architecture-audits/<date>.md` scoped to finding 3. Run nothing further on their behalf.

## 6. Summary

Print the report's path, the number of findings by confidence, and the finding to start with. Nothing else.
