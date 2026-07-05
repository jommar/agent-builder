# Agent Design Guide

An agent is an AI system that uses tools in a loop to accomplish tasks, making independent decisions
along the way. This guide covers architecture patterns, design principles, and when to use each
approach.

---

## Naming Conventions

Agent names are the primary identifier used across the platform. Follow these rules:

- **Lowercase-hyphenated** — e.g., `code-security-reviewer`, not `CodeSecurityReviewer` or
  `code_security_reviewer`
- **Descriptive over clever** — the name should tell you what the agent does at a glance:
  `rental-finance-cfo`, `customer-support-router`, `data-pipeline-monitor`
- **Two to four words** — short enough to type, specific enough to be unambiguous
- **Avoid reserved words** — don't use platform-specific keywords, tool names, or common agent
  archetype labels as the sole identifier
- **Check for conflicts** — if multiple agents share the same context, ensure names don't collide or
  cause ambiguity

---

## Agent vs Simple LLM Call

| Aspect          | Simple LLM Call         | Agent                              |
| --------------- | ----------------------- | ---------------------------------- |
| Steps           | 1                       | Multiple, unknown in advance       |
| Tool use        | None or static          | Dynamic, chosen per step           |
| Decision-making | Model responds directly | Model decides next action          |
| Duration        | Single request-response | Multi-turn, possibly long-running  |
| Cost            | Low, predictable        | Higher, variable                   |
| Failure modes   | Hallucination, bias     | Compounding errors, infinite loops |

**Rule:** Start with a simple LLM call. Only upgrade to an agent when you demonstrably need:

- Multi-step reasoning with tool use
- Dynamic decision-making during execution
- Operations across multiple turns that can't be predicted upfront

---

## The Agent Loop

Every agent follows this core loop:

```
1. Receive task + context
2. Think: analyze current state, plan next action
3. Act: select and invoke a tool
4. Observe: receive tool result, evaluate progress
5. Decide: is the task complete?
   - YES → return result
   - NO → return to step 2
```

The quality of this loop depends on:

- **Tool design** — are tools well-documented and hard to misuse?
- **Context management** — does the agent have the right information to decide?
- **Termination criteria** — does it know when to stop?

---

## Architecture Patterns

### 1. Prompt Chaining

Decompose task into fixed sequential steps. Each step's output feeds the next.

```
Step 1 → Gate? → Step 2 → Gate? → Step 3 → Output
```

**Best for:** Tasks that cleanly decompose into subtasks. Translating, outline-then-write,
generate-then-review.

**Avoid when:** Steps can't be predicted upfront or depend on dynamic conditions.

### 2. Routing

Classify input, route to specialized handler.

```
Input → Classify → Handler A
                 → Handler B
                 → Handler C
```

**Best for:** Complex tasks with distinct categories. Customer support (billing vs tech vs
complaints), code review (security vs style vs logic).

**Avoid when:** Categories overlap or classification is unreliable.

### 3. Parallelization

Run multiple LLM calls simultaneously and aggregate.

**Sectioning:** Break task into independent subtasks, run in parallel, merge results. **Voting:**
Same task, multiple attempts — take best or majority result.

**Best for:** Tasks where subtasks don't depend on each other. Multi-perspective review, guardrail +
response in parallel.

**Avoid when:** Subtasks share dependencies or ordering matters.

### 4. Orchestrator-Workers

Central orchestrator dynamically breaks down tasks and delegates to specialized workers.

```
Orchestrator → Worker A (file ops)
             → Worker B (web search)
             → Worker C (code exec)
             → Synthesize results
```

**Best for:** Complex tasks where subtasks can't be predicted in advance. Coding agents,
multi-source research.

**Avoid when:** Tasks are simple enough for a single agent.

### 5. Evaluator-Optimizer

Generate → Evaluate → Refine loop.

```
Generator → Output → Evaluator → Pass? → Done
                          ↓ Fail
                     Feedback → Generator → ...
```

