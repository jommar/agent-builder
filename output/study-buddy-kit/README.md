# Study Buddy Kit — for parents 👨‍👩‍👧

A small, self-contained set of AI helpers designed for **kids aged 10–15**, primarily for
schoolwork but useful for general curiosity too. Everything is plain markdown, so it works with
almost any AI tool — copy, paste, or upload as needed. (Read the age note below before you pick
a tool.)

This kit was built to be **safe and learning-first**. It coaches your child to *understand*
their work rather than doing it for them, and it has age-appropriate safety and privacy rules
built in.

---

## What's in the box

```
study-buddy-kit/
├── START-HERE.md          ← Have your child read this first (written for them)
├── README.md              ← You are here (for you)
├── house-rules.md         ← The shared safety + learning rules everything follows
├── agents/
│   └── study-buddy.md     ← The main "tutor" personality. This is the core file.
└── skills/                ← Focused helpers the buddy uses
    ├── profile-builder.md   (get to know the student; save a tailored Learning Profile)
    ├── explain-it.md        (explain any concept at the right level)
    ├── homework-helper.md   (step-by-step homework coaching)
    ├── quiz-me.md           (interactive practice quizzes)
    ├── flashcard-maker.md   (turn notes into study cards)
    ├── writing-coach.md     (essay/writing feedback — never ghost-writes)
    ├── reading-buddy.md     (reading comprehension + hard words)
    ├── research-helper.md   (find & check sources, cite correctly)
    ├── project-planner.md   (break big projects into steps)
    └── ai-smarts.md         (how to use AI well and safely)
```

---

## How to set it up (pick your tool)

> **⚠️ Check the AI tool's own minimum age first.** As of 2026, consumer **Claude requires users
> to be 18+**; **ChatGPT and Gemini require 13+** (with parental consent for minors, and under-13
> access on Gemini only through a Google **Family Link** account). So for the younger end of the
> 10–15 range, most consumer tools aren't a fit without a parent-managed or education/family
> account. Verify current terms, use an age-appropriate account you manage, and prefer a
> family/education tier where one exists. **This kit is instructions layered *on top of* whatever
> tool you choose — it does not change the tool's own age rules or safety systems.**

You only strictly need **`agents/study-buddy.md`** to get started — it has the rules and the
coaching style built in. The `skills/` files add specialized help you can include as you like.

**Easiest — a "Project" or "custom instructions" (Claude, ChatGPT, Gemini):**
1. Create a new Project (Claude) or a Custom GPT / saved instructions (ChatGPT), etc.
2. Paste the contents of `agents/study-buddy.md` into the system prompt / custom instructions
   box.
3. (Optional) Upload the `skills/*.md` files and `house-rules.md` as project knowledge/files so
   the buddy can pull them in.
4. Have your child open that Project and start chatting.

**Quickest — just paste:** Open any AI chat, paste the contents of `agents/study-buddy.md` as
the first message, then let your child talk to it. (Rules reset each new chat, so a Project is
better for regular use.)

**As Claude Skills (advanced):** put each skill in its own folder as `SKILL.md` (e.g.
`skills/quiz-me/SKILL.md`) and place them where your Claude tool loads skills. The frontmatter
is already in the right format.

---

## Design choices you should know about

- **Balanced coaching.** You chose "balanced": the buddy coaches with hints first, but *will*
  give a direct answer with the reasoning when your child has tried or explicitly asks. It then
  checks understanding. For math/science this means "answer + why"; for anything they'll hand in
  for a grade it shifts to coaching (see next point).
