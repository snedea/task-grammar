# Task Grammar

**The rules for how the units of work you hand an AI agent should be
written. English is the new programming language; it shipped without a
style guide. This is the style guide for the layer nobody named.**

A spec says what to build. A task is the piece of it you hand a single
loop, the unit that gets marked done. Spec-driven development, as
GitHub's [Spec Kit](https://github.com/github/spec-kit) popularized it,
walks you from a spec, through a plan, into a numbered task list, and
then hands that list to an agent. It organizes that list well (see
Lineage below), and it never names a discipline for the unit inside
it. Task Grammar governs that unit: what makes one task well-formed
for a reader that is not human.

## The membership test

A rule belongs in Task Grammar only if it is true *because the reader
is an LLM*. If it would improve a ticket handed to a junior developer
just as much, it is good writing advice, and good writing advice
already exists. What is left after that filter is a grammar for a
reader with no memory of yesterday, a fixed window of attention, and a
habit of sounding confident either way.

## The seven rules

Every rule is a relationship, never a constant. A grammar with
constants in it dies with the next model release; the ratios survive.

| # | Rule | The relationship | Why it is LLM-native |
|---|------|------------------|----------------------|
| 1 | **Declare the cold start** | Every task carries its own opening state: the files that matter, the decisions it depends on, what the last task left behind. | The agent that picks up task six was not in the room for task five. Human teams amortize context; agents cannot. |
| 2 | **End with a check the agent can run** | A task closes with a self-executable verification. The weaker the model, the more executable (versus judgment-based) the check must be. | An LLM reports success either way. A runnable check converts a claim into a fact. |
| 3 | **Fence scope by naming what is in, not what is out** | Scope is an allowlist of files and areas. Translate any "do not touch X" into the list of what may be touched. | Telling the model not to touch X places X squarely in its attention. A human hears a prohibition; an LLM hears a topic. |
| 4 | **Restate the load-bearing constraint** | A constraint that must survive the whole run is repeated in the task even when the spec already states it, and because the spec already states it. | Attention decays with distance. An instruction from the top of a long run is a whisper by the middle of it. |
| 5 | **Spend the word budget where the ambiguity is** | Task prose competes with code for the same context window. Detail pays exactly where a human would have asked a question. | The model does not stop to ask. It resolves ambiguity by guessing, in full confidence. |
| 6 | **Let the artifact carry the state** | No task may reference the conversation. Every task boundary is a context wipe; state passes through files. | "As discussed" points at a discussion the executing agent never had. |
| 7 | **Size to the engine** | The one-pass budget (the largest chunk one loop reliably finishes) scales with model capability. A size constant is wrong within a model generation. | An oversized task loses the thread and stalls; an undersized one spends more on spin-up than on work. The right size is real but relative to who is driving. |

## The two skills

| Skill | Job | When |
|-------|-----|------|
| [`compose-tasks`](skills/compose-tasks/SKILL.md) | Generate a task list from a spec, with the grammar applied at write time. | You have a spec and need the TASKS.md an agent will execute. |
| [`task-grammar`](skills/task-grammar/SKILL.md) | Review an existing task list against the seven rules: a scorecard (verifiability, statefulness, scope discipline, fit), a readiness gate, and per-task rewrites. Also works backward from a failed run via its symptom index. | The tasks already exist, written by you or by another agent; or a loop run went wrong and you want to know which rule broke. |

The review skill is deliberately Grammarly-shaped: scored dimensions
with honest calibration, findings that quote the offending line and
teach the rule, and a gate that a fake verification caps regardless of
the other scores. Same concept, different grammar; the reader it
protects is an LLM, not a human.

They assume the split that is coming either way: increasingly, agents
write the task lists and humans review them. `compose-tasks` teaches
the writing agent the grammar; `task-grammar` arms the reviewer.

## Install

As a Claude Code plugin (the repo doubles as its own marketplace):

```
/plugin marketplace add snedea/task-grammar
/plugin install task-grammar@task-grammar
```

Or copy the skill folders into your agent's skills directory:

```
git clone https://github.com/snedea/task-grammar
cp -r task-grammar/skills/* ~/.claude/skills/
```

Or per-project: copy into `<project>/.claude/skills/`. The skills are
plain markdown and portable to any tool that reads skill files.

## Lineage

Rules for requirements are not new; this grammar stands on their
shoulders and differs in exactly one assumption, the reader.

- **EARS** (Rolls-Royce): constrained syntax for requirement sentences,
  alive today inside AI tooling. Assumes a human reader.
- **INVEST** (agile): what makes a user story well-formed. Assumes a
  human team with shared memory.
- **@tasks-md/lint** and structural task linters: check the shape of a
  tasks file, not the grammar of the work inside it.

### The nearest neighbor: GitHub Spec Kit

[Spec Kit](https://github.com/github/spec-kit) is the toolkit that made
spec-driven development a named practice: `/speckit.specify` through
`/speckit.plan` to `/speckit.tasks`, running across 30+ coding agents,
with a
[written methodology](https://github.com/github/spec-kit/blob/main/spec-driven.md)
and a constitution of project principles. Credit where due: its tasks
template DOES prescribe structure for the task list. Unique IDs, `[P]`
markers for parallel-safe tasks, user-story grouping with independent
checkpoints, explicit dependency ordering, a warning against "vague
tasks", and the instruction to "include exact file paths in
descriptions". That last one matters here: a second team, working from
spec-side reasoning, independently converged on naming the files a task
touches. This grammar reads that as validation of rule 3, not as
competition.

The difference is which layer the rules govern and which reader they
assume. Spec Kit's task rules organize the LIST: ordering, grouping,
parallelism, traceability to stories. They are good rules, and nearly
all of them would improve a sprint backlog handed to a human team,
which is exactly the membership test failing. What the template leaves
unaddressed is the set that exists because the reader is an LLM:
per-task verification is absent and tests are "OPTIONAL - only include
them if explicitly requested" (rule 2 here holds that a task without a
runnable check ends with a claim); no cold-start declarations (rule 1);
no model-relative sizing (rule 7); no treatment of prohibitions versus
allowlists (rule 3, beyond the file-path convergence); no restating of
constraints that must survive the run (rule 4). Its `/speckit.analyze`
command checks artifacts against each other for coverage and
consistency, which is the closest thing in the ecosystem to a task
check, and it still checks the list against the spec, not the task
against the reader.

So the relationship is complementary by construction: Spec Kit
generates and organizes the artifact; Task Grammar governs the units
inside it. Run both. Its `[P]` markers and dependency ordering are
orchestration concerns this grammar deliberately does not duplicate.

The rules here were extracted from a production app factory that
machine-lints its task files (every task carries a file allowlist and
exactly one verification line) and from a year of loop builds. That is
evidence, not proof. Proof looks like a benchmark: the same task
written several ways, run across models and tiers, pass rates
published. It does not exist yet; it is the roadmap.

## The loop these tools close

- [spec-grader](https://github.com/snedea/spec-grader) grades the spec
  before anything builds from it (also at
  [specgrader.com](https://specgrader.com)).
- **task-grammar** (this repo) governs the tasks the spec decomposes
  into.
- [doubt-in-the-loop](https://github.com/snedea/doubt-in-the-loop)
  falsifies the claims after the build.

Grade the input, grammar the work, doubt the output.

## License

MIT. By [Silviu Nedea](https://github.com/snedea).