**Best for:** Tasks with clear evaluation criteria and where iterative refinement helps. Writing,
translation, code generation.

**Avoid when:** Evaluation criteria are subjective or can't be automated.

### 6. Autonomous Agent

Full agent loop. Plans, uses tools, recovers from errors, knows when to stop.

**Best for:** Open-ended problems where neither the number of steps nor the strategy is predictable.
SWE-bench tasks, computer use, complex research.

**Avoid when:** The task has a known, fixed procedure — use a workflow instead.

**Note on pattern selection:** The `architecture` field in agent definitions accepts exactly one
pattern from the six catalogued above. Choose the dominant pattern — the one that best describes how
the agent dispatches work. Agents often exhibit traits of multiple patterns in practice, but pick
the primary one.

---

## Agent Definition Triad

Every agent needs three things:

### Role

Defines **what** the agent is. Think job title + specialization.

- Good: "Senior Data Analyst specializing in financial fraud detection"
- Bad: "Helper"
- Good: "Healthcare triage nurse with 15 years of ER experience"
- Bad: "Medical assistant"

### Goal

Defines **why** the agent acts. The objective that drives every decision.

- Good: "Identify fraudulent transactions with 95%+ accuracy while minimizing false positives"
- Bad: "Find fraud"
- Good: "Resolve customer issues in one interaction by diagnosing root causes"
- Bad: "Help customers"

### Backstory

Defines **how** the agent thinks. Context, personality, methodology, constraints.

- Good: "You've audited 10,000+ financial statements. You know that fraud patterns shift quarterly,
  so you always cross-reference against recent cases. You're skeptical of anomalies that can be
  explained by seasonal trends."
- Bad: "You are a helpful assistant."

---

## Autonomy Spectrum

| Level                 | Description                                               | When to Use                              |
| --------------------- | --------------------------------------------------------- | ---------------------------------------- |
| **Supervised**        | Every action requires human approval                      | High-risk operations, untested agents    |
| **Checkpoint**        | Runs freely, pauses at predefined points for human review | Complex tasks with known decision points |
| **Human-in-the-loop** | Runs autonomously but can ask the human questions         | When the agent may encounter ambiguity   |
| **Fully autonomous**  | Runs without intervention, self-recovers from errors      | Trusted environments, well-tested agents |

Start at a lower autonomy level and increase as the agent proves reliable.

---

## Memory Strategy

See `guides/memory-patterns.md` for full details. Quick reference:

| Task Duration              | Memory Tier                                          |
| -------------------------- | ---------------------------------------------------- |
| Single request             | None — just the prompt                               |
| Multi-turn session         | Session memory (conversation history)                |
| Hours/days across sessions | Progress tracking (feature lists, git, status files) |
| Cross-project knowledge    | Knowledge base (vector store, RAG)                   |

---

## Anti-Patterns

- **Over-engineering** — using orchestrator-workers for a task that needs one LLM call
- **Under-specifying** — role/goal/backstory too vague for the agent to make good decisions
- **Tool overload** — giving 20 tools when 3 would do. More tools = more decision errors.
- **No termination condition** — agents that loop forever because they don't know when they're done
- **Silent failures** — tools that return empty or error without the agent noticing

---

## Integrations Design

When an agent connects to external systems (APIs, databases, MCP servers), design the integration
layer with these principles:

### API Authentication

- **Use environment variables** for secrets — never hardcode API keys, tokens, or credentials in
  agent definitions
- **Document the auth method** — OAuth 2.0, API key header, basic auth, or MCP-managed credentials
- **Handle token expiry** — design for refresh flows; the agent should know when to re-authenticate

### Rate Limits & Backoff

- **Identify rate limits** during research — document maximum calls per second/minute/day per API
- **Implement exponential backoff** — the agent should retry with increasing delays (1s, 2s, 4s,
  8s...) on 429 responses
