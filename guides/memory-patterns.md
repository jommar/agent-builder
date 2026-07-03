# Memory Patterns Guide

Agents that forget everything between sessions are crippled. This guide covers when and how to give agents memory.

---

## Three Memory Tiers

### Tier 1: Session Memory (Short-Term)

**What:** Conversation history within a single agent session.

**Storage:** In-memory message array. Lost when session ends.

**Best for:**
- Multi-turn conversations where context from earlier turns matters
- Clarifying follow-up questions
- Tasks completed within a single session

**Not for:**
- Tasks spanning multiple sessions
- Knowledge that should persist across users
- Very long conversations (use summarization or compaction)

**Implementation:** Simply maintain the message list. For long sessions, summarize older messages to stay within context limits.

### Tier 2: Progress Tracking (Medium-Term)

**What:** State that persists across sessions. The agent knows what happened last time.

**Storage:** Files on disk — progress logs, feature lists, git history.

**Best for:**
- Multi-session tasks (hours to days)
- Complex projects with many subtasks
- Handoff between different agent instances

**Not for:**
- Permanent knowledge storage
- Cross-project information

**Key patterns (from Anthropic's long-running agent research):**

1. **Feature/Progress list** — a structured file (JSON or markdown checklist) tracking what's done and what remains
2. **Progress log** — chronological entries: "Session #3: implemented auth module, tests passing"
3. **Git commits** — descriptive commit messages serve as implicit progress log
4. **Init/teardown scripts** — `init.sh` to set up the environment, `status.sh` for quick state check

**Example `claude-progress.txt`:**
```
Session 1 (2025-07-01): Initial setup — project scaffolded, dependencies installed, init.sh created
Session 2 (2025-07-01): Implemented task list CRUD API. 12/45 features passing.
Session 3 (2025-07-02): Added task filtering and sorting. 18/45 features passing.
Session 4 (2025-07-02): CURRENT — working on user authentication. Auth module partially done, DB schema ready, login endpoint WIP.
```

### Tier 3: Knowledge Base (Long-Term)

**What:** Permanent knowledge that persists across all sessions and projects.

**Storage:** Vector database (pgvector, Chroma, Pinecone), traditional database, or documented reference files.

**Best for:**
- Company knowledge: policies, procedures, design decisions
- Domain expertise: industry standards, technical references
- User preferences: past decisions, style guides, known constraints
- Learned patterns: solutions that worked well before

**Not for:**
- Session-specific state (use Tier 2)
- Raw conversation history (use Tier 1)
- Frequently changing data (cache instead)

**Implementation options:**
- **RAG (Retrieval-Augmented Generation):** Embed knowledge, search by semantic similarity at query time
- **Reference files:** CURATED.md, project docs, API specs — read when relevant
- **Structured storage:** Database with explicit schemas for known entity types

---

## Choosing a Memory Strategy

```
Task duration < 1 session?
  → Tier 1 (session memory) is enough

Task spans multiple sessions but is finite?
  → Tier 1 + Tier 2 (progress tracking)

Agent needs to learn and improve over time?
  → Tier 1 + Tier 2 + Tier 3 (knowledge base)

Multiple agents sharing context?
  → Tier 2 or Tier 3 with explicit read/write protocols
```

---

## Shared Memory for Multi-Agent Systems

When multiple agents collaborate:

1. **Write to shared state explicitly** — don't rely on implicit context
2. **Use a structured format** — JSON with schema, not free text
3. **Include agent identity** — who wrote what and when
4. **Version state files** — git or timestamp-based, for recovery
5. **Define read/write protocols** — which agent owns which section of state

---

## Memory Pitfalls

| Pitfall | Solution |
|---------|----------|
| Memory never cleaned → context overload | Summarize old entries, expire outdated information |
| Agent trusts memory blindly → propagates errors | Verify critical facts from source, not memory |
| Memory grows unbounded → cost/speed issues | Tiered storage: hot (session) → warm (progress files) → cold (knowledge base) |
| Shared state conflicts → two agents write simultaneously | Lock files, append-only logs, or single-writer design |
| Memory contains sensitive data → security risk | Encrypt at rest, access control per agent, never store raw secrets |

---

## State Serialization Formats

| Format | Best For | Avoid For |
|--------|----------|-----------|
| **JSON** | Structured task lists, feature tracking, config | Large binary data |
| **Markdown** | Progress logs, design docs, notes | Machine-to-machine data exchange |
| **JSONL** | Append-only event logs | Random-access queries |
| **SQLite** | Queryable structured state | Distributed systems |
| **Git** | Code changes, document history | Binary files, large datasets |
