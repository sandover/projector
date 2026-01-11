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

### Sequencing sibling work units (shallow trees)

Projector plans encode dependencies most mechanically by nesting prerequisites as descendants, but deep nesting can be undesirable. In a **shallow** folder (many sibling work units under the same parent), treat the siblings as a small backlog and infer a sensible execution order using:

1. Explicit dependencies in YAML frontmatter (optional): `depends_on: goal-a, goal-b` or `depends_on: none`
2. Implicit dependencies in the spec text (heuristic): references like “(from `goal-some-prereq`)” in `## Inputs`

When selecting the next work unit to start, prefer siblings whose dependencies are already `execution: complete`, even if they are not descendants in the tree.

A work unit is **doable by an agent** when:
- it is doable
- `automatable: yes` is set in `work.md`
- the goal and validation gates are fully objective/mechanical (e.g. tests pass, files exist, lint passes), or any subjective judgment is explicitly delegated to a human child work unit
- it requires no missing user decisions during execution (any such decision is split into a child work unit)
- it can be completed end-to-end by an AI agent with the currently available tools/capabilities in ~20 minutes

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
# depends_on (optional): single-line scalar; use `none` or comma-separated `goal-*` names
depends_on: none
---

## Goal
...and validation gate(s)

## Inputs

## Capabilities

## Constraints

## Assumptions (optional)
```

### results.md

When `execution: complete`, `results.md` must exist, be non-empty, and include (briefly):
- **Summary**: what changed / what was produced
- **Outputs**: file paths, links, or decisions produced
- **Validation**: how the goal was verified (tests run, checklist, acceptance criteria)

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

# Suggest next work units among siblings (heuristic dependencies)
python3 - "$root" <<'PY'
from collections import defaultdict, deque
from dataclasses import dataclass
from pathlib import Path
from typing import Optional
import re
import sys


@dataclass(frozen=True)
class Unit:
    path: Path
    name: str
    parent: Optional[Path]
    execution: str
    depends_on: tuple[str, ...]
    inferred_depends_on: tuple[str, ...]


def read_text(path: Path) -> str:
    return path.read_text(encoding="utf-8")


def parse_frontmatter(text: str) -> dict[str, str]:
    lines = text.splitlines()
    if not lines or lines[0].strip() != "---":
        return {}
    out: dict[str, str] = {}
    for line in lines[1:]:
        if line.strip() == "---":
            break
        if ":" not in line:
            continue
        key, value = line.split(":", 1)
        out[key.strip()] = value.strip()
    return out


def parse_depends_on(value: Optional[str]) -> tuple[str, ...]:
    if not value:
        return ()
    if value == "none":
        return ()
    return tuple(v.strip() for v in value.split(",") if v.strip())


def infer_depends_on(text: str) -> tuple[str, ...]:
    # Heuristic: "(from goal-...)" or "(from `goal-...`)" anywhere in the spec.
    found = re.findall(r"from\\s+`?(goal-[a-z0-9-]+)`?", text)
    # Preserve order + dedupe
    seen: set[str] = set()
    ordered: list[str] = []
    for name in found:
        if name not in seen:
            seen.add(name)
            ordered.append(name)
    return tuple(ordered)


def parent_work_unit(root: Path, dir_path: Path) -> Optional[Path]:
    cur = dir_path.resolve()
    while True:
        if cur == root:
            return None
        cur = cur.parent
        if (cur / "work.md").exists():
            return cur


def topo_sort(nodes: list[str], edges: dict[str, set[str]]) -> list[str]:
    indeg = {n: 0 for n in nodes}
    for src, dsts in edges.items():
        for dst in dsts:
            indeg[dst] += 1
    q = deque([n for n in nodes if indeg[n] == 0])
    out: list[str] = []
    while q:
        n = q.popleft()
        out.append(n)
        for dst in sorted(edges.get(n, set())):
            indeg[dst] -= 1
            if indeg[dst] == 0:
                q.append(dst)
    return out if len(out) == len(nodes) else nodes


root = Path(sys.argv[1]).resolve()
work_units = {p.parent.resolve() for p in root.rglob("work.md")}
units: dict[Path, Unit] = {}

for d in sorted(work_units):
    work_md = d / "work.md"
    text = read_text(work_md)
    fm = parse_frontmatter(text)
    name = d.name
    parent = parent_work_unit(root, d)
    units[d] = Unit(
        path=d,
        name=name,
        parent=parent,
        execution=fm.get("execution", ""),
        depends_on=parse_depends_on(fm.get("depends_on")),
        inferred_depends_on=infer_depends_on(text),
    )

children_by_parent: dict[Optional[Path], list[Unit]] = defaultdict(list)
for unit in units.values():
    children_by_parent[unit.parent].append(unit)

for parent, children in sorted(children_by_parent.items(), key=lambda kv: str(kv[0] or root)):
    sibling_names = {c.name for c in children}
    if len(children) <= 1:
        continue

    edges: dict[str, set[str]] = defaultdict(set)
    deps_by_name: dict[str, set[str]] = {}

    for c in children:
        deps = set(c.depends_on) | set(c.inferred_depends_on)
        deps = {d for d in deps if d in sibling_names and d != c.name}
        deps_by_name[c.name] = deps
        for d in deps:
            edges[d].add(c.name)

    order = topo_sort(sorted(sibling_names), edges)

    print(f"\n# Parent: {parent if parent else root}")
    for name in order:
        u = next(x for x in children if x.name == name)
        deps = sorted(deps_by_name[name])
        print(f"- {name}\t{u.execution}\tdeps={','.join(deps) if deps else 'none'}")

    completed = {c.name for c in children if c.execution == "complete"}
    candidates = []
    for c in children:
        if c.execution != "todo":
            continue
        missing = sorted(d for d in deps_by_name[c.name] if d not in completed)
        if not missing:
            candidates.append(c.name)
    if candidates:
        print("Next candidates:")
        for name in sorted(candidates):
            print(f"- {name}")
PY

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
from typing import Optional
from pathlib import Path

root = Path(sys.argv[1]).resolve()
work_units = {p.parent.resolve() for p in root.rglob("work.md")}

def parent_work_unit(dir_path: Path) -> Optional[Path]:
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
