# Agent Design Guide

An agent is an AI system that uses tools in a loop to accomplish tasks, making independent decisions along the way. This guide covers architecture patterns, design principles, and when to use each approach.

---

## Agent vs Simple LLM Call

| Aspect | Simple LLM Call | Agent |
|--------|-----------------|-------|
| Steps | 1 | Multiple, unknown in advance |
| Tool use | None or static | Dynamic, chosen per step |
| Decision-making | Model responds directly | Model decides next action |
| Duration | Single request-response | Multi-turn, possibly long-running |
| Cost | Low, predictable | Higher, variable |
| Failure modes | Hallucination, bias | Compounding errors, infinite loops |

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

**Best for:** Tasks that cleanly decompose into subtasks. Translating, outline-then-write, generate-then-review.

**Avoid when:** Steps can't be predicted upfront or depend on dynamic conditions.

### 2. Routing

Classify input, route to specialized handler.

```
Input → Classify → Handler A
                 → Handler B
                 → Handler C
```

**Best for:** Complex tasks with distinct categories. Customer support (billing vs tech vs complaints), code review (security vs style vs logic).

**Avoid when:** Categories overlap or classification is unreliable.

### 3. Parallelization

Run multiple LLM calls simultaneously and aggregate.

**Sectioning:** Break task into independent subtasks, run in parallel, merge results.
**Voting:** Same task, multiple attempts — take best or majority result.

**Best for:** Tasks where subtasks don't depend on each other. Multi-perspective review, guardrail + response in parallel.

**Avoid when:** Subtasks share dependencies or ordering matters.

### 4. Orchestrator-Workers

Central orchestrator dynamically breaks down tasks and delegates to specialized workers.

```
Orchestrator → Worker A (file ops)
             → Worker B (web search)
             → Worker C (code exec)
             → Synthesize results
```

**Best for:** Complex tasks where subtasks can't be predicted in advance. Coding agents, multi-source research.

**Avoid when:** Tasks are simple enough for a single agent.

### 5. Evaluator-Optimizer

Generate → Evaluate → Refine loop.

```
Generator → Output → Evaluator → Pass? → Done
                          ↓ Fail
                     Feedback → Generator → ...
```

**Best for:** Tasks with clear evaluation criteria and where iterative refinement helps. Writing, translation, code generation.

**Avoid when:** Evaluation criteria are subjective or can't be automated.

### 6. Autonomous Agent

Full agent loop. Plans, uses tools, recovers from errors, knows when to stop.

**Best for:** Open-ended problems where neither the number of steps nor the strategy is predictable. SWE-bench tasks, computer use, complex research.

**Avoid when:** The task has a known, fixed procedure — use a workflow instead.

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

- Good: "You've audited 10,000+ financial statements. You know that fraud patterns shift quarterly, so you always cross-reference against recent cases. You're skeptical of anomalies that can be explained by seasonal trends."
- Bad: "You are a helpful assistant."

---

## Autonomy Spectrum

| Level | Description | When to Use |
|-------|-------------|-------------|
| **Supervised** | Every action requires human approval | High-risk operations, untested agents |
| **Checkpoint** | Runs freely, pauses at predefined points for human review | Complex tasks with known decision points |
| **Human-in-the-loop** | Runs autonomously but can ask the human questions | When the agent may encounter ambiguity |
| **Fully autonomous** | Runs without intervention, self-recovers from errors | Trusted environments, well-tested agents |

Start at a lower autonomy level and increase as the agent proves reliable.

---

## Memory Strategy

See `guides/memory-patterns.md` for full details. Quick reference:

| Task Duration | Memory Tier |
|---------------|-------------|
| Single request | None — just the prompt |
| Multi-turn session | Session memory (conversation history) |
| Hours/days across sessions | Progress tracking (feature lists, git, status files) |
| Cross-project knowledge | Knowledge base (vector store, RAG) |

---

## Anti-Patterns

- **Over-engineering** — using orchestrator-workers for a task that needs one LLM call
- **Under-specifying** — role/goal/backstory too vague for the agent to make good decisions
- **Tool overload** — giving 20 tools when 3 would do. More tools = more decision errors.
- **No termination condition** — agents that loop forever because they don't know when they're done
- **Silent failures** — tools that return empty or error without the agent noticing
