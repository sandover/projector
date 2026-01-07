---
name: projector
description: System for planning a project by decomposing a large goal into a tree of well-specified work units; and then doing the work. Use when asked to plan a project, update an existing projector plan, or carry out work in a projector plan.
---

# Projector

Plan and run any type of project by recursively decomposing goals into tractable work units.

## When to Use

Use this skill when the user asks to:
- CREATE a project or accomplish a goal that is too large to be tackled in one shot
- READ, inspect, understand, analyze an existing projector plan
- UPDATE, extend, or modify an existing projector plan
- WORK within an existing projector plan (or tells you to do so)

## Core Concepts

### Work unit

A **work unit** is a well-defined piece of work with five components (the fifth is optional):

#### 1. Goal: What is the end state + how to verify we reached it. 

A goal can be:
- An artifact (a document, a piece of code, a signed contract)
- A decision (which vendor to use, whether to proceed)
- Knowledge (the answer to a question, a completed analysis)
- An integration (combining child work unit outputs into a coherent whole)

Must include verification criteria as part of the goal (e.g. tests, checklists, acceptance criteria). Some verification gates are subjective, social, or even probabilistic (e.g. winning an election), and this is fine, as long as we name it clearly.

#### 2.  Inputs: The most relevant information and resource inputs to the work

What do I need to know, and what do I need to have?

- Relevant documents, specs, or prior work
- Outputs from completed child work units
- Resources from the world such as money, compute budget, floor space, transportation, etc.

For knowledge inputs, try to set up conditions to avoid thrashing — repeatedly consulting external sources during execution. Gather what's needed upfront.

#### 3. Capabilities: Necessary and sufficient tools, skills, and authority to do the work

The action space. What can I do?

- Tools available (software, equipment, APIs)
- Skills relevant to the work
- Authority to make decisions or spend resources
- Access to people, systems, or information

Capabilities determine what's feasible. A work unit requiring capabilities the actor doesn't have isn't doable.

#### 4. Constraints: The most relevant limitations on the work

What can't I do?

**Hard constraints** are inviolable:
- Physics, laws, permissions
- Budgets, deadlines
- Technical requirements

**Soft constraints** are preferences:
- Norms, conventions
- Stylistic preferences
- Nice-to-haves

####  5. Assumptions (optional): Key assumptions we are making which, if they are not true, would make this work infeasible or irrelevant.

### Doable work

A work unit is **doable** when:
- its `execution` is `todo`
- it has no open descendant work units (i.e. nothing unfinished under it)
- all components are fully specified
- inputs and capabilities are available
- a competent actor could execute the work unit without further planning, in less than 4 hrs (human) or 20 min (AI agent)

Example: My *goal* is to make hot cocoa for my son (to be completed when I hand it to him). I have to *know* where the relevant ingredients are in my kitchen and they must *be there*; I have to be *capable* of operating the relevant kitchen appliances; and I have time to make the cocoa. 

It's important to identify work units that are doable by an AI agent. Those can later be queued up for automation, leaving the rest to be tackled by people.

### Key Activities

- **Interactive planning** is done as a dialogue with the user, at the beginning of the project or any time during the project. Use this phase to ask the user high-information-gain questions that will help shape the plan according to their intentions.
   - Planning is also a type of work. If defining a work unit will take so much effort that it would block your realtime interaction with the user, define a child work unit instead of doing it live. 
- **Human work** is when a person makes progress on a work unit. 
- **AI work** is when an agent makes progress on a work unit.
- **Plan upkeep** must be done whenever there is a change to the plan or some progress is made on a work unit. If you are doing the work and generating the results, you should immediately do relevant plan upkeep. Always preserve the invariants defined in the Formal Core.

## Plan Tree Representation

Each work unit is a folder containing a `work.md` specification file and zero or more child work units. The folder name starts with the word "goal-" and phrases the goal in 4-8 words (hyphen-separated). 

