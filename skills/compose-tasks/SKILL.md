---
name: compose-tasks
description: Generate a task list (TASKS.md) from a spec with Task Grammar applied at write time. Use when the user has a spec, requirements doc, or build brief and needs the executable task list an agent (or a queue of agents) will run. Every emitted task declares its cold start, carries a file allowlist, restates any load-bearing constraint, and ends with exactly one runnable verification. Trigger with phrases like "compose the tasks", "turn this spec into tasks", "write the TASKS.md". Pass the spec path as args.
allowed-tools: Read, Write, Edit, Grep, Glob
version: 2.0.0
license: MIT
author: Silviu Nedea <https://github.com/snedea>
tags:
- tasks
- agents
- spec-driven
- codegen
compatibility: Designed for Claude Code; portable to any tool that reads skill files
---

# Compose Tasks

You are writing for a reader that is not human: an executing agent with
no memory of the conversation that produced the spec, a fixed attention
window, and no ability to ask a clarifying question mid-run. Every task
you emit must survive being read cold, alone, by a different model than
you.

## Prerequisites

- A spec, requirements doc, or build brief exists as a file. If the
  requirements live only in the conversation, write them to a spec
  file first; a task list composed from chat memory violates rule 6
  before the first task is written.
- The executing model tier is known, or the user has been asked (see
  step 2 below).

## Before writing anything

1. **Read the entire spec first.** Extract, into working notes:
   - the constraints that must survive the whole build (security rules,
     platform limits, "always/never" statements),
   - the out-of-scope list,
   - the vocabulary the spec treats as canonical.
2. **Establish the engine.** Ask the user which model tier will EXECUTE
   the tasks if it is not stated. Sizing is relative: a one-pass task
   for a frontier model is two or three passes for a smaller one. If the
   tier is unknown and the user cannot say, size for a mid-tier model
   and say so in the task file header.
3. **Establish the verification vocabulary.** What can the executing
   agent actually run in its environment: a test command, a build, a
   validator script, a grep? Checks must be phrased in commands that
   exist there. If nothing is runnable, say so to the user before
   composing; a task list with no executable checks is a list of
   claims.

## The task format

```
- [ ] T<N>.<M>: <imperative title, the outcome not the activity>
  - Files: <allowlist of files/dirs this task may create or modify>
  - Opens: <the state this task starts from: decisions made, artifacts
    that exist, what the previous task left behind>
  - Constraint: <restated load-bearing constraint, only when one must
    survive this task; omit the bullet otherwise>
  - Verification: <exactly one runnable check with its expected result>
```

Rules of the format:

- **One `Verification:` bullet per task, always, never zero, never
  two.** If a task needs two verifications, it is usually two tasks.
- **`Files:` is an allowlist.** Never emit "do not touch X" in a task.
  Translate every out-of-scope statement and every "don't" in the spec
  into the list of what MAY be touched. Naming the forbidden thing
  places it in the executing agent's attention; naming the permitted
  things steers without priming. The spec is allowed to say
  out-of-scope; tasks are not.
- **`Opens:` replaces memory.** The executing agent was not in the room.
  Never write "as discussed", "as above", "per the conversation", or a
  reference to any earlier chat. State passes through files: the spec,
  this task list, and artifacts previous tasks wrote.
- **Restate, do not trust distance.** If a constraint from the spec must
  still be holding at the end of a long task, repeat it in the task even
  though the spec says it, and because the spec says it. Attention
  decays with distance; a constraint two files away is a whisper.

## Sizing

Aim every task at the one-pass size: the largest chunk the EXECUTING
model reliably finishes start to end without losing the thread and
without spending more on spin-up than on work.

- Oversized symptoms you are designing against: the agent starts strong,
  second-guesses finished work midway, and ends uncertain of the whole
  build. If a task's Files list is growing past what one context can
  hold alongside the code itself, split at a boundary where the
  artifact can carry the state across.
- Undersized symptoms: the agent spends most of the run re-reading
  context to make a trivial change. Merge undersized neighbors that
  share an allowlist.
- Prefer fewer, larger tasks up to the one-pass ceiling; the pipeline
  around the task handles breadth better than the task handles depth.
- Never state a size constant (no "under N lines", "at most M files").
  Sizing is a judgment against the declared engine, revisited when the
  engine changes.

## Word budget

Task prose and the codebase compete for the same context window. Spend
detail exactly where a human reader would have stopped to ask a
question, because the executing agent will not stop; it will guess in
full confidence. Everywhere else, be brief. Do not restate what the
allowlisted files already say.

## Self-review before returning

Check the composed list against the seven rules of Task Grammar (see
the repo README). For each task confirm: cold start declared, one
runnable check, allowlist fencing (zero "don't" phrasings), load-bearing
constraints restated, detail placed at ambiguity, zero conversation
references, sized to the declared engine. Fix violations before
presenting. Study `examples/weak-and-strong.md` in this repo for a
worked pair; the strong version is the target shape.

Then present the list with the scorecard the `task-grammar` skill
defines (Verifiability, Statefulness, Scope discipline, Fit, each
0-100, plus the gate verdict). A composed list should score READY; if
your own scoring says FIX FIRST, fix it before returning, not after.

## Next steps

- An independent pass with the `task-grammar` skill catches what
  self-review misses, especially when a different agent (or the human)
  runs it: the composing agent grading its own work is the weakest
  form of review.
- After the loop runs the list, the loop's claims need falsifying;
  that is doubt-in-the-loop's job, downstream of this skill.
