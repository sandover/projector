# Projector

A Claude Skill for structured project planning and task execution.

Projector decomposes large goals into well-specified, dependency-ordered tasks using a methodology that works with both humans and AI agents. It integrates with task backends like [ergo](https://github.com/semanticart/ergo) or [bd](https://github.com/anthropics/bd).

## Installation

### Claude Code (Plugin Marketplace)

```
/plugin marketplace add brandonharvey/projector-skill
/plugin install projector@projector-skill
```

### Manual Installation

Add the `skills/` directory contents to your `.claude/skills/` folder:

```bash
# Clone and copy
git clone https://github.com/brandonharvey/projector-skill.git
cp -r projector-skill/skills/project-planning ~/.claude/skills/
```

Or add as a submodule in your project's `.claude/skills/` directory.

## What It Does

When you ask Claude to "plan this project", "break this down", or "help me plan", Projector activates and:

1. **Clarifies the goal** through dialogue with you
2. **Decomposes work** into well-specified tasks with clear goals, inputs, constraints, and verification criteria
3. **Tracks dependencies** so tasks are worked in the right order
4. **Executes tasks** claim → do → attach results → complete
5. **Maintains consistency** as work progresses

### Core Concepts

- **Task**: A unit of work with goal, inputs, capabilities, constraints, and assumptions
- **Epic**: A named group of related tasks
- **Dependencies**: Task→Task and Epic→Epic sequencing
- **States**: `todo` → `doing` → `done` (plus `blocked` and `error`)
- **Worker types**: `any`, `agent`, `human`

### Doable Tasks

A task is "doable" when it's fully specified, appropriately sized (~20min for agents, ~4hr for humans), and a competent actor could execute it without stopping to plan.

## Task Backends

Projector works with any task backend that supports dependencies. Recommended:

- **[ergo](https://github.com/semanticart/ergo)** (preferred) — Fast, minimal, native Projector support
- **[bd](https://github.com/anthropics/bd)** — Dependency-aware issue tracker

## Usage

```
# Start a new project
"Let's plan out building a REST API for user authentication"

# Work on an existing plan  
"What's the next ready task?"
"Do the next task"

# Check status
"Show me the project status"
```

## Contributing

Contributions welcome! Please open an issue or PR.

## License

MIT License — see [LICENSE](LICENSE) for details.
