# Generated Skills

Skill definitions produced by the Agent & Skill Builder workflow live here.

Each file follows the format specified in `CLAUDE.md` Phase 4:

**YAML frontmatter fields:**

- `name` — lowercase-hyphenated identifier
- `description` — one-sentence summary of what the skill does
- `trigger` — one of: `auto`, `keyword:<word>`, `intent:<description>`, `explicit`
- `inputs` — map of parameter name to type and description
- `outputs` — map of field name to type and description
- `tools` — list of tool names the skill requires
- `composes_with` — list of other skill names this one chains with

**Body sections:** Procedure (step-by-step), Edge Cases, Success Criteria
