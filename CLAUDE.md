# Harness — Claude Code Plugin

## What Is This?

Harness is a Claude Code plugin that automatically builds a dedicated team of AI agents for your project. Instead of relying on one general-purpose AI session, Harness analyzes your codebase and creates a set of specialized agents — each with a defined role — that work together on your behalf.

Think of it like hiring a consultant who doesn't just do the work themselves. They study your project, hire and train a team of specialists, write up each person's job description, and set up a system so the work keeps running smoothly on its own.

The result is a custom AI team wired specifically to your project, stored as files in your repo, and automatically available in every future Claude Code session.

---

## What Harness Produces

After running, you'll find these new files in your project:

```
your-project/
├── .claude/
│   ├── agents/      # One file per agent, describing their role and behavior
│   └── skills/      # Instructions each agent follows to do their job
├── _workspace/      # Temporary files used during setup (safe to delete afterward)
└── CLAUDE.md        # Updated with context about your new agent team
```

---

## Requirements

Before installing, set this environment variable in your shell:

```
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Add it to your `.zshrc`, `.bashrc`, or equivalent so it persists across sessions.

---

## Installation

**Option 1 — Via the Claude Code Marketplace (recommended):**

```
/plugin marketplace add revfactory/harness
/plugin install harness@harness
```

**Option 2 — Manual:**

```
cp -r skills/harness ~/.claude/skills/harness
```

---

## How to Use It

Once installed, just tell Claude what you want in plain language:

- "Build a harness for this project"
- "Design an agent team for this codebase"
- "Set up a harness"
- "Harness this project"

Claude will take it from there.

---

## What Happens Behind the Scenes

Harness works in phases. You don't need to manage these — they run automatically.

1. **Audit** — checks whether a harness already exists and decides what to do
2. **Domain Analysis** — reads your project, understands the tech stack and your experience level
3. **Team Architecture Design** — picks the right structure for how agents will work together
4. **Agent Definition Generation** — creates the agent files in `.claude/agents/`
5. **Skill Generation** — creates the instruction files in `.claude/skills/`
6. **Integration and Orchestration** — connects everything and updates your CLAUDE.md
7. **Validation** — verifies the whole team works correctly

---

## Team Structures (Architecture Patterns)

Harness picks one of these based on your project's needs:

- **Pipeline** — tasks flow one after another (A, then B, then C)
- **Fan-out / Fan-in** — tasks run in parallel and results are combined
- **Expert Pool** — different specialists are called depending on the situation
- **Producer-Reviewer** — one agent creates work, another reviews and approves it
- **Supervisor** — a central coordinator assigns tasks to workers dynamically
- **Hierarchical Delegation** — a top-level agent delegates down through layers

---

## Ongoing Maintenance

After the initial setup, you can keep your agent team up to date:

- "Inspect the harness" — review the current setup
- "Audit the harness" — check for problems or gaps
- "Sync agents/skills" — update definitions based on changes to your project

---

## Why Bother?

In A/B testing across 15 software engineering tasks, projects using Harness showed a 60% improvement in output quality and outperformed sessions without Harness in every single trial.

The short version: a team of specialists beats one generalist, and Harness builds that team automatically.

---

## Project Layout (This Repo)

```
harness/
├── .claude-plugin/
│   ├── plugin.json         # Plugin metadata
│   └── marketplace.json    # Marketplace listing
├── skills/
│   └── harness/
│       ├── SKILL.md        # The main instructions Claude follows
│       └── references/     # Detailed guides loaded on demand
├── README.md
├── README_KO.md
├── README_JA.md
└── CHANGELOG.md
```
