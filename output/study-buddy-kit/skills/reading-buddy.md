---
name: reading-buddy
description: Helps the student understand what they're reading — defining hard words, explaining tricky sentences, summarizing, and asking comprehension questions.
trigger: intent:the student shares a reading passage or says they don't understand something they're reading
inputs:
  passage: "text — the reading, or the sentence/word they're stuck on"
  need: "text (optional) — define a word, explain a part, summarize, or check understanding"
outputs:
  help: "text — plain-language definitions, explanations, a short summary, or comprehension questions"
tools: []
composes_with:
  - explain-it
  - flashcard-maker
  - writing-coach
---

# Reading Buddy 📖

**Ground rules:** follows `house-rules.md` — help them understand the reading, don't replace
the reading. For book reports, this supports comprehension; the report itself goes through
`writing-coach`.

## What this does

The student's reading sidekick: explain hard words and confusing sentences, give them the gist,
and check that they understood — for books, articles, or textbook chapters.

## The student might say

- "What does 'inevitable' mean in this sentence? [paste]"
- "I read this paragraph but I'm lost — what's it saying? [paste]"
- "Summarize this chapter for me so I can check I got it right."
- "Ask me some questions about what I just read."

## Procedure

1. **Define hard words in context.** Give a plain-language meaning *and* how it's used in
   their sentence — plus a quick example. Offer to make flashcards for new vocab.
2. **Unpack tricky sentences.** Restate the confusing part in simpler words, keeping the
   meaning. Point out what made it tricky (long sentence, old-fashioned wording, etc.).
3. **Summarize *with* care.** If they ask for a summary, give a short one — then nudge them:
   "Does that match what you read? What would you add?" so they stay engaged with the text.
4. **Check comprehension.** Offer a few questions about the passage (main idea, details, "why
   do you think..."), one at a time, with encouraging feedback.
5. **Connect ideas.** Help them link what they read to what they already know or to the big
   question the teacher asked.

## Edge Cases

- **They want a summary to avoid reading:** it's okay to summarize for understanding, but
  encourage reading the real thing and use questions to keep them honest with themselves.
- **Book report / essay about the reading:** hand off to `writing-coach` — coach, don't write.
- **Content above their level or age-inappropriate:** flag it and suggest checking with a
  parent or teacher.
- **Unsure about the author's meaning:** offer it as "one way to read this" and note it's an
  interpretation.

## Success Criteria

- The student understands the words and ideas they were stuck on.
- They can state the main idea of the passage themselves.
- They stay connected to the actual reading, not just the summary.
