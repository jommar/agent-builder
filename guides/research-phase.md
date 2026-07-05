# Research Phase Protocol

Before designing any agent or skill, research the internet to ground it in real-world domain
knowledge. Do not skip this phase — assumptions without research produce brittle agents.

---

## Goals

1. Understand the domain deeply enough to ask informed interview questions
2. Avoid reinventing wheels that already exist
3. Identify tools, APIs, and MCP servers the agent/skill can use
4. Learn from others' failures before making your own

---

## Research Steps

### Step 1: Domain Discovery

Search for the problem domain itself — not AI solutions yet, just the domain.

**Example queries:**

- "how does [process] work in [industry]"
- "[industry] standard workflow for [task]"
- "common challenges in [domain]"
- "[domain] best practices 2026"
- "[domain] compliance requirements"

**Goal:** Understand the terminology, key concepts, standard workflows, and regulatory constraints
before thinking about AI.

### Step 2: Existing Solutions

Search for existing AI agents, tools, or products that solve similar problems.

**Example queries:**

- "AI agent for [task]"
- "[task] automation tool"
- "open source [domain] agent"
- "[tool name] review problems limitations"
- "building an agent for [task] lessons learned"
- "MCP server for [domain]"

**For each solution found, assess:**

- What does it do well?
- What are common complaints or limitations?
- What tools/APIs does it use?
- Is it open source? Can you inspect its approach?
- What model does it use and why?

### Step 3: API & Tool Landscape

Identify the specific technical resources available.

**Example queries:**

- "[service] API documentation"
- "[service] API rate limits pricing"
- "MCP server for [domain]"
- "[functionality] SDK Node.js"
- "[data source] public API"

**For each API/tool found, capture:**

- Base URL and authentication method
- Key endpoints relevant to the task
- Rate limits and pricing
- Data format (JSON, CSV, binary)
- SDK availability (Node.js / Python)

### Step 4: Failure Modes & Risks

Search for what goes wrong. This is where you build robustness.

**Example queries:**

- "[domain] automation mistakes"
- "common bugs in [task] automation"
- "[API name] error handling edge cases"
- "AI agent failure mode [task]"
- "pitfalls of using LLMs for [task]"

**Capture:**

- Common error scenarios
- Edge cases that break naive implementations
- Security or privacy concerns
- Performance bottlenecks
- Cost surprises

### Step 5: Benchmarks & Evaluation

Find ways to measure success objectively.

**Example queries:**

- "[task] accuracy benchmark"
- "evaluating [domain] performance metrics"
- "[task] test dataset"
- "how to measure [outcome]"
- "[domain] KPIs standards"

**Capture:**

- Existing benchmarks or test datasets
- Industry-standard metrics
- Acceptance criteria used by similar tools
- What "good enough" looks like in this domain

### Step 6: Codebase Deep Dive (When Applicable)

If the user provided a codebase path in Triage, analyze it alongside your web research. This grounds
the agent/skill in the real code — conventions, tooling, and structure that dictate what it needs to
know.

**Analysis steps:**

1. **Explore structure** — top-level directories, key folders (`src/`, `tests/`, `docs/`,
   `scripts/`), monorepo vs single-project layout
2. **Identify tech stack** — package manager (`package.json`, `requirements.txt`, `Cargo.toml`,
   `go.mod`), language versions, major frameworks, build tools
3. **Read config files** — linter config (`.eslintrc`, `.prettierrc`, etc.), CI pipelines
   (`.github/workflows/`, `Jenkinsfile`), TypeScript/compiler config, Docker setup
4. **Understand testing** — test framework (Jest, pytest, etc.), test conventions, test file
   locations, coverage expectations
5. **Note conventions** — naming patterns, file organization, import patterns, comment style, error
   handling approach
6. **Map existing tools** — what scripts exist in `package.json`? What CLI tools does the project
   already use? This avoids reinventing or conflicting.

**This analysis feeds into:**

- **Tool selection** — tools the agent/skill needs to interact with the codebase (git, linters, test
  runners, build tools)
- **Architecture pattern** — e.g., orchestrator-workers for multi-service repos, routing for
  projects with distinct modules
- **Guardrails** — project-specific constraints (must use TypeScript strict mode, tests required per
  file, naming conventions)
- **Success criteria** — concrete, codebase-specific metrics (tests pass, linting clean, build
  succeeds)

**When to go deep vs quick:** For a simple skill (formatter, validator), a quick scan (package
manager + top-level structure) is enough. For a complex agent (coding agent, deployment
orchestrator), dig into configs, test patterns, and CI pipelines.

---

## Research Output Format

Produce a structured brief in this format:

```markdown
## Research Brief: <agent/skill name>

### Domain Summary

<2-3 sentences on the problem space, key concepts, and standard practices>

### Existing Solutions

| Solution | Strengths | Weaknesses | Relevant? |
| -------- | --------- | ---------- | --------- |
| <name>   | ...       | ...        | yes/no    |

### Available Tools & APIs

| Tool/API | Purpose | Auth | Rate Limits | Docs URL |
| -------- | ------- | ---- | ----------- | -------- |
| <name>   | ...     | ...  | ...         | <url>    |

### Known Risks & Failure Modes

- <risk 1>
- <risk 2>

### Recommended Complexity Level

- [ ] Single augmented LLM call
- [ ] Workflow (chaining / routing / parallelization)
- [ ] Autonomous agent
- [ ] Multi-agent system

### Codebase Context (if applicable)

- **Path:** `<path>`
- **Tech stack:** <languages, frameworks, package manager>
- **Test framework:** <jest, pytest, etc.>
- **CI/CD:** <platform and pipeline>
- **Key conventions:** <naming, file structure, linting rules>

### Key Research Sources

- <url>
- <url>
```

---

## Duration & Depth

- **Simple skill** (data formatter, validator): 2-3 searches, 3-5 pages fetched
- **Moderate agent** (research assistant, code analyzer): 3-4 searches, 4-5 pages fetched
- **Complex agent** (customer support bot, ops automation): 5+ searches, 6+ pages fetched

Don't over-research — once you have enough to ask informed interview questions and identify the
right complexity level, move on.

---

## Red Flags

During research, watch for:

- **No existing solutions** — either you found a novel problem (rare) or the problem is poorly
  defined. Dig deeper.
- **All solutions are complex** — the domain may require multi-agent. But first verify: can a
  simpler approach work?
- **No available APIs** — the agent may need to operate via browser automation or screen reading
  instead.
- **Heavy regulation** — compliance may require additional guardrails, audit logging,
  human-in-the-loop.
- **Cost-prohibitive APIs** — the task may need local models or caching strategies.

Surface any red flags immediately after the research brief.
