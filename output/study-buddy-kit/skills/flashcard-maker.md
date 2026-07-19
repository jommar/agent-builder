---
name: flashcard-maker
description: Turns notes, a textbook section, or a topic into a clean set of study flashcards (question/answer pairs) the student can review or export.
trigger: intent:the student asks for flashcards, study cards, or a way to memorize terms/facts
inputs:
  source: "text — notes, a list of terms, a topic, or pasted reading to build cards from"
  count: "number (optional) — how many cards (default: cover the key points, usually 8–15)"
outputs:
  flashcards: "a numbered list of Front (question/term) / Back (answer/definition) pairs, plus optional copy-paste formats"
tools: []
composes_with:
  - quiz-me
  - reading-buddy
---

# Flashcard Maker 🃏

**Ground rules:** follows `house-rules.md` — accurate, level-appropriate, honest.

## What this does

Turns the student's pile of notes or a topic into neat study cards they can quiz themselves
with. Perfect for vocab, dates, formulas, and definitions.

## The student might say

- "Make flashcards from these notes: [paste]."
- "Give me 12 flashcards on the parts of a cell."
- "Turn my Spanish vocab list into cards."

## Procedure

1. **Get the material.** Use the student's pasted notes if they have them (best — it matches
   class). Otherwise build from the topic. Confirm roughly how many cards. If the pasted text
   looks like a graded worksheet to fill in (rather than study notes), check first: "Is this an
   assignment you'll hand in? I can help you *study* the topic instead of just filling in the
   answers."
2. **Pull the key points.** Identify the most important terms, facts, or ideas — the stuff
   likely to be tested. Don't pad with trivia.
3. **Write clear pairs.** Each card = a short **Front** (a question or term) and a concise
   **Back** (the answer/definition in plain, grade-appropriate words). One idea per card.
4. **Number them** and keep the set focused. Group by sub-topic if it's long.
5. **Offer next steps:** a plain list, a `Term — Definition` format that pastes into Quizlet/
   Anki, or "want me to quiz you on these?" (→ `quiz-me`).

## Edge Cases

- **Notes are messy or incomplete:** ask a clarifying question or fill obvious gaps, but flag
  anything you had to guess.
- **Too many cards:** cap it and keep the highest-value ones; tell them what you left out.
- **Facts you're unsure of:** leave them out or mark them "double-check" rather than guessing.
- **Whole textbook pasted:** focus on the section that matters; ask which part is on the test.
- **Pasted text is actually a graded fill-in worksheet:** don't just complete it — offer to make
  study cards from the underlying topic so they learn it, and suggest checking with their teacher
  about AI use.

## Success Criteria

- Cards cover the important, testable material — not filler.
- Each card is one clear idea, front and back, at the right level.
- The set is easy to review or export.
