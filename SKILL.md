---
name: project-planning
description: >-
  Plans projects using the Projector methodology: decomposes large goals into well-specified, dependency-ordered tasks. Use when user says "plan this project", "break this down", "create a project plan", "help me plan", or when working with ergo/bd task trackers.
---

## Quick Start

**First time?** Ensure there is a task CLI such as ergo (first choice) or bd (more popular, also works)

**Create a new plan:**
1. Ask user for the goal and constraints
2. Create epics and 3-7 top-level tasks
3. Decompose only when tasks aren't doable

**Work on an existing plan:**
1. Get a ready task from ergo or bd
2. Claim → Do → Attach results → Complete

See methodology below for definitions and edge cases.

---

# Methodology

The Projector methodology is independent of how tasks are stored. This section defines the conceptual framework.

## Core Concepts

### Task

A **task** is a well-defined piece of work with five components:

#### 1. Goal
What is the end state, and how do we verify we reached it?

A goal can be:
- An **artifact** (a document, code, a signed contract)
- **Knowledge** (the answer to a question, a completed analysis)
- A **decision** (which vendor to use, whether to proceed)

Must include verification criteria (tests, checklists, acceptance criteria). Some gates are subjective or probabilistic (e.g. "user approves design")—this is fine if named clearly.

#### 2. Inputs
What do I need to know, and what do I need to have?

- Relevant documents, specs, or prior work
- Results from prerequisite tasks
- Resources (money, compute)

Gather inputs upfront to avoid thrashing during execution. Obtaining any input can be the goal of a prerequisite task.

#### 3. Capabilities
What tools, skills, and authority are required?

- Tools available (software, equipment, APIs)
- Skills relevant to the work
- Authority to make decisions or spend resources
- Access to people, systems, or information

A task requiring capabilities the actor doesn't have isn't doable. Obtaining any capability can be the goal of a prerequisite task.

#### 4. Constraints
What limitations apply?

**Hard constraints** are inviolable:
- Physics, laws, permissions
- Budgets, deadlines
- Technical requirements

**Soft constraints** are preferences:
- Norms, conventions
- Stylistic preferences

Learning or specifying a constraint can be the goal of a prerequisite task.

#### 5. Assumptions (optional)
Key assumptions which, if false, would make this work infeasible or irrelevant. Validating an assumption can be the goal of a prerequisite task.

### Epic

An **epic** is a named group of related tasks. Epics organize work into phases or themes.

- Tasks belong to at most one epic
- Epics can depend on other epics (sequencing phases)
- An epic is complete when all its tasks are done

### Dependencies

Dependencies express sequencing: "A depends on B" means B must complete before A can start.

- **Task → Task**: Fine-grained ordering within or across epics
- **Epic → Epic**: Phase sequencing (all tasks in epic A are blocked until epic B completes)

### States

| State | Meaning |
|-------|---------|
| `todo` | Not started |
| `doing` | In progress (claimed by an actor) |
| `done` | Completed (results attached) |
| `error` | Attempted but failed—needs attention |
| `blocked` | Cannot proceed (explicit block, e.g. waiting on external input) |

A task marked `done` without results is considered **canceled/skipped**.

### Ready and Blocked

A task is **ready** when:
- State is `todo`
- Not claimed by anyone
- All task dependencies are `done`
- Its epic's epic-dependencies are complete

A task is **blocked** when:
- State is `blocked`, OR
- State is `todo` but has unmet dependencies

### Doable

A task is **doable** when:
- It is ready
- All components (goal, inputs, capabilities, constraints) are fully specified
- Inputs and capabilities are actually available
- A competent actor could execute it without further planning
- It is appropriately sized: ~4 hours (human) or ~20 minutes (agent)

**Judging "doable"**: When uncertain, ask: "Could I start this right now and finish without stopping to plan or ask questions?" If yes, it's doable. If no, either specify what's missing or decompose. When genuinely unsure, default to smaller tasks—they're easier to verify as doable.

