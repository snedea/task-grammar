---
name: task-grammar
description: Grammarly-style review of a task list against the seven rules of Task Grammar, BEFORE an agent loop runs it or AFTER a run went wrong. Scores statefulness, verifiability, scope discipline, and fit, then lists concrete per-task corrections with a readiness gate. Use when the user provides a TASKS.md or task list (human-written or agent-generated) and wants it checked, asks whether tasks are ready to execute, or asks why a loop run failed. Trigger with phrases like "grammar the tasks", "check my TASKS.md", "are these tasks ready", "why did the loop fail". Pass the file path or pasted tasks as args.
allowed-tools: Read, Grep, Glob, Bash
version: 2.0.0
license: MIT
author: Silviu Nedea <https://github.com/snedea>
tags:
- tasks
- agents
- review
- spec-driven
compatibility: Designed for Claude Code; portable to any tool that reads skill files
---

# Task Grammar Review

A task list is a program written in English, and this review is its
static analysis. You are checking whether each task will survive being
executed cold by an agent that was not in the room when the task was
written. The author may have been a human or another agent; the grammar
does not care. Local-first: you do the analysis yourself, no external
service. Modeled on the Grammarly writing-score shape (scored
dimensions, calibrated honestly, findings that teach), with the seven
rules of Task Grammar as the grammar being checked.

## False-positive guard (read first)

These constructions look like violations and are NOT. Never flag:

- **A "do not" in the SPEC.** Only tasks are fenced by allowlist. The
  spec is allowed to say out-of-scope; rule 3 governs tasks alone.
- **A prohibitive constraint that is an invariant, not a fence.**
  "Never log credentials" inside a task is a rule 4 restatement doing
  its job, not a rule 3 violation. Rule 3 findings are about SCOPE
  (which files and areas may be touched); behavioral always/never
  invariants belong in the task and are a strength.
- **Repetition of a load-bearing constraint.** Rule 4 deliberately
  repeats what the spec already says. Do not flag it as padding under
  rule 5; the two rules trade off by design, and rule 4 wins.
- **References to artifacts.** "Per SPEC.md section 3" or "the schema
  T2.1 wrote to db/schema.sql" is state traveling through files,
  exactly what rule 6 wants. Rule 6 forbids references to
  conversation, not references to artifacts.
- **A judgment-based check in a genuinely unrunnable environment.**
  If nothing is executable where the tasks will run and the task says
  so, a precise human-checkable acceptance line is the best available
  check. Flag it only when a runnable alternative exists.
- **Large tasks under a frontier engine.** Sizing is relative to the
  declared tier. Without a declared tier, report sizing findings as
  tier-dependent, never as absolute.

When a fix would trade one rule against another, choose the correction
that satisfies both (usually: move the detail, do not delete it); if
none exists, present both options and name the tradeoff.

## Establish context first

1. Identify the EXECUTING model tier if the user can tell you (sizing
   verdicts are relative to it; without it, flag sizing findings as
   tier-dependent rather than absolute).
2. Locate the spec the tasks decompose, if one exists. Rule 4 findings
   (missing restated constraints) require knowing which constraints in
   the spec are load-bearing.
3. Identify what is runnable in the execution environment, because a
   Verification line that names a command that will not exist there is
   a fake check. When in doubt, probe: `command -v <tool>` in the
   target environment settles it.

## The review

Check every task against each rule. Evidence is a quoted line or its
absence, never a paraphrase.

| # | Rule | What you are looking for |
|---|------|--------------------------|
| 1 | Declare the cold start | Does the task name its opening state: files, prior decisions, artifacts it depends on? Or does it assume the reader remembers? |
| 2 | End with a runnable check | Exactly one verification per task, phrased as something the executing agent can run, with an expected result. "Make sure it works" is a claim, not a check. |
| 3 | Fence by allowlist | Scope is a list of what may be touched. Every "do not touch X" is a finding: it primes X in the executing agent's attention. |
| 4 | Restate the load-bearing constraint | Constraints that must survive the run appear in the task itself, not only in the spec. |
| 5 | Word budget at the ambiguity | Detail sits where an executing agent would otherwise guess. Padding that restates what the files already say is a finding too. |
| 6 | Artifact carries the state | Zero references to any conversation: no "as discussed", "as above", "per our chat". State must travel through files. |
| 7 | Size to the engine | Each task plausibly one-pass for the declared tier: not so large it will lose the thread, not so small the spin-up outweighs the work. |