- **No doing graded work for them.** The buddy won't write final essays/reports, and won't just
  dump every answer to a graded worksheet, problem set, or vocab list on request — it coaches
  and checks understanding instead. This protects your child from accidentally crossing into
  plagiarism and keeps the learning real. (A prompt can't perfectly enforce this if a child
  insists something isn't graded — supervision still matters.)
- **Personalized, safely.** The first time your child chats, the buddy opens with a quick,
  friendly get-to-know-you (at minimum "what grade are you in?") and adapts its reading level and
  tone on the spot — simpler and more playful for a younger child, more grown-up for a teen — so
  it fits whoever's using it with no setup from you. With `profile-builder` it can also save a
  short **Learning Profile** (grade, interests, how they learn) so that tailoring carries across
  sessions. It collects *learning* info only — a nickname (or at most a first name), never last
  name, school, address, or sensitive details — and the profile is plain text you can read, edit,
  or delete. On tools without built-in memory, save the profile card and paste it back at the
  start of a session (see the skill file for how). The saved card is a small file with your
  child's nickname and grade — keep it somewhere private and delete it when you're done, and on a
  shared account with more than one child use a separate card per child rather than the tool's
  built-in memory. Saving a profile is optional; **for under-13s, set it up together the first
  time.**
- **Model-agnostic.** No specific AI model is assumed or required — it works with whatever
  you're running.

## Safety features built in

- Won't ask for private info (full name, address, school, passwords, phone, photos), and won't
  repeat it back if your child shares it. *(A prompt can't control how the AI company logs,
  retains, or trains on chats — check your tool's data/privacy settings and turn off chat
  history / model-training where you can.)*
- Always presents itself as an AI, not a person.
- Steers serious personal issues (self-harm, bullying, abuse, anything scary) to a trusted adult
  **and shares real crisis resources** — 988 Suicide & Crisis Lifeline (call/text), Crisis Text
  Line (text HOME to 741741), and Childhelp (1-800-422-4453). *These are US numbers — if you're
  elsewhere, swap in your country's lines in `house-rules.md`.*
- Keeps content school-appropriate (while still allowing age-appropriate health/history/science).
- Teaches your child that AI can be wrong and how to verify facts.
- Encourages breaks during long sessions.

**Please note:** these rules live inside the prompt files and depend on the AI following them.
They dramatically reduce risk but are **not a guarantee**, and they do not replace your AI tool's
own built-in safety systems. For younger kids especially, we recommend using these together —
nearby and occasionally checking in — rather than fully unsupervised. Also turn on your AI tool's
own parental/family settings for an added layer.

## Make it yours

Every file is plain text you can edit. Common tweaks:
- Change the coaching style (edit "How you work" in `agents/study-buddy.md`).
- Tighten or loosen the rules, or localize the crisis numbers (edit `house-rules.md`).
- Add your family's own rules (screen-time limits, which subjects, etc.).

---

## Research notes (why it's built this way)

Grounded in current (2026) guidance from educators and child-safety sources:

- **Guided / Socratic learning is the 2026 gold standard** for AI tutors (e.g. Claude Learning
  Mode, ChatGPT Study Mode, Gemini Guided Learning, Khanmigo) — guide toward understanding
  rather than handing over answers.
- **Academic integrity:** teach AI as a *partner, not a shortcut*. The guiding question is
  *"Am I using this to learn, or just to skip the effort?"* AI-written graded work is
  considered cheating at most schools.
- **Verify facts:** AI can invent facts, quotes, and even sources — kids should learn healthy
  skepticism and cross-check important claims.
- **Safety for minors:** 2026 guidance and new laws (e.g. California SB 243) push for clear
  AI-not-human disclosure, break reminders, a "may not be suitable for minors" warning, and —
  importantly — a **crisis-referral protocol** that directs at-risk kids to a suicide/crisis
  line. (Minimizing kids' personal data comes from privacy law like COPPA.)

Sources:
- [Best AI Tutoring Apps for Kids in 2026 — SpellingJoy](https://spellingjoy.com/best-apps/ai-tutoring-apps)
- [Teaching AI Ethics in Middle and High School — Bespoke ELA](https://www.bespokeclassroom.com/blog/2026/7/8/teaching-ai-ethics-in-middle-and-high-school-the-complete-guide-to-responsible-ai-use-in-the-classroom)
- [How to teach students to use AI responsibly — Understood.org](https://www.understood.org/en/articles/ai-responsible-use-students)
- [Teaching Students to Use AI Responsibly — Edutopia](https://www.edutopia.org/article/teaching-students-use-ai-responsibly/)
- [State Children's Online Safety Laws Expand to AI Chatbots — MultiState](https://www.multistate.us/insider/2026/4/30/state-childrens-online-safety-laws-expand-beyond-social-media-in-2026)
- [California SB 243 (companion chatbot law) — CA Legislative Info](https://leginfo.legislature.ca.gov/faces/billNavClient.xhtml?bill_id=202520260SB243)
