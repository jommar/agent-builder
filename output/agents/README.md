# Generated Agents

Agent definitions produced by the Agent & Skill Builder workflow live here.

Each file follows the format specified in `CLAUDE.md` Phase 4:

**YAML frontmatter fields:**

- `name` — lowercase-hyphenated identifier
- `description` — one-sentence summary
- `tools` — list of tool names available to the agent
- `tools_description` — map of tool name to short description of what it does and when to use it
- `architecture` — one of: `prompt-chaining`, `routing`, `parallelization`, `orchestrator-workers`,
  `evaluator-optimizer`, `autonomous-agent`
- `memory` — `none`, `session`, or comma-separated combination (e.g.,
  `session, progress, knowledge`)
- `guardrails` — list of constraints the agent must follow

**Body sections:** Role/Backstory, Goal, Tools (with params and edge cases), Operating Protocol,
Constraints, Success Criteria
