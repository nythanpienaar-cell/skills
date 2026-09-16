---
name: establish-architecture
description: Design the four-layer architecture foundation for a project from its existing plan, so implementation work builds on it.
disable-model-invocation: true
---

# Establish Architecture

Design the project's architecture **foundation**: a written structure showing how four layers — **modular monolith**, **vertical slices**, **hexagonal ports**, **deep modules** — take shape in *this* project. Implementation work reads the foundation and builds to it.

**The foundation is the standard.** It is written once and then holds still. Decisions made later accumulate as ADRs that build on it; the foundation itself is never re-derived. A foundation rule changes only when the user deliberately decides it should, recorded as an ADR that names the rule it supersedes.

This skill designs. The code stays exactly as it is — nothing is moved, refactored, or retrofitted. Finding where existing code departs from the foundation is `/audit-architecture`'s separate job. The skill settles no open choice on the user's behalf either: where the plan leaves something undecided, the foundation names the question and leaves it open.

**Sourced.** Every statement the foundation makes about the project cites the file or issue it came from. What you cannot source, you do not state. The foundation adds structure; it never invents facts, reasons, or history.

## 1. Preflight

**Instructions file.** `CLAUDE.md` if it exists, `AGENTS.md` otherwise. If neither exists, ask which to create.

**Existing foundation.** Search that file for `## Architecture foundation`. If it is there, the foundation already stands: go to **Linking decisions** at the end of this skill, and skip steps 2–7. Otherwise, continue with step 2.

Done when you know the instructions file, and whether a foundation already stands.

## 2. Read the plan

The project already has a plan, whether or not it calls it one. Read all of it before designing anything:

- **Vocabulary** — `CONTEXT.md`, `CONTEXT-MAP.md`, any glossary. Note every term, and every _Avoid_ list.
- **Decisions** — every ADR, wherever the project keeps them.
- **Intent** — PRDs, specs, roadmaps, READMEs, build or workflow docs, the instructions file itself.
- **Specs on the issue tracker** — a spec is often an issue, not a file. If `docs/agents/issue-tracker.md` exists, follow its commands to find the spec issues (for example, open issues labelled `ready-for-agent` that have a *Problem Statement* and *Solution*) and read each one with its comments. If more than one could be the current spec, ask the user which. Cite an issue by its id or URL.
- **Existing structure** — how source is sorted today, and which modules already hold the core logic.
- **Framework constraints** — locations the language or framework imposes: where routes, pages, entry points, and handlers must live to be reachable at all.
- **Test convention** — where tests sit, how they are named, and any rule about when they are written.

Done when every document and spec issue above has been read, and you can list what the plan already decides about structure, each item with its source.

## 3. Design the foundation

Map the four layers onto the plan — the plan's existing decisions are the ground the foundation stands on.

**Names first.** Check every name the foundation will introduce — folder names, file-role names, the layer names themselves — against the glossary. When a name already means something in this project's domain ("module", "rule", "handler", "port" are the usual collisions), choose a name that does not clash and say which clash it avoids. Where the project already has a name for one of the four concepts (a type called `Writer` that is really a port; a named engine that is already a deep module), use the project's name and map it to `/codebase-design`'s term — **module**, **interface**, **depth**, **seam**, **adapter**, **leverage**, **locality** — rather than introduce a second word.

**Then the layers.** For each of the four, write down:

- what the plan already establishes, with its source;
- the structural rule the foundation adds;
- where that rule meets the plan awkwardly — as an open question.

**Then the slice shape.** Design the shape one feature takes, in the project's own spelling:

- the folder layout, at whatever depth fits the project (not necessarily `<module>/<feature>`);
- the role of each file — wiring, domain logic, and one adapter per external dependency;
- where its tests go, following the project's test convention;
- how a slice becomes reachable — which framework-imposed entry point wires it in.

**Open questions.** Every conflict, gap, or undecided choice goes here, never resolved silently. For each, state what it would affect. These are what the project's future ADRs start from.

Then show the user the draft foundation. Where an open question blocks the slice shape itself, ask it now and record the user's answer as theirs. Let them edit the draft before anything is written — this is the one moment the standard is set.

Done when the user has seen the draft, every name has been checked against the glossary, and every statement about the project carries a source.

## 4. Write the foundation

Write it where the project keeps design docs — `docs/architecture.md` unless the project has a clear home of its own. Keep it as short as the project allows; implementation work reads it before every feature.

```markdown
# Architecture foundation

The standard new work builds on. Existing code keeps its current shape. Later decisions are recorded as ADRs that build on this document; a rule here changes only by an ADR that names the rule it supersedes.

## Vocabulary
<Names this foundation uses, what each maps to in /codebase-design's vocabulary, and any domain-term clash each name avoids.>

## The four layers here
### Modular monolith
<What the plan already establishes (source) · the structural rule · open points>
### Vertical slices
<same>
### Hexagonal ports
<same>
### Deep modules
<same>

## Slice shape
<Folder tree in the project's spelling · the role of each file · where tests go · how a slice is wired into its entry point>

## Existing code
<How today's structure relates to the foundation — a description of what exists, not a migration plan. `/audit-architecture` turns the specific departures into findings.>

## Open questions
<Each question · what it affects · the user's answer, if they gave one during design>

## Sources
<Every file and issue this foundation draws on>
```

Done when the file matches the draft the user approved.

## 5. Point implementation at it

Append to the end of the instructions file, leaving every existing line intact:

```markdown
## Architecture foundation

Before implementing a feature, read `<foundation path>`, then the ADRs that build on it, and build the feature in the foundation's slice shape. Where an ADR supersedes a foundation rule, follow the ADR.

When the work reaches a choice the foundation leaves open, surface it rather than deciding silently; a decision worth keeping becomes an ADR. Changing a foundation rule is the user's decision, recorded as an ADR that names the rule it supersedes.
```

Done when the section is in that file exactly once, pointing at the real path.

## 6. Offer the slice generator

Ask whether the user wants a script that creates a new, empty slice in the designed shape. If not, skip this step.

If they do: write it in whatever the project already runs scripts with, placed where the project keeps scripts. It takes the slice's name(s), refuses to overwrite an existing slice, and creates exactly the files the foundation's **Slice shape** lists — including the test file, when the convention has one — each opening with a one-line comment stating its role. Record the project's existing checks before running it, so a pre-existing failure is not mistaken for one the script caused.

Done when the script and its output both pass those checks, a run with throwaway names produced exactly the foundation's slice shape, and the throwaway slice is deleted. Add its command to the step-5 section.

## 7. Summary

Print every path written or changed, one per line, then the number of open questions the foundation carries. Nothing else.

## Linking decisions

When a foundation already stands, this skill makes at most two small, visible changes, and designs nothing:

- **Answered questions.** Read every ADR written since the foundation. For each open question an ADR answers, add `Answered by ADR-NNNN` beneath that question. The question's text stays as written.
- **The instructions section.** If its wording differs from the step-5 block, replace that section with the step-5 block, pointing at the same path.

Show the user both changes as a diff and write them only on their approval. Every rule, the vocabulary, and the slice shape stay exactly as they are — including where an ADR supersedes a rule, since that ADR already carries the change. If the user wants a foundation rule itself reworded, make only the change they name, and cite the superseding ADR beside it.

Done when every open question an ADR answers carries its link, and nothing else in the foundation has changed.
