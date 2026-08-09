---
name: task-grammar
description: Review an existing task list against the seven rules of Task Grammar BEFORE an agent loop runs it. Use when the user provides a TASKS.md or task list (human-written or agent-generated) and wants it checked, or asks whether tasks are ready to execute. Returns per-rule verdicts with evidence, findings ranked by expected failure mode, and concrete rewrites. "NO FINDINGS" is an allowed answer.
---

# Task Grammar Review

A task list is a program written in English, and this review is its
static analysis. You are checking whether each task will survive being
executed cold by an agent that was not in the room when the task was
written. The author may have been a human or another agent; the grammar
does not care.

## Establish context first

1. Identify the EXECUTING model tier if the user can tell you (sizing
   verdicts are relative to it; without it, flag sizing findings as
   tier-dependent rather than absolute).
2. Locate the spec the tasks decompose, if one exists. Rule 4 findings
   (missing restated constraints) require knowing which constraints in
   the spec are load-bearing.
3. Identify what is runnable in the execution environment, because a
   Verification line that names a command that will not exist there is
   a fake check.

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

## Output format

1. **Verdict table**: rule number, PASS or FLAG, one evidence line.
2. **Findings, ranked by expected failure mode** (worst first):
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
3. **Rewrites, not lectures.** For each finding, show the corrected task
   line or bullet, ready to paste. The user should be able to apply the
   review mechanically.
4. **NO FINDINGS is allowed.** If the list passes, say so in one line
   per rule and stop. Do not invent findings to appear thorough.

Study `examples/weak-and-strong.md` in this repo: the weak version is
what findings look like in the wild; the strong version is the shape
your rewrites should produce.
