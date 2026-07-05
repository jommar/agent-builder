# Evaluation & Testing Guide

An agent is only as good as your ability to verify it works. This guide covers how to define success
criteria, test agents rigorously, and build guardrails.

---

## Defining Success Criteria

Every agent and skill must have testable success criteria. Vague goals produce unverifiable agents.

**Bad:**

- "The agent should be helpful"
- "The output should be good"
- "It should work most of the time"

**Good:**

- "The agent correctly classifies support tickets into 5 categories with >90% accuracy on a held-out
  test set of 500 tickets"
- "The data pipeline processes 10,000 rows in under 30 seconds with zero schema violations"
- "The content writer produces copy that passes human review 80%+ on first submission"

### Criteria Types

| Type           | Example                                              | Measures     |
| -------------- | ---------------------------------------------------- | ------------ |
| **Accuracy**   | "95% of SQL queries return correct results"          | Correctness  |
| **Latency**    | "Responds within 3 seconds for 95th percentile"      | Speed        |
| **Cost**       | "Average $0.02 per query"                            | Efficiency   |
| **Robustness** | "Handles malformed input without crashing"           | Reliability  |
| **Coverage**   | "Handles all 12 defined edge cases correctly"        | Completeness |
| **Safety**     | "Never outputs PII or executes destructive commands" | Guardrails   |

---

## Testing Strategy

### Level 1: Unit Testing Tools

Test each tool in isolation before integrating into an agent.

```
Input: valid params → Output: expected result
Input: invalid params → Output: documented error
Input: edge case → Output: correct handling
Input: missing required params → Output: clear error
```

A tool that fails in isolation will fail more confusingly inside an agent loop.

### Level 2: Single-Turn Behavior Testing

Test the agent on individual, well-defined tasks with known correct outputs.

Create a test suite:

```jsonc
[
  {
    "input": "What is the capital of France?",
    "expected_tools": [], // should not need tools for this
    "expected_contains": "Paris",
    "expected_not_contains": ["I don't know", "Let me search"],
  },
  {
    "input": "Search the web for the latest TypeScript release",
    "expected_tools": ["web_search"],
    "expected_contains": ["TypeScript", "version"],
    "max_duration_ms": 15000,
  },
]
```

### Level 3: Multi-Turn Scenario Testing

Test complete workflows that span multiple agent turns.

Example: "A user reports a bug. The agent should: read the error log, search the codebase, propose a
fix, and write a test."

Verify:

- All expected tools were called
- The fix is valid (test passes)
- No unnecessary tools were called
- The agent didn't loop or dead-end

### Level 4: Stress & Edge Case Testing

- **Long conversations** — does the agent handle context buildup?
- **Ambiguous requests** — does it ask clarifying questions or guess wrong?
- **Conflicting instructions** — does it detect and surface conflicts?
- **Tool failures** — does it recover gracefully when a tool returns an error?
- **Rate limits** — does it back off and retry?

### Level 5: Red-Teaming

Apply red-team patterns to probe agent weaknesses:

- Adversarial inputs designed to bypass guardrails
- Prompt injection attempts
- Requests for disallowed operations
- Sensitive data extraction attempts
- Tool misuse patterns (calling the wrong tool, calling tools with dangerous parameters)

For a structured approach to agent evaluation, look up evaluation frameworks and research papers for
the platform selected in Phase 0 during Phase 1 research. Examples include Anthropic's "Demystifying
evals for AI agents" (Jan 2026) and "Quantifying infrastructure noise in agentic coding evals" (Feb
2026), OpenAI's evals framework, and LangSmith for LangChain-based agents.

---

## Agent Containment

As agents grow more capable, containment becomes a critical evaluation dimension. An agent that
works correctly but can escape its sandbox is not production-ready.

**Key containment dimensions to test:**

- **Filesystem boundaries** — can the agent access files outside its working directory?
- **Network boundaries** — can it make unauthorized outbound connections?
- **Tool permission escalation** — can it use a read-only tool to achieve write effects?
- **Cross-session leakage** — does data from one user's session bleed into another's?
- **Prompt extraction** — can an adversarial user extract the agent's system prompt?

See your platform's agent containment documentation for production containment patterns. Look this
up during Phase 1 research. Examples include Anthropic's "How we contain Claude across products" and
equivalent containment documentation for OpenAI, Google Vertex AI, and LangChain.

---

## Evaluator-Optimizer Pattern

For agents that need to improve their own output:

```
1. Agent generates output
2. Evaluator (separate LLM call or rule-based) scores the output
3. If score < threshold → feedback is sent back to the agent
4. Agent regenerates with feedback
5. Repeat until score passes or max iterations reached
```

**When to use:**

- Output quality is subjective but evaluatable (writing, translation, code)
- Clear evaluation criteria can be defined
- Iterative improvement is faster/cheaper than getting it right the first time

**When not to use:**

- Objective, deterministic tasks (math, data transforms)
- Time-sensitive operations where latency matters
- Tasks where the evaluator is as unreliable as the generator

---

## Guardrails Architecture

Guardrails must operate at multiple levels:

### Input Guardrails (before the agent acts)

- Reject obviously malicious or out-of-scope requests
- Sanitize inputs (strip PII, normalize format)
- Validate against schema

### Execution Guardrails (during agent operation)

- Tool allowlists/blocklists
- Permission tiers (read-only, read-write, destructive)
- Rate limiting per tool
- Budget caps (max tokens, max cost, max duration)

### Output Guardrails (before returning to user)

- Filter PII and sensitive data
- Validate output format
- Fact-check critical claims against source
- Flag low-confidence responses

### Design Principle: Fail Closed

When unsure, refuse or escalate. Never guess on safety-critical decisions.

```
if (confidence < threshold || operation_is_dangerous) {
    return "I'm not confident enough to proceed. Let me escalate this."
}
```

For autonomous agents in production, add containment boundaries as a second line of defense — even
if the agent's guardrails fail, the sandbox limits the blast radius. See "Agent Containment" above.

---

## Benchmarking Over Time

Track these metrics across agent versions:

- **Task completion rate** — what % of test cases pass?
- **Tool selection accuracy** — did it pick the right tool?
- **Average turns to completion** — fewer is better (faster, cheaper)
- **Error recovery rate** — when a tool fails, does the agent recover?
- **Hallucination rate** — how often does it fabricate information?
- **Cost per task** — tokens consumed, API calls made

Store results in `test-results/` for trend analysis.

---

## Evaluation Checklist

Before deploying an agent:

- [ ] Success criteria are specific and measurable
- [ ] Tool tests pass in isolation
- [ ] Single-turn test suite has >90% pass rate
- [ ] Multi-turn scenarios work end-to-end
- [ ] Edge cases are handled gracefully (no crashes, no loops)
- [ ] Guardrails block known-dangerous operations
- [ ] Red-team review found no critical bypasses
- [ ] Cost estimates are within budget
- [ ] Performance meets latency requirements
