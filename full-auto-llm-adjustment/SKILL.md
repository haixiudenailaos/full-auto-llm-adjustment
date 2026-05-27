---
name: full-auto-llm-adjustment
description: "Use when an agent must enter an unfamiliar project or environment, collect user clues first, establish safe initial behavior with an AGENT.md or CLAUDE.md contract, learn local constraints from evidence, and drive the work forward through a Hermes-style iterative loop of summarize -> infer root cause -> dead-end check -> targeted research -> minimal smoke -> log and repeat."
---

# 全自动大模型调整

Use this skill when the agent is working in a new, poorly documented, or high-uncertainty environment and needs a reliable way to make progress without thrashing.

This skill is no longer XGuard-specific. Its job is to help the agent:

- pull as many useful clues as possible from the user first
- constrain early behavior before acting
- discover local rules from evidence instead of assumption
- iterate toward the goal with small falsifiable steps
- keep progress legible across repeated runs

## Core Principle

Before trying to solve the task, collect clues from the user, then constrain the agent's starting behavior.

In unfamiliar environments, the first failure is usually not "bad implementation"; it is "acting with the wrong operating assumptions" or "solving the wrong problem because the environment was never clarified." The skill therefore starts by pulling high-value clues from the user, using those clues to inspect the environment, and only then creating or updating a local contract file such as `AGENT.md` or `CLAUDE.md`.

If both files are absent, prefer creating `AGENT.md` unless the surrounding ecosystem already uses `CLAUDE.md`.
If one exists, extend it instead of introducing a second competing contract unless there is a strong local reason.

## Phase 0: User-Guided Clue Collection

Before reading large amounts of local context or creating a contract file, interact with the user as much as needed to extract the highest-value clues.

Default rule:

- ask for clues before assuming the environment
- prefer a short sequence of focused questions over one broad vague question
- keep the questions tied to execution, not abstract brainstorming

Look for clues such as:

- what outcome actually matters
- what has already been tried
- what is currently blocked
- where the source of truth probably lives
- which files, commands, logs, dashboards, or documents matter
- what constraints or preferences are already known
- what "success" and "done" mean in this environment

If the user already provided enough concrete clues, do not ask redundant questions. Summarize the clues and move on.

## Phase 1: Evidence Collection From Those Clues

Use the user's clues to guide what you inspect next.

Do not scan the environment blindly if the user already pointed to likely sources. Follow the strongest leads first:

- specific files or folders
- named scripts or commands
- known failing traces
- prior experiment logs
- linked docs or external systems

The purpose of this phase is to replace vague user statements with concrete evidence.

## Phase 2: Establish the Local Contract

After the first round of user clues and environment evidence, create or update `AGENT.md` or `CLAUDE.md` with the minimum useful operating contract for the environment.

The contract should be short and concrete. It should usually cover:

- the primary objective
- hard constraints and prohibited actions
- source-of-truth order
- validation expectations
- how to handle uncertainty
- logging or experiment-record expectations
- environment-specific commands or entrypoints if already known

Do not write a manifesto. Write an operational contract.

Suggested structure:

1. `Goal`
2. `Hard Constraints`
3. `Source Priority`
4. `Working Rules`
5. `Validation`
6. `Logging`

Example starter shape:

```md
# AGENT.md

## Goal
Deliver the requested outcome with the smallest reliable change.

## Hard Constraints
- Do not assume undocumented behavior is allowed.
- Do not make destructive changes without explicit evidence they are safe.
- Prefer existing project entrypoints, scripts, and conventions.

## Source Priority
1. User request
2. Existing local docs and config
3. Code and tests
4. External references

## Working Rules
- Summarize the current state before changing strategy.
- Form a narrow root-cause hypothesis before broad exploration.
- Prefer minimal falsifiable experiments over large rewrites.

## Validation
- Run the smallest relevant verification after each meaningful change.

## Logging
- Record decisions, failed paths, and open risks in a durable local note when iteration matters.
```

The contract must reflect what has been learned from the user and local evidence, not generic best practices alone.

## Phase 3: Read the Environment Before Solving It

After the first contract exists, continue inspecting the local environment to replace assumptions with evidence.

Gather only the context needed to move:

- repo structure
- existing docs
- build/test/run entrypoints
- active constraints from config, CI, or toolchain
- prior logs, experiment notes, issue history, or failure traces

Treat local artifacts as primary evidence. Use web research only after the bottleneck is clear or when the information is likely time-sensitive.

## User Interaction Rules

In the initial stage, bias toward more interaction with the user, not less.

Use interaction to reduce ambiguity around:

- objective
- constraints
- prior attempts
- evaluation criteria
- important environment landmarks

Do not ask the user things the workspace can answer cheaply.
Do ask the user things that define intent, hidden constraints, or where to search.

Good early questions:

- which result matters most right now
- what you already tried that failed
- which files or commands are most relevant
- whether there is an existing contract, runbook, or operating convention

Bad early questions:

- generic open-ended "tell me more"
- asking for information already obvious from local files
- abstract design questions before the concrete problem is framed

## Default Iteration Loop

