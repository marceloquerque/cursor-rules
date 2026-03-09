# Cursor Rules

A collection of rules, agents, commands, and skills for [Cursor](https://cursor.com) that I use in my workflows.

## Structure

```
.agents/
└── skills/
    └── pr-manager/              # Create PRs and pull review comments
.cursor/
├── agents/
│   └── code-auditor.md          # Codebase analysis & refactoring specialist
├── rules/
│   ├── convex_rules.mdc
│   ├── decision-making.mdc
│   ├── file-length-modularity.mdc
│   ├── general.mdc
│   ├── research-guidelines.mdc
│   └── tailwind-v4.mdc
└── skills/
    └── conductor-json/          # Generate conductor.json for workspaces
```

## What's Included

### Agents

- **code-auditor** — Expert code auditor that finds duplicate code, unused code, DRY violations, technical debt, and unused dependencies. Ideal after major feature work or before refactoring.

### Commands

The PR workflows now live in `.agents/skills/pr-manager` instead of `.cursor/commands/*`.

### Rules

> **TODO:** I don't use Rules anymore, they have been replaced by a collection of [Skills](https://skills.sh/) and my [AGENTS.md](https://github.com/robinebers/agents.md).

- **convex_rules** — Convex-specific patterns and best practices
- **decision-making** — Structured decision-making guidelines
- **file-length-modularity** — File length and modularity standards
- **general** — General coding conventions
- **research-guidelines** — Research and exploration guidelines
- **tailwind-v4** — Tailwind CSS v4 patterns

### Skills

- **pr-manager** — Creates PRs from the current branch, waits for checks, and fetches and organizes PR review comments.
- **conductor-json** — Generates `conductor.json` files for Conductor workspaces, with templates for Tauri, Next.js, and React projects.

## Usage

### Manual Install

Copy the `.cursor/` and `.agents/` directories into your project root. Cursor will automatically pick up rules and agents from `.cursor/`, and the `pr-manager` skill from `.agents/skills/`.

Good luck!