### Worker Types

| Worker | Meaning |
|--------|---------|
| `any` | Anyone can do this |
| `agent` | AI agent can do this (objective verification, no human judgment) |
| `human` | Requires human judgment or authority |

A task is **doable by an agent** when:
- It is doable
- Worker is `agent` or `any`
- Verification is fully objective (tests pass, files exist, etc.)
- No user decisions required during execution

---

## Operating Model

### Key Activities

1. **Planning**: Decompose goals into tasks via dialogue with the user
2. **Execution**: Do the work, attach results
3. **Upkeep**: Keep the plan consistent as work progresses

### Situation Context

Store project context (Who, What, When, Where, Why) in `situation.md` at project root. Read it when working on any task.

### Planning Algorithm

1. **If no plan exists**: Do "Initial Decomposition", below.

2. **For each `todo` task**, determine if it's doable:
   - Is it fully specified?
   - Are all its components available & known?
   - Is it appropriately sized?
   - Could a competent actor execute it without further planning?

3. **If not doable**, either:
   - **Specify it**: Fill in missing components
   - **Decompose it**: Create prerequisite tasks with dependencies

4. **Recurse** until all ready tasks are doable

### Initial Decomposition

When starting a new project:

1. **Create one epic** for the overall goal
2. **Create 3-7 top-level tasks** covering major milestones or deliverables
3. **Decompose further only when needed** — when a task isn't doable, break it down

Avoid over-fragmentation. A task that's clear and sized appropriately doesn't need children. When uncertain whether to decompose:
- If you can describe the work in one paragraph and it fits the time budget, it's probably fine
- If you find yourself writing "first... then... then..." with multiple distinct steps, decompose

### Information Gathering

To fill in task components, use (in order of preference):
1. Reason from available context
2. Read files in the project
3. Use available tools (search, file operations, CLI tools, scripts, browser)
4. Ask the user when information is missing, judgment is required, or important decisions must be made. If you have a skill for "asking the user questions", invoke that.
5. Create a prerequisite task when defining a component requires substantial work

### Execution Loop

```
□ Select the next ready, doable task
□ Claim it (state → doing)
□ Do the work
□ Attach results
□ Mark complete (state → done)
□ Repeat
```

If execution fails: state → `error`. Retry, reassign, or cancel.

### Interaction Guidelines

**Ask the user when:**
- Critical constraints are unknown
- Decisions require their judgment
- Information cannot be obtained otherwise

**Don't ask when:**
- You can make a reasonable assumption and note it
- Information is obtainable through research
- The question can be deferred to a prerequisite task

---

# Task Backends

Projector uses a task backend to store and manage tasks. **Use ergo** unless bd is already in use. Run `<tool> --help` or `<tool> quickstart` to learn the CLI.

## ergo (preferred)

ergo is a fast, minimal plan tracker with native support for all Projector concepts.

| Projector Concept | ergo Representation |
|-------------------|---------------------|
| Task | task entity |
| Epic | epic entity |
| Spec | Structured markdown in task body |
| Dependencies | First-class (task→task, epic→epic) |
| State | `todo`, `doing`, `done`, `blocked`, `error` |
| Claim | `claimed_by` field (atomic) |
| Worker type | `worker` field: `any`, `agent`, `human` |
| Results | `result.path` + `result.summary` |

Title and body are separate fields. Write clear, action-oriented titles.

## bd (alternative)

bd is a dependency-aware issue tracker. It requires conventions for some Projector concepts.

| Projector Concept | bd Representation |
|-------------------|-------------------|
| Task | Issue |
| Epic | Issue with `--type epic` + `--parent` for containment |
| Spec | Structured markdown in description |
| Dependencies | First-class (A depends on B) |
| State | `open`→todo, `in_progress`→doing, `closed`→done |
| Error state | `blocked` status + `error` label |
| Claim | Atomic claim operation |
| Worker type | Labels: `worker:any`, `worker:agent`, `worker:human` |
| Results | Store file paths in notes |

