# Tool Design Guide (ACI)

Tool design is the single highest-leverage activity in building capable agents. **Anthropic's SWE-bench team spent more time optimizing tool definitions than the overall agent prompt.** A poorly designed tool will cause more failures than a mediocre prompt.

This document codifies **Agent-Computer Interface** (ACI) best practices.

---

## Core Principles

### 1. Write for a Junior Developer

If a junior engineer on your team would be confused by a tool's description or parameters, the LLM will be too.

- Explain what the tool does, not just its technical name
- Describe each parameter in plain language
- Include valid values, formats, and constraints
- State what happens on success AND failure

### 2. Keep Formats Natural

LLMs perform best with formats they've seen in training data — plain text, markdown, common serialization.

- **Prefer:** plain text, markdown, JSON object with simple types
- **Avoid:** formats requiring pre-counting (diffs with chunk headers), heavy escaping (JSON-embedded code), or obscure serialization

### 3. Poka-Yoke Your Parameters

Design parameters so that mistakes are hard to make.

- Use **absolute paths** instead of relative (Anthropic found relative paths caused errors after the agent navigated)
- Use **enums** instead of free-text for constrained choices
- Use **boolean** instead of string "yes"/"no"
- Make dangerous parameters explicitly named (e.g., `force_delete: boolean` instead of `force: boolean`)

### 4. Give the Model Room to Think

Don't force the model into a format corner where one wrong character breaks everything. Prefer formats with forgiving structure.

---

## Tool Definition Template

Every tool must include:

```yaml
name: tool-name
description: |
  What this tool does, in one sentence.
  Include WHEN to use this tool and WHEN NOT to.
parameters:
  param_name:
    type: string | number | boolean | array | object
    description: What this parameter means. Include format, valid values, constraints.
    required: true | false
    default: <value>  # if optional
  param_name2:
    type: ...
    description: ...
    required: true | false
returns:
  type: ...
  description: What the tool returns. Include error response format.
examples:
  - input: { param: "value" }
    output: { result: "..." }
  - input: { param: "edge-case" }
    output: { error: "..." }
errors:
  - condition: When X happens
    response: { error: "message", code: "ERROR_CODE" }
```

---

## Description Writing Checklist

For every tool, verify:

- [ ] **What** — does the description say what the tool does?
- [ ] **When** — does it say when to use this tool vs others?
- [ ] **When not** — does it say what this tool should NOT be used for?
- [ ] **Input format** — are parameter types, formats, and constraints specified?
- [ ] **Output format** — is the success response format documented?
- [ ] **Error states** — are error responses documented?
- [ ] **Example** — is there at least one example of correct usage?
- [ ] **Boundaries** — is it clear where this tool's responsibility ends and another's begins?

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Vague description: "Searches for things" | Specific: "Searches the codebase for files matching a glob pattern. Use for finding files by name. Not for searching file contents (use Grep instead)." |
| Missing parameter constraints: `limit: number` | Constrained: `limit: number — maximum results to return (1-100, default 20)` |
| No error states documented | Add: "Returns `{ error: "File not found" }` if the path doesn't exist" |
| Multiple tools with the same purpose | Merge them or clearly differentiate in descriptions |
| Tool that does too much | Split into focused single-purpose tools |
| Unclear data format: "Returns data" | Specific: "Returns JSON array of objects with { name: string, size: number, modified: ISO8601 }" |

---

## Testing Tools

Before integrating a tool into an agent:

1. **Unit test the tool implementation** — does it actually work?
2. **Run the tool in isolation** with an LLM — does the model call it correctly?
3. **Test edge cases** — invalid params, empty results, errors
4. **Test confusion** — if two tools are similar, does the model choose the right one?
5. **Iterate on description** — if the model misuses the tool, the description is wrong, not the model

---

## Tool Inventory Best Practices

- **Start with 3–5 tools.** Add more only when needed.
- **Too many tools = decision paralysis.** The model has to evaluate every tool on every turn.
- **Group related tools under clear namespaces** (e.g., `file_read`, `file_write`, `file_delete`).
- **Provide tool search/discovery** if >10 tools — an agent can't hold 20 tool descriptions in context effectively. Use the Tool Search server tool or MCP `defer_loading` to surface only relevant tools per query.

---

## Server Tools vs Client Tools

| | Client Tools | Server Tools (Anthropic) |
|---|---|---|
| Execution | Your code | Anthropic's infrastructure |
| Examples | Custom APIs, database queries, MCP tools | WebSearch, WebFetch, CodeExecution, Advisor, ToolSearch, MCPConnector |
| Cost | Token usage only | Token usage + per-use pricing |
| Best for | Proprietary systems, internal APIs | Common operations, no infrastructure needed |

### Anthropic Server Tools (Complete)

| Tool | Purpose |
|------|---------|
| **Web Search** | Search the web for information beyond the model's knowledge cutoff, with cited sources |
| **Web Fetch** | Retrieve full content of specified web pages and PDFs |
| **Code Execution** | Run Python and bash in a sandboxed container for data analysis |
| **Advisor** | Let a faster executor model consult a higher-intelligence advisor mid-generation |
| **Tool Search** | Discover and load tools on demand from large tool collections (enables agents with dozens/hundreds of tools) |
| **MCP Connector** | Connect to remote MCP servers directly from the API without a separate MCP client |

## MCP Integration

Model Context Protocol (MCP) is the standard for connecting agents to external tools, data sources, and services. When designing agent tools, consider:

- **MCP servers** — pre-built connectors for common services (Jira, GitHub, Slack, databases, filesystems). Search the [MCP registry](https://modelcontextprotocol.io) before building custom tools.
- **MCP Connector** — Anthropic's API natively connects to remote MCP servers via the `mcp_servers` parameter (beta). This eliminates the need for a separate MCP client in simple integrations.
- **Client-side MCP** — when you need local STDIO servers, MCP prompts, or MCP resources, use the SDK helpers (`mcpTools()`, `mcpMessages()`) available in Python, TypeScript, Java, Go, Ruby, and PHP.
- **Tool configuration** — MCP toolsets support allowlisting, denylisting, and per-tool `defer_loading` for large tool collections.
