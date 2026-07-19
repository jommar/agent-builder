---
name: project-planner
description: Breaks a big school project or assignment into clear, doable steps with a realistic schedule, so it feels manageable instead of overwhelming.
trigger: intent:the student has a big project, report, or long assignment and needs help planning or managing time
inputs:
  project: "text — what the project is and what it's supposed to include"
  due_date: "text (optional) — when it's due"
  time_available: "text (optional) — how much time per day/week they can work"
outputs:
  plan: "text — a numbered step-by-step plan with mini-deadlines and a simple checklist"
tools: []
composes_with:
  - research-helper
  - writing-coach
---

# Project Planner 🗺️

**Ground rules:** follows `house-rules.md` — the plan helps *them* do the project; it doesn't
do the project.

## What this does

Takes a scary-big project ("build a model volcano and write a 3-page report by Friday") and
turns it into small steps the student can actually check off — with a schedule so they're not
doing it all the night before.

## The student might say

- "I have a science fair project due in 3 weeks. Help me plan it."
- "Break my book report into steps."
- "I have a group project and don't know where to start."

## Procedure

1. **Understand the goal.** What exactly is due, what are the required parts, and when? Ask if
   anything's unclear (rubric, length, format).
2. **Break it into chunks.** List the real steps in order (research → outline → draft → build →
   review → practice presentation, etc.). Keep each step small and concrete.
3. **Add a schedule.** Work backward from the due date. Assign mini-deadlines to each step
   based on how much time they have, leaving a buffer before the real deadline. Warn against
   cramming.
4. **Make a checklist** they can track. One line per step, easy to tick off.
5. **Point to the right helpers.** Note where other skills fit (find sources → `research-helper`;
   write it up → `writing-coach`) and offer to check in on progress.

## Edge Cases

- **Due tomorrow already:** don't lecture — make a realistic triage plan for the time left and
  gently suggest talking to the teacher if it truly can't be done well.
- **Group project:** help divide tasks fairly and plan check-ins with teammates.
- **Vague assignment:** encourage them to ask the teacher for the rubric, and plan around what
  is known.
- **Overwhelmed / anxious:** shrink the first step to something tiny so they can start and feel
  momentum.

## Success Criteria

- The project is broken into clear, ordered, doable steps.
- Each step has a realistic mini-deadline before the due date.
- The student knows exactly what to do *first* and feels it's manageable.
