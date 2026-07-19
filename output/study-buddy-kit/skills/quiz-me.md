---
name: quiz-me
description: Creates a friendly practice quiz on any topic, asks one question at a time, gives encouraging feedback, and finishes with a score and what to review.
trigger: intent:the student asks to be quizzed, tested, or wants to practice/review for a test
inputs:
  topic: "text — what to be quizzed on"
  count: "number (optional) — how many questions (default 5)"
  style: "text (optional) — multiple choice, true/false, short answer, or mixed (default mixed)"
outputs:
  quiz: "an interactive question-by-question quiz with feedback and a final score plus review tips"
tools: []
composes_with:
  - flashcard-maker
  - explain-it
---

# Quiz Me 📝

**Ground rules:** follows `house-rules.md` — encouraging, level-appropriate, honest about
correct answers.

## What this does

Turns studying into a game: ask the student questions one at a time, tell them how they did,
and show them exactly what to review before a test.

## The student might say

- "Quiz me on the water cycle, 5 questions."
- "Give me 10 multiple-choice questions on the American Revolution."
- "Test me on my Spanish vocab — I'll paste the words."

## Procedure

1. **Set it up.** Confirm the topic, how many questions, and the style (multiple choice,
   true/false, short answer, or mixed). Use the Learning Profile's grade if one is loaded;
   otherwise ask their grade if it affects difficulty.
2. **One question at a time.** Ask a question, then **wait** for the answer. Don't reveal the
   answer or ask the next question until they respond.
3. **Give feedback right away.** Say if it's right or not, warmly. If wrong, give a short
   explanation of the correct answer (teach, don't scold) — this is fair game since a quiz is
   for learning.
4. **Track the score roughly** and vary difficulty a little based on how they're doing.
5. **Wrap up.** Show the final score, highlight the topics they missed, and suggest what to
   review (offer to make flashcards with `flashcard-maker` or explain a tricky one with
   `explain-it`).

## Edge Cases

- **Student pastes their own notes:** build the quiz from *their* material so it matches class.
- **They keep missing the same idea:** pause the quiz and actually teach it before moving on.
- **Fact you're unsure about:** don't include shaky facts; stick to what you're confident in,
  or flag it.
- **They want the answers list up front:** that defeats the practice — offer flashcards
  instead for review, then quiz.

## Success Criteria

- Questions are accurate and match the topic and grade level.
- Feedback is immediate, kind, and explains the correct answer.
- The student ends knowing their score and exactly what to review.
