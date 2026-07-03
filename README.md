# Agent & Skill Builder

A guided workflow for designing and producing production-ready AI agents and skills. The AI assistant reads the playbook, researches the internet, interviews you, reviews the design, and generates a definition file.

## Quick Start

Open this directory in Claude Code or any AI tool that reads `CLAUDE.md`. Tell the AI what you want to build, and it handles the rest.

```
User: "I need a data-pipeline agent that processes CSV uploads"
AI: TRIAGE → RESEARCH → INTERVIEW → DESIGN REVIEW → OUTPUT → output/agents/data-pipeline.md
```

## Structure

```
CLAUDE.md                # Main playbook — the 5-phase meta-agent workflow
guides/
  research-phase.md      # Web research protocol (done first, before any design)
  agent-design.md        # Architecture patterns, autonomy spectrum, anti-patterns
  skill-design.md        # Skill types, triggers, procedures, composability
  tool-design.md         # Agent-Computer Interface (ACI) best practices
  memory-patterns.md     # Session → progress → knowledge tier selection
  evaluation.md          # Testable success criteria, guardrails, red-teaming
output/
  agents/                # Generated agent definitions land here
  skills/                # Generated skill definitions land here
```

## Agent vs Skill

| | Agent | Skill |
|---|---|---|
| Behavior | Autonomous, makes decisions | Follows a fixed procedure |
| Invocation | Given a task, operates independently | Triggered on demand |
| Tool use | Dynamic, chosen per step | Static, known in advance |
| Example | Customer support bot, coding agent | CSV validator, markdown formatter |

When in doubt, the workflow starts with triage to pick the right one.

## How It Works

1. **Triage** — Agent or skill? Quick decision tree.
2. **Research** — Web search for domain knowledge, existing solutions, APIs, and failure modes.
3. **Interview** — Structured questions to nail down role, goal, tools, constraints.
4. **Design Review** — Cross-check against all guides. Flag gaps before generating.
5. **Output** — Produce a definition file ready for use.
