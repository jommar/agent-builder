# Agent & Skill Builder

A guided workflow for designing and producing production-ready AI agents and skills. The AI assistant reads the playbook, researches the internet, interviews you, reviews the design, and generates a definition file.

## Quick Start

Open this directory in Claude Code or any AI tool that reads `CLAUDE.md`. Tell the AI what you want to build, and it guides you through the process.

### Example: Building a Code Review Agent

```
USER: "I need an agent that reviews pull requests for security issues"

AI: TRIAGE: agent — it makes independent decisions, uses tools in a loop,
    and the scope is multi-step.

    RESEARCHING... [searches web for code review best practices, existing
    tools, common vulnerability patterns, available APIs]

    RESEARCH COMPLETE — see brief above. Let's design the agent.

    1. Name? code-security-reviewer
    2. Role? Senior security engineer specializing in PR code review
    3. Goal? Identify security vulnerabilities in PR diffs with
       actionable fix recommendations
    4. Backstory? You've reviewed 5000+ PRs across monorepos and
       microservices. You know OWASP Top 10 cold. You prioritize
       high-severity findings but flag medium ones with context.
    5. Tools needed? GitHub API (read PRs, diffs, comments),
       code search, SAST integration
    6. Architecture? Orchestrator-workers — one agent dispatches
       to specialized file reviewers
    [... more interview questions ...]

    DESIGN REVIEW: PASSED

    OUTPUT GENERATED: output/agents/code-security-reviewer.md
```

The generated definition includes the agent's role, goal, backstory, tools, architecture pattern, memory strategy, guardrails, and operating protocol — everything needed to deploy it.

---

## What You Get

Each agent or skill is produced as a standalone markdown definition file with YAML frontmatter. Example agent output:

```markdown
---
name: code-security-reviewer
description: Reviews pull request diffs for security vulnerabilities
tools:
  - github_api
  - code_search
tools_description:
  github_api: "Read PR diffs, comments, and file metadata"
  code_search: "Search codebase for vulnerable patterns"
architecture: orchestrator-workers
memory: progress
guardrails:
  - Never approve or merge PRs — only comment
  - Flag but don't fix — leave fixes to the author
---

You are a senior security engineer with deep expertise in PR review...

## Goal

Identify security vulnerabilities in pull request diffs and provide
actionable, prioritized fix recommendations.

## Tools

github_api: Read PR details, diffs, and post review comments...
code_search: Search the repository for known vulnerable patterns...

## Operating Protocol

1. Receive PR URL → fetch diff via github_api
2. Classify changed files (auth, data, config, UI, tests)
3. Route each file category to a specialized review pass
...
```

Files land in `output/agents/` or `output/skills/` — drop them into any agent platform that supports markdown-based definitions.

---

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