```
goal-do-a-thing/
├── work.md          # required: work unit definition. Keep top matter up to date!
├── situation.md     # optional: local context (inherits from ancestors)
├── results.md       # required when complete: non-empty work output or a summary
├── [other output]   # optional: images, PDFs, other results
└── [children]/      # child work units (subfolders)
```

### work.md

```yaml
---
# id: 8-char hex, immutable after creation
id: a1b2c3d4
# execution: todo | active | complete | error
execution: todo
# actor: required when execution is active|complete|error; use `unassigned` otherwise
actor: unassigned
# automatable: yes | no (accomplishable by claude code or codex?)
automatable: no
---

## Goal
...and validation gate(s)

## Inputs

## Capabilities

## Constraints

## Assumptions (optional)
```

### situation.md

Captures real-world context that flows down the tree and contextualizes all work on all goals:

```markdown
# Situation

## Who
Who is doing this? What are their capabilities?

## What
What is the broader context?

## When
What are the relevant timeframes?

## Where
Physical, market, or organizational context?

## Why
What is the motivation? What does success enable?
```

Root situation flows to all descendants. Add local `situation.md` only to override or extend. For example, a CEO may be responsible for the overall project, but part of it might be delegated to someone more junior, who has less authority.

## Operating Model

### Entry Points

Given a goal or an existing plan tree, we will create, update, extend, or make progress against a plan.

### Definitions

- **Root directory**: the folder you point tools at (the `root=...` variable in Utilities)
- **Work unit**: any directory under `root` that contains a `work.md`
- **Parent**: the nearest ancestor directory (toward `root`) that also contains a `work.md`
- **Root work unit**: the unique work unit under `root` with no parent work unit
- **Child** work unit: a subdirectory containing a `work.md` (immediate child in the plan tree)
- **Descendant** work unit: any nested child work unit at any depth
- **Open** work unit: `execution` is `todo`, `active`, or `error`
- **Closed** work unit: `execution` is `complete`
- **Unblocked** work unit: `execution: todo` AND there are no open descendant work units
- **Doable** work unit: as defined above; in particular it is `execution: todo`, unblocked, and fully specified with available inputs and capabilities

### Formal Core

```text
A projector plan is a rooted tree with labeled nodes.

V                    finite set of work units (nodes)
parent: V\{r} → V    each non-root work unit has exactly one parent (r is the unique root work unit)

id: V → H             H = {0-9a-f}^8, and id is injective (unique IDs)
execution: V → {todo, active, complete, error}
actor: V → Actors ∪ {unassigned}
automatable: V → {yes, no}

children(v)           = { u ∈ V | parent(u)=v }
desc(v)               = transitive closure of children(v) (all descendants of v)
open(v)               ⇔ execution(v) ∈ {todo, active, error}
closed(v)             ⇔ execution(v) = complete
unblocked(v)          ⇔ execution(v)=todo ∧ ∀u∈desc(v). closed(u)
doable(v)             ⇔ unblocked(v) ∧ Specified(v) ∧ InputsAvail(v) ∧ CapsAvail(v)

Invariants:
  execution(v)=complete ⇒ (∀c∈children(v). execution(c)=complete) ∧ ResultsPresent(v)
  execution(v)∈{active,complete,error} ⇒ actor(v)≠unassigned

Transitions:
  Start(v,a)           if doable(v): execution(v):=active; actor(v):=a
  Complete(v)          if execution(v)=active ∧ ResultsPresent(v): execution(v):=complete
  Fail(v)              if execution(v)=active: execution(v):=error
  Decompose(v, kids)   add new child nodes under v (fresh ids); ensure execution(v)=todo
```

Filesystem mapping:
- `V` are directories under `root` containing `work.md`
- `parent` is the nearest ancestor directory (toward `root`) containing `work.md`
- `id/execution/actor/automatable` come from YAML frontmatter in `work.md`
- `ResultsPresent(v)` means `results.md` exists and is non-empty

### Execution (Doing Work)