Use this loop when the user asks to continue, improve, debug, explore, validate, or find the next direction.

1. `收集用户线索`
2. `总结现状`
3. `透过现象看本质`
4. `牛角尖检测`
5. `定向研究`
6. `快速冒烟`
7. `记录并复盘`

This is the default house style of the skill.

### 1. 收集用户线索

Start by extracting the most decision-relevant clues from the user.

Required output:

- the user's stated objective
- declared constraints or preferences
- known important files, commands, or systems
- prior failed attempts
- unresolved ambiguities

Then use these clues to drive the next inspection steps.

### 2. 总结现状

Before changing code or strategy, compress the current evidence into a concrete state summary.

Required output:

- current objective
- current best known working state
- newest failures or regressions
- what changed
- what did not change
- repeated failure pattern

Separate facts from interpretation.

### 3. 透过现象看本质

Infer the narrowest plausible root-cause class from the evidence.

Always separate:

- `现象层`: what is visibly failing
- `本质层`: what underlying bottleneck class best explains it

Typical root-cause classes:

- missing environment understanding
- wrong source-of-truth or stale assumption
- interface / contract mismatch
- data or input quality problem
- decision logic coupling
- evaluation mismatch
- optimization path problem
- tooling / infra limitation

Do not propose a large solution before naming the bottleneck class.

### 4. 牛角尖检测

Check whether the current direction has become a local dead end.

Classify explicitly:

- `未进入牛角尖`
- `接近牛角尖`
- `已经进入牛角尖`

Signals of a dead end:

- repeated attempts on the same mechanism are not improving the real target
- only proxy indicators move while the real objective stays flat
- recent work mostly tweaks parameters, wording, thresholds, or nearby details
- the same failure mode remains dominant

If the status is `接近牛角尖` or `已经进入牛角尖`, the next smoke set must include:

- one different bottleneck class
- one different intervention point
- one stop-loss option that abandons the current road

### 5. 定向研究

Research only after the bottleneck class is clear.

Research targets should be narrow and directly tied to the identified bottleneck.

Rules:

- prefer local evidence first
- if external info may have changed, browse instead of relying on memory
- prefer primary sources for technical claims
- map every external method or idea back to the local bottleneck

Do not search broadly for "best practices" when the actual issue is narrower.

### 6. 快速冒烟

Test the smallest faithful change that can falsify the hypothesis quickly.

Required behavior:

- design `3+` candidate smokes when the problem is non-trivial
- make the smokes meaningfully different roads, not parameter variants
- prefer minimal code or process changes
- reuse existing scripts and workflows when possible
- validate against the real objective when feasible, not only a proxy

When a smoke fails, record where it failed:

- setup or environment
- implementation correctness
- proxy evaluation
- transfer to the real objective

When a smoke succeeds, deepen only that direction first.

### 7. 记录并复盘

After each meaningful result:

- record the hypothesis
- record what was changed
- record the observed result
- record the failure mode or signal quality
- record the next decision

If the environment is iterative, keep these notes in a durable local file or directory rather than only in chat.

## Execution Rules

When the user asks to continue or improve, default to execution rather than discussion unless there is a real blocker.

Carry the work through this order:

1. collect user clues
2. inspect the environment from those clues
3. establish or refresh the local contract file
4. summarize the current state
5. infer the bottleneck class
6. run dead-end detection
7. research only the relevant bottleneck if needed
8. run the smallest useful smoke
9. log the result
10. either deepen the winning direction or switch roads

Do not stop at listing ideas if the task is executable inside the workspace.

## Contract File Guidance

Use `AGENT.md` or `CLAUDE.md` as a local alignment layer for future runs.

The file should evolve when you learn:

- hidden constraints
- preferred commands or scripts
- recurring failure modes
- evaluation traps
- repo-specific style or workflow expectations

Keep the contract stable enough to guide future behavior, but update it when evidence proves it incomplete or wrong.

Good contract updates:

- add a newly discovered non-negotiable constraint
- raise the priority of a source that proved authoritative
- ban a repeated dead-end pattern
- codify a verification command that should always run

Bad contract updates:

- copying large amounts of transient output
- encoding a speculative theory as a rule
- adding redundant generic advice

## Hermes-Style Iteration

Follow a Hermes-style progression:

- begin with user interaction to gather clues
- start with a small but explicit operating contract
- observe the environment before forcing a plan
- turn raw symptoms into a narrow hypothesis
- test by minimal falsifiable steps
- log outcomes so future iterations start from a better prior
- let the contract and the plan co-evolve with evidence

The contract is not the final answer. It is the stabilizer that prevents low-quality early moves while the agent is still learning the environment.

## Output Style

When using this skill:

- be concise
- lead with conclusions
- distinguish facts from hypotheses
- state when evidence is insufficient
- prefer concrete next actions over broad frameworks

## Minimal Deliverables

For substantial unfamiliar-environment work, try to leave behind:

- an `AGENT.md` or `CLAUDE.md` contract
- a short record of current bottlenecks
- at least one falsifiable next-step experiment or verification

If the task is small, compress this proportionally rather than forcing ceremony.
