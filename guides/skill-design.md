# Skill Design Guide

A skill is a specific, repeatable procedure that an AI performs on demand. Unlike agents
(autonomous, looping, decision-making), skills are invoked, execute a known sequence, and return a
result.

---

## Naming Conventions

Skill names follow the same conventions as agents (see `guides/agent-design.md`):

- **Lowercase-hyphenated** — e.g., `csv-validator`, `markdown-formatter`, `cash-flow-forecasting`
- **Verb-noun pattern** is preferred for procedural skills: `image-resizer`, `tone-adjuster`,
  `data-validator`
- **Domain-noun pattern** works for knowledge-domain skills: `legal-clause-analyzer`,
  `code-style-checker`
- **Avoid tool names** — don't name a skill after the API it wraps (use `weather-reporter`, not
  `openweather-wrapper`)

---

## Skill vs Agent

| Aspect          | Skill                                 | Agent                                            |
| --------------- | ------------------------------------- | ------------------------------------------------ |
| Control flow    | Predetermined steps                   | Dynamic decisions                                |
| Invocation      | Triggered (keyword, intent, explicit) | Given a task, operates independently             |
| Duration        | Short, single invocation              | Multi-turn, may be long-running                  |
| Decision-making | None — follows procedure              | Chooses tools, evaluates progress                |
| State           | Stateless or input-only               | Maintains context across turns                   |
| Example         | "Format this CSV as JSON"             | "Research competitor pricing and write a report" |

**Promote a skill to an agent when:** the procedure can't be fully predetermined — the AI needs to
decide which tool to use next based on intermediate results.

---

## Skill Types

### 1. Procedural Skill

A fixed sequence of operations. Transform input → validate → output.

**Examples:**

- CSV validator: parse → check schema → report errors
- Markdown formatter: read → apply rules → write
- Image resizer: load → scale → save

### 2. Tool-Wrapper Skill

Wraps an external API or tool with preset parameters and error handling.

**Examples:**

- Weather lookup: given location → call API → format response
- Database query: given SQL → execute → format results
- Web scraper: given URL → fetch → extract structured data

### 3. Knowledge-Domain Skill

Applies domain expertise to transform or analyze input.

**Examples:**

- Legal clause analyzer: given contract text → identify risky clauses
- Code style checker: given code → check conventions → report violations
- Tone adjuster: given text + target tone → rewrite

---

## Trigger Strategies

How the skill gets invoked:

| Trigger                  | Mechanism                                              | Best For                                              |
| ------------------------ | ------------------------------------------------------ | ----------------------------------------------------- |
| **Auto-detect: keyword** | Scans every user message for trigger words             | Common operations ("format", "validate", "translate") |
| **Auto-detect: intent**  | LLM classifies if the user's request matches the skill | Nuanced requests that need understanding              |
| **Explicit command**     | User types `/skill-name` or names the skill            | Rarely used skills, power-user operations             |
| **Chain-triggered**      | Invoked by another skill as part of a pipeline         | Composable workflows                                  |

**Rule:** Use keyword triggers sparingly — they add latency to every message. Intent-based or
explicit invocation is usually better.

---

## Skill Structure

Every skill definition must include:

### Required

1. **Name** — unique, lowercase-hyphenated identifier
2. **Description** — one sentence the AI uses to decide if this skill applies
3. **Trigger** — how it's invoked
4. **Inputs** — typed parameters with descriptions
5. **Procedure** — step-by-step execution plan
6. **Outputs** — what it returns, typed

### Recommended

7. **Edge cases** — known failure modes and how to handle them
8. **Tools** — external tools/APIs needed
9. **Composes with** — skills this one chains with

---

## Writing Procedures

Procedures should be specific enough that an AI can execute them without guessing.

**Bad:**

```
1. Process the data
2. Return results
```

**Good:**

```
1. Parse input as CSV with headers
2. Validate all rows have the same column count
3. For each column, check type consistency (string, number, date)
4. For date columns, validate ISO 8601 format
5. If errors found, return { valid: false, errors: [{row, col, value, reason}] }
6. If no errors, return { valid: true, rowCount: N, columns: [...] }
```

---

## Composability

Skills should be designed to chain together. A skill's output should be usable as another skill's
input.

**Example chain:**

```
CSV Parser → Data Validator → Report Generator
```

When designing a skill:

- Define the output format explicitly
- Document which skills naturally precede or follow it
- Use consistent data formats (JSON, typed fields)

## Skills vs Agent Skills

This guide covers **standalone skills** — self-contained procedures with defined inputs, outputs,
and triggers. There is also the related concept of **Agent Skills** (as defined by Anthropic, Oct
2025), which are domain-specific capability packages that agents can be equipped with at runtime.

|            | Standalone Skill (this guide)            | Agent Skill (Anthropic, 2025)                   |
| ---------- | ---------------------------------------- | ----------------------------------------------- |
| Scope      | Fixed procedure, single invocation       | Domain expertise package for an agent           |
| Invocation | Triggered by keyword, intent, or command | Loaded into an agent's context at runtime       |
| State      | Stateless or input-only                  | May carry persistent domain knowledge           |
| Example    | "Format this CSV as JSON"                | "You have legal-contract-review skills enabled" |

When building a standalone skill that could also function as an agent capability, consider designing
it with both patterns in mind: a clean input/output interface (for standalone invocation) plus
embeddable domain instructions (for agent integration).

---

## Anti-Patterns

- **Vague procedure** — "analyze the data" instead of specific steps
- **Too many responsibilities** — one skill that validates, transforms, formats, and emails. Split
  it.
- **Missing edge cases** — assumes happy path, breaks on surprising input
- **Wrong trigger** — keyword trigger for a rarely-used skill that adds latency
- **Skill that should be an agent** — procedure requires dynamic decisions → upgrade to agent