Execution is gated: only **doable** work units can be started. Non-leaf work units are treated as **integration steps**: validate child outputs, combine them, and record the integrated result. If integration reveals missing work, decompose by creating new child work units and set the current work unit back to `execution: todo` (it is no longer unblocked until those children are complete).

Execution loop:
1. Select a doable work unit
2. Set it to `execution: active` and set `actor`
3. Do the work and write/refresh `results.md`
4. Set it to `execution: complete`

### Planning (Shaping the Tree)

1. **If no tree exists**: Create root work unit

2. **For each `execution: todo` work unit**, determine if it's doable:
   - Is it fully specified? (All components meaningfully defined)
   - Is it realistic? (All inputs actually available)
   - If it has open child work units, recurse into them (the parent is not unblocked until they are complete)
   - Could a competent actor execute it without further planning?
   - Is it appropriately sized? (Agent: ~20 min, Human: ~4 hrs)

3. **If not**, either:
   - **Specify it**: Fill in missing components via information gathering, below
   - **Decompose it**: Create child work units that collectively enable the goal

4. **Recurse** until all unblocked work units are doable

### Information Gathering

To fill in work unit components, use (in order of preference):
1. Reason directly from inputs and situation
2. Read files in the plan tree and project
3. Use available tools (web search, file operations, scripts)
4. Ask the user targeted questions when information is genuinely missing or judgment is needed
5. Create a child work unit when a component would require substantial work to define. 

### Interaction

Ask the user when:
- Critical constraints are unknown (budget, timeline, hard requirements)
- Decisions require their judgment or preferences
- Information cannot be obtained through other means

Don't ask when:
- You can make a reasonable assumption and note it
- The information is obtainable through research
- The question can be deferred to a child work unit

### Plan Upkeep

When working with an existing plan tree:

1. Establish correctness
    - Run the validation recipes below to check invariants
    - Fix any violations
    - Verify completed work units have a non-empty `results.md` and integrate work upwards in the tree as needed
    - Check that `todo` work units remain doable given new information
2. Ask yourself: have we triggered the need for more planning? If so, use the planning algorithm


## Utilities

```bash
# Grep-friendly frontmatter contract (required for the recipes below):
# - `id`, `execution`, `automatable`, `actor` are single-line `key: value` scalars
# - keys appear in the YAML frontmatter (between `---` lines), near the top of the file
# - do not put inline comments on these key lines; put comments on their own `# ...` lines
# - `execution` is one of: todo | active | complete | error
# - `automatable` is one of: yes | no
# - `actor` is required when execution is active|complete|error; use `actor: unassigned` otherwise

root=./my-project

# List work units (directories containing work.md)
find "$root" -name work.md -print | sed 's|/work\\.md$||'

# List work units by execution state
rg -l --glob 'work.md' '^execution: todo$' "$root" | sed 's|/work\\.md$||'
rg -l --glob 'work.md' '^execution: active$' "$root" | sed 's|/work\\.md$||'
rg -l --glob 'work.md' '^execution: complete$' "$root" | sed 's|/work\\.md$||'
rg -l --glob 'work.md' '^execution: error$' "$root" | sed 's|/work\\.md$||'

# Find next unblocked work units (todo + no open descendant work units)
rg -l --glob 'work.md' '^execution: todo$' "$root" |
  while read -r work_md; do
    d="${work_md%/work.md}"
    if ! find "$d" -mindepth 2 -name work.md -exec rg -l '^execution: (todo|active|error)$' {} + | rg -q .; then
      echo "$d"
    fi
  done

# Find next unblocked work units for an agent (todo + automatable: yes + no open descendant work units)
rg -l --glob 'work.md' '^execution: todo$' "$root" |
  while read -r work_md; do
    d="${work_md%/work.md}"
    if rg -q '^automatable: yes$' "$work_md" && ! find "$d" -mindepth 2 -name work.md -exec rg -l '^execution: (todo|active|error)$' {} + | rg -q .; then
      echo "$d"
    fi
  done