- **Queue or batch** requests when possible to stay under limits
- **Fail gracefully** — if an API is down or rate-limited, return a clear error with retry timing,
  not a silent failure

### Data Format Bridges

- **Map external schemas** to the agent's internal data model explicitly — don't leak API-specific
  field names into the agent's reasoning
- **Validate at the boundary** — check that API responses match expected schemas before the agent
  processes them
- **Handle versioning** — APIs change. Document which API version the agent targets and what fields
  are critical vs. optional

### Fallback Patterns

- **Degrade, don't crash** — if a secondary data source is unavailable, continue with reduced
  capability rather than failing the entire task
- **Stale data tolerance** — define how old cached data can be before the agent must refuse rather
  than use it
- **Human escalation path** — when integrations fail in a way the agent can't resolve, surface to
  the human with context on what broke and what's needed

### When to Document

Capture integration details in two places:

1. **Agent definition** — list integrations and their purpose at a high level (see interview
   question 11)
2. **Tool definitions** — each tool that wraps an external API must document auth requirements, rate
   limits, and error states (see `guides/tool-design.md`)

---

## Multi-Agent Systems

When a single agent can't handle the task, consider splitting across multiple agents:

```
Orchestrator → Agent A (specialized domain) → Synthesize
             → Agent B (specialized domain) →
             → Agent C (specialized domain) →
```

**When to use:**

- Tasks with distinctly different domains of expertise (e.g., legal review + code audit + UX
  analysis)
- Workloads that benefit from parallel independent agents voting or cross-checking
- Systems where no single prompt can cover all required expertise

**Key design rules:**

- **Clear handoffs** — each agent must know what it owns and what it passes downstream
- **Shared state** — use structured formats (JSON with schema), not free text (see
  `guides/memory-patterns.md`)
- **Identity tracking** — record which agent wrote what and when
- **Single-writer principle** — avoid two agents writing to the same state simultaneously

**Avoid when:** A single agent with well-designed tools and memory can handle the scope. Multi-agent
systems compound debugging difficulty.

---

## Context Engineering

Agent performance depends as much on context design as on prompt engineering. Context engineering is
the practice of designing what information an agent sees at each decision point — prompts, tool
results, memory state, and conversation history.

**Key principles:**

- **Minimize noise** — every token in the agent's context window competes for attention. Remove
  irrelevant tool results, stale conversation, and redundant information.
- **Structure for scanning** — LLMs process context sequentially. Put critical instructions first,
  tool results with clear headers, and use consistent formatting. Models can miss instructions
  buried in the middle of long contexts.
- **Progressive disclosure** — don't dump everything at once. Start with the task + key constraints,
  then reveal details as the agent investigates.
- **State summaries** — for long-running agents, periodically summarize progress and compress older
  context to prevent context window overflow.

See also: your platform's context engineering documentation (look this up during Phase 1 research).
Anthropic's "Effective context engineering for AI agents" (Sep 2025) is an example of
platform-specific guidance in this area.

---

## Tool Management at Scale

When agents need access to many tools (10+), avoid loading all tool definitions into every prompt:

| Strategy                   | How It Works                                                                   | Best For                                      |
| -------------------------- | ------------------------------------------------------------------------------ | --------------------------------------------- |
| **MCP `defer_loading`**    | Tool descriptions aren't sent to the model until explicitly loaded             | MCP servers with many rarely-used tools       |
| **Tool Search**            | A dedicated server tool discovers and loads tools on demand based on the query | Agents with 20+ tools across multiple domains |
| **Namespace grouping**     | Group related tools into named sets, loaded together                           | Moderate tool collections (10-30)             |
| **Dynamic tool discovery** | Agent uses a search/index tool to find the right tool before invoking it       | Unbounded tool collections                    |

The key insight: **more tools degrade decision quality.** Every additional tool description competes
for the model's attention. Invest in tool discovery before adding more tools.