---

# Appendix: Formal Core

A projector plan is a structure $(T, E, \text{epic}, D_T, D_E)$ where:

```text
T                     finite set of tasks
E                     finite set of epics
epic: T → E ∪ {⊥}     each task belongs to at most one epic (⊥ = no epic)
D_T ⊆ T × T           task dependency edges (acyclic)
D_E ⊆ E × E           epic dependency edges (acyclic)
```

## Task Attributes

```text
state: T → {todo, doing, done, error, blocked}
worker: T → {any, agent, human}
claim: T → Actors ∪ {⊥}
results: T → ℘(FilePath)      set of result file references (may be empty)
spec: T → Spec                 structured specification (goal, inputs, capabilities, constraints, assumptions)
```

## Derived Predicates

```text
# Task t's direct task dependencies
taskDeps(t) = { t' ∈ T | (t, t') ∈ D_T }

# Epic e's direct epic dependencies  
epicDeps(e) = { e' ∈ E | (e, e') ∈ D_E }

# Tasks in an epic
tasksIn(e) = { t ∈ T | epic(t) = e }

# Epic completion: all tasks done (empty epic is trivially complete)
epicComplete(e) ⇔ ∀t ∈ tasksIn(e). state(t) = done

# Task dependency satisfaction
taskDepsMet(t) ⇔ ∀t' ∈ taskDeps(t). state(t') = done

# Epic dependency satisfaction (for a task's epic)
epicDepsMet(t) ⇔ epic(t) = ⊥ ∨ ∀e' ∈ epicDeps(epic(t)). epicComplete(e')

# Ready: can be claimed and worked
ready(t) ⇔ state(t) = todo ∧ claim(t) = ⊥ ∧ taskDepsMet(t) ∧ epicDepsMet(t)

# Blocked: cannot proceed
blocked(t) ⇔ state(t) = blocked ∨ (state(t) = todo ∧ (¬taskDepsMet(t) ∨ ¬epicDepsMet(t)))

# Doable: ready and fully specified with available inputs/capabilities
doable(t) ⇔ ready(t) ∧ Specified(spec(t)) ∧ InputsAvail(t) ∧ CapsAvail(t)

# Doable by agent
doableByAgent(t) ⇔ doable(t) ∧ worker(t) ∈ {any, agent}
```

## Invariants

```text
# Acyclicity
D_T is acyclic (no task depends on itself transitively)
D_E is acyclic (no epic depends on itself transitively)

# No cross-type dependencies
D_T ⊆ T × T and D_E ⊆ E × E (tasks depend on tasks; epics depend on epics)

# Claim requirement
state(t) ∈ {doing, error} ⇒ claim(t) ≠ ⊥

# Claim cleared on terminal states
state(t) ∈ {todo, done} ⇒ claim(t) = ⊥
```

## State Transitions

```text
Claim(t, a)      if ready(t): state(t) := doing; claim(t) := a
Complete(t)      if state(t) = doing: state(t) := done; claim(t) := ⊥
Fail(t)          if state(t) = doing: state(t) := error
Block(t)         if state(t) ∈ {todo, doing}: state(t) := blocked
Unblock(t)       if state(t) = blocked: state(t) := todo
Retry(t, a)      if state(t) = error: state(t) := doing; claim(t) := a
Cancel(t)        if state(t) ∈ {todo, blocked, error}: state(t) := done; claim(t) := ⊥
Reopen(t)        if state(t) = done: state(t) := todo
```

## Backend Requirements

A task backend is **sufficient** for Projector if it can represent:
1. Tasks with state, worker, claim, results, and a body (for spec)
2. Epics as task containers
3. Task→task and epic→epic dependencies (acyclic)
4. Atomic claim operations (safe for parallel agents)
5. The state transitions above