# Validate IDs: missing/invalid 8-char hex
find "$root" -name work.md -print0 |
  xargs -0 -n1 awk '
    function bad(msg) { print msg ": " FILENAME }
    /^---$/ {
      if (!yaml_started) { yaml_started = 1; next }
      yaml_done = 1
      exit
    }
    yaml_started && !yaml_done && $1=="id:" { id = $2 }
    END {
      if (id=="") { bad("missing id"); exit }
      if (id !~ /^[0-9a-f]{8}$/) { print "invalid id: " FILENAME "\t" id }
    }'

# Validate IDs: duplicates
find "$root" -name work.md -print0 |
  xargs -0 -n1 awk '
    /^---$/ {
      if (!yaml_started) { yaml_started = 1; next }
      yaml_done = 1
      exit
    }
    yaml_started && !yaml_done && $1=="id:" { print $2 "\t" FILENAME; exit }' |
  sort |
  awk 'prev==$1 { print "duplicate id: " $1 "\n  " prevfile "\n  " $2 } { prev=$1; prevfile=$2 }'

# Validate: actor present when execution active|complete|error
find "$root" -name work.md -print0 |
  xargs -0 -n1 awk '
    /^---$/ {
      if (!yaml_started) { yaml_started = 1; next }
      yaml_done = 1
      exit
    }
    yaml_started && !yaml_done && $1=="execution:" { ex = $2 }
    yaml_started && !yaml_done && $1=="actor:" { actor = $2 }
    END {
      if (ex ~ /^(active|complete|error)$/ && (actor=="" || actor=="unassigned")) {
        print "missing actor: " FILENAME "\t" "execution=" ex
      }
    }'

# Validate: execution present + valid enum
find "$root" -name work.md -print0 |
  xargs -0 -n1 awk '
    function bad(msg) { print msg ": " FILENAME }
    /^---$/ {
      if (!yaml_started) { yaml_started = 1; next }
      yaml_done = 1
      exit
    }
    yaml_started && !yaml_done && $1=="execution:" { ex = $2 }
    END {
      if (ex=="") { bad("missing execution"); exit }
      if (ex !~ /^(todo|active|complete|error)$/) { print "invalid execution: " FILENAME "\t" ex }
    }'

# Validate: automatable present + valid enum
find "$root" -name work.md -print0 |
  xargs -0 -n1 awk '
    function bad(msg) { print msg ": " FILENAME }
    /^---$/ {
      if (!yaml_started) { yaml_started = 1; next }
      yaml_done = 1
      exit
    }
    yaml_started && !yaml_done && $1=="automatable:" { a = $2 }
    END {
      if (a=="") { bad("missing automatable"); exit }
      if (a !~ /^(yes|no)$/) { print "invalid automatable: " FILENAME "\t" a }
    }'

# Validate: root work unit is unique (exactly one work unit with no parent work unit)
python3 - "$root" <<'PY'
import os
import sys
from pathlib import Path

root = Path(sys.argv[1]).resolve()
work_units = {p.parent.resolve() for p in root.rglob("work.md")}

def parent_work_unit(dir_path: Path) -> Path | None:
    cur = dir_path.resolve()
    while True:
        if cur == root:
            return None
        cur = cur.parent
        if (cur / "work.md").exists():
            return cur

roots = [d for d in work_units if parent_work_unit(d) is None]
if len(roots) != 1:
    print(f"invalid root count: {len(roots)}")
    for d in sorted(roots):
        print(f"  {d}")
    sys.exit(1)
PY

# Validate: completed work units have no open descendants
rg -l --glob 'work.md' '^execution: complete$' "$root" |
  while read -r work_md; do
    d="${work_md%/work.md}"
    if find "$d" -mindepth 2 -name work.md -exec rg -l '^execution: (todo|active|error)$' {} + | rg -q .; then
      echo "complete has open descendants: $d"
    fi
  done

# Validate: completed work units have non-empty results.md
rg -l --glob 'work.md' '^execution: complete$' "$root" |
  while read -r work_md; do
    d="${work_md%/work.md}"
    test -s "$d/results.md" || echo "missing/empty results.md: $d"
  done
```
