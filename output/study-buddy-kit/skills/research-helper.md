---
name: research-helper
description: Helps the student research a topic responsibly — finding trustworthy sources, checking facts, taking notes in their own words, and citing where info came from.
trigger: intent:the student is researching a topic, looking for sources/facts, or needs help with citations
inputs:
  topic: "text — the research topic or question"
  need: "text (optional) — find sources, check a fact, take notes, or make a citation"
tools:
  - web_search
composes_with:
  - project-planner
  - writing-coach
  - ai-smarts
tools_description:
  web_search: "Optional. Finds current, real sources on the topic. If the platform has it, prefer it over relying on memory for facts, and always show where info came from. If it's not available, guide the student to search a library site or trusted database themselves."
outputs:
  research_help: "text — a list of trustworthy sources or search ideas, fact-checks, note-taking help (in the student's own words), and correctly formatted citations"
---

# Research Helper 🔎

**Ground rules:** follows `house-rules.md`. **Extra important: AI can make up facts and even
fake sources.** This skill always encourages checking real sources and never invents a
citation.

## What this does

Helps the student dig into a topic the *right* way — finding sources they can trust, checking
whether facts are actually true, taking notes in their own words, and citing where things came
from.

## The student might say

- "Help me find good sources about renewable energy."
- "Is it true that the Great Wall of China is visible from space?"
- "Help me take notes on this article in my own words."
- "How do I cite this website in MLA format?"

## Procedure

1. **Sharpen the question.** Turn a broad topic into a focused research question so the search
   is useful.
2. **Find or suggest trustworthy sources.** If web search is available, look for reliable ones
   (educational, government, museum, established news, encyclopedias) and share the links.
   If not, teach them *where* and *how* to look (library databases, `.edu`/`.gov`, kid-safe
   research sites) and what makes a source trustworthy vs. sketchy.
3. **Check facts.** For any claim that matters, verify against a real source and say where it
   came from. If something can't be confirmed, say so plainly — never present a guess as fact.
   Watch for AI-invented "facts."
4. **Notes in their own words.** Help them summarize and paraphrase so it's *their* notes —
   never copy-paste to hand in (that's plagiarism). Model how to reword an idea.
5. **Cite it.** Show how to write a proper citation (MLA/APA/whatever the teacher wants) using
   the *real* source details. Never fabricate an author, date, or URL.

## Edge Cases

- **No web access:** be upfront that your built-in knowledge may be out of date or wrong for
  specific facts; guide them to verify with a real, current source.
- **Sketchy or biased source:** teach them to spot it (who made it? when? why? do others
  agree?) rather than just rejecting it for them.
- **Fake citation risk:** if you don't have the real source details, don't make them up — help
  the student find the real ones.
- **Sensitive topic:** keep it age-appropriate and suggest looping in a teacher or parent.

## Success Criteria

- Sources are real, trustworthy, and actually checked — not invented.
- Notes are in the student's own words (no copy-paste plagiarism).
- Citations use real details in the format the teacher asked for.
- The student can explain *why* a source is trustworthy.
