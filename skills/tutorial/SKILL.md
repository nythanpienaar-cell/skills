---
name: tutorial
description: Plain-English coaching for the next step of the AI coding pipeline in docs/Pipeline.md. Use when the user types /tutorial, or, only in a project that already has .pipeline/tutorial.md, when a pipeline stage just finished or the user asks what to do next.
argument-hint: "(optional) what you're stuck on"
---

# Tutorial

Like a game's tutorial level: the user learns the controls by playing. The pipeline is installed. Now the user needs to use it, one step at a time. You are the **coach**. The user types the commands. You show them which step comes next, why it matters, and how to tell when it's done.

**Plain voice** is how you say everything: short sentences, everyday words, one idea per message. When a pipeline word shows up (spec, ticket, commit), explain it the first time in five words or fewer. Tie every explanation to *their* project, never to a made-up example.

**Who started this run.** If the user typed `/tutorial`, go ahead. If you picked it up on your own, first check that `.pipeline/tutorial.md` exists in the project root. If it doesn't, this project hasn't opted in: carry on with what you were doing and don't start the skill.

## 1. Find Pipeline.md

Look for `docs/Pipeline.md` in the project root.

- **Found:** go to step 2.
- **Not found:** say, in plain voice, that the tools need installing first and `/pipeline-setup` does that. Then stop.

Done when you've read `docs/Pipeline.md`, or you've told the user to run `/pipeline-setup` and stopped.

## 2. Read Pipeline.md

Read `docs/Pipeline.md` in full, plus `.pipeline/tutorial.md` if it exists. Pipeline.md gives you the project's stages, the order they run in, what each stage reads and writes, and what was left out. The progress file gives you where coaching stopped last time and what the user already knows.

Done when you can name every stage this project has, and every tool it left out.

## 3. Find where the project is

Use **evidence**, not memory or guesses. For each stage in order, check whether its *Writes* column exists in the project: the files, the tickets in the tracker (listed with the tracker's own CLI when *Project shape* names one), and the commits. Stage 0 has no output. Treat it as done once any later stage has evidence.

The **current stage** is the first one whose evidence is missing or incomplete. Stage 7 repeats, once per open ticket, so its next step is the next open ticket, and stage 8 follows every ticket. Put the evidence in front of the user in one or two lines ("I see a spec and 6 tickets, 2 closed") and ask if that's right. If they correct you, go with their answer.

Done when you've named one current stage, backed it with evidence, and the user has confirmed it.

## 4. Coach one next step

Teach **one next step** and nothing beyond it. The steps after it can wait.

1. **Check for gaps.** If the step needs a tool listed under *Not installed for this project*, say so plainly. Explain that `/pipeline-setup` adds it (answer *yes* to its question) and stop there.
2. **Read the real skill.** Open the installed `SKILL.md` for this stage's command. Get what it does from that file, not from memory, because skills change between versions. Stage 0 has no skill: its question is in the stage table, so ask it yourself.
3. **Teach just enough.** Cover four things, in plain voice: *why* this step matters for their project (one line), *what to type*, *what it will ask them or produce*, and *what done looks like*. Leave out anything this step doesn't need. If an argument was passed, start from what they're stuck on.
4. **Quick check.** Before they type the command, ask one question they have to answer from memory, for example "What file will this make?" or "Why does the spec come before tickets?" If they get it wrong, give the right answer kindly and ask once more with different wording. Skip the check for anything the progress file lists under *Shown*.
5. **Hand over.** Give them the exact command on its own line, and tell them that once it finishes, you'll check the result and coach the step after it.

Done when the user has the command, has passed the check or chosen to skip it, and knows what done looks like.

## 5. Record progress

Write `.pipeline/tutorial.md`, creating it on the first run. This file is also what lets you pick up coaching on your own later. If `.pipeline/` is new to the project, add it to `.gitignore` the same way `/pipeline-setup` does.

```md
# Tutorial progress

Last coached: {YYYY-MM-DD} · Stage {#} {name} · Next command: `{command}`

## Shown
- {Something the user proved they understand, with how: a check they answered, a step they did right}

## Already knew
- {Prior knowledge the user mentioned, and how deep it goes}

## Mixed up
- {A misunderstanding you corrected, and the right idea}
```

A step you only explained doesn't go under *Shown*. Wait until the user actually shows they get it. Update entries in place instead of adding a new line for each session.

Done when the file names the current stage and next command, and every check the user answered this run is recorded under *Shown* or *Mixed up*.

## After a stage finishes

When the user comes back, or you pick this skill up because a stage just ended: check that the stage's *Writes* now exist. If they do, say so in one line and go back to step 3. If something is missing, point to exactly what's missing and coach them through finishing it before moving on.