## Scoring

Score 0-100 per dimension, honestly calibrated: 90s = hand it to the
loop, 70s = fix the findings first, below 70 = recompose from the spec
(hand off to `compose-tasks`). Each dimension aggregates the rules it
covers across ALL tasks; one bad task drags the dimension.

| Dimension | Rules | What it measures |
|-----------|-------|------------------|
| **Verifiability** | 2 | Every task ends in a check the executing agent can run, with an expected result. |
| **Statefulness** | 1, 6 | Every task opens cold and state travels through artifacts, never through memory of a conversation. |
| **Scope discipline** | 3, 4 | Fencing is by allowlist and load-bearing constraints are restated where they must survive. |
| **Fit** | 5, 7 | Detail sits at the ambiguity and each task is one-pass for the declared engine. |

**Readiness gate** (a gate, not an average): the list is READY only if
Verifiability >= 90 AND overall >= 80 AND no HIGH finding remains.
A fake or missing check caps the whole list at FIX FIRST no matter how
the other dimensions score, because it is the failure nobody detects:
the loop reports success either way. Verdicts: READY, FIX FIRST, or
RECOMPOSE.

## Output format

1. **Scorecard**: the four dimension scores, the overall score, and
   the gate verdict (READY / FIX FIRST / RECOMPOSE) with the gating
   reason when it is not READY.
2. **Verdict table**: rule number, PASS or FLAG, one evidence line.
3. **Findings, ranked by expected failure mode** (worst first):
   - missing or fake check (rule 2): the loop will report success either
     way and nobody can tell,
   - oversized task (rule 7): the loop starts strong, loses the thread,
     ends doubting its own build,
   - conversation reference or missing cold start (rules 6, 1): the
     executing agent starts from state that does not exist,
   - "don't" fencing (rule 3): the forbidden thing enters the agent's
     attention,
   - unrestated constraint (rule 4): the guardrail is a whisper by the
     time it matters,
   - undersized task (rule 7): the queue pays spin-up repeatedly for
     trivial work,
   - misplaced word budget (rule 5): detail where none was needed,
     silence where the agent will guess.
   Each finding carries: the exact quoted line (or the named absence),
   what is wrong, the corrected task line ready to paste, and the rule
   in one plain-language sentence. Teach, do not just patch; skip the
   teaching note only for trivial mechanical fixes.
4. **NO FINDINGS is allowed.** If the list passes, give the scorecard,
   one PASS line per rule, and stop. Do not invent findings to appear
   thorough.

Rewrites are the deliverable, not lectures: the user should be able to
apply the review mechanically. Never edit the task file itself without
the user's go-ahead, unless they asked for the fix in the same message.

## Symptom index (reviewing after a failed run)

When the user arrives with a bad loop run instead of a task list, work
backward from the symptom to the rule, then review the list as above
with that rule in focus.

| Observed failure | Likely rule | The fix |
|------------------|-------------|---------|
| Loop reported success but the feature does not work | 2 | Replace claim-shaped checks with one runnable verification and its expected result. |
| Agent rebuilt or contradicted something a previous task already did | 1, 6 | Declare the cold start; pass the prior task's output through a named artifact. |
| Agent modified exactly the thing it was told not to touch | 3 | Translate the prohibition into an allowlist of what may be touched. |
| A spec constraint held early, broke late in the run | 4 | Restate the constraint inside the task where it must survive. |
| Agent finished strong then unraveled, second-guessing done work | 7 (oversized) | Split at a boundary where an artifact can carry the state across. |
| Run spent most of its budget re-reading context for a trivial change | 7 (undersized) | Merge undersized neighbors that share an allowlist. |
| Agent guessed confidently at something the author knew but never wrote | 5 | Move the word budget to that ambiguity; cut padding elsewhere. |

## Study the examples

Study `examples/weak-and-strong.md` in this repo: the weak version is
what findings look like in the wild; the strong version is the shape
your rewrites should produce.

## Next steps

- Score below 70, or the user wants the list rebuilt: hand off to
  `compose-tasks`, which applies the grammar at write time.
- List is READY and the loop has run: the claims the loop produced are
  the next thing to check; that is doubt-in-the-loop's job, not this
  skill's.
