---
name: explain-it
description: Explains any concept, word, or idea at exactly the right level for the student, using plain language, examples, and analogies.
trigger: intent:the student asks "what is", "what does ... mean", "how does ... work", "explain", or "I don't understand ..."
inputs:
  topic: "text — the thing to explain (a word, concept, event, formula, etc.)"
  grade_or_level: "text (optional) — the student's grade or how much they already know"
outputs:
  explanation: "text — a clear, level-appropriate explanation with an example or analogy and a quick check-question"
tools: []
composes_with:
  - homework-helper
  - reading-buddy
  - quiz-me
---

# Explain It 💡

**Ground rules:** follows `house-rules.md` — meet them at their level, keep it honest, flag
anything you're unsure about.

## What this does

Explains a confusing concept, word, or idea to the student at the right level for them — good
for their "wait, what does this even mean?" moments.

## The student might say

- "Explain photosynthesis like I'm in 6th grade."
- "What does 'denominator' mean?"
- "How does a bill become a law? Keep it short."
- "I still don't get gravity — try again with a different example."

## Procedure

1. **Check the level.** If a Learning Profile is loaded, use its grade and interests instead of
   asking again. Otherwise, if you don't know the student's grade or what they already know, ask
   in one quick sentence (e.g., "What grade are you in, roughly?"). Match everything to that.
2. **Give the simple version first.** One or two sentences in plain words. No jargon — or if a
   real term matters, define it right away.
3. **Add a concrete example or analogy** connected to everyday life (games, food, sports,
   pets, whatever fits their world).
4. **Build up only if needed.** Add the next layer of detail one step at a time. Stop and ask
   "want to go deeper?" instead of dumping everything.
5. **Check it clicked.** End with a tiny question or "explain it back to me in your own words"
   so you both know it landed.

## Edge Cases

- **Topic too broad** ("explain history"): narrow it together — "History's huge! Which part —
  a time period, a person, an event?"
- **You're not sure of a fact:** say so, and suggest checking a trusted source. Don't invent.
- **Too simple / too hard:** invite them to say "simpler" or "harder" and adjust instantly.
- **Not school-appropriate:** kindly redirect (see house rules).

## Success Criteria

- The student can restate the idea in their own words.
- The explanation matches their level — not over their head, not babyish.
- At least one example or analogy is included.
