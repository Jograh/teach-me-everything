# course-tutor-setup

A Claude skill that turns a university course into a personal tutoring system inside a claude.ai Project.

Give it your syllabus and lesson files, and it sets up:

- **A tutor** that teaches only from your course materials, one topic at a time, with inline diagrams, interactive simulations and quizzes. It never skips the comprehension check.
- **A Google Sheet progress tracker** with a dashboard (pace, days to the final, topics due for review), lessons, topics with spaced-repetition review dates, a session log and grade tracking.
- **A Google Drive folder** for lesson notes and reference docs (glossary, cheat sheet, final review).
- **Optional check-ins** on a schedule, e.g. Mon/Wed/Sat mornings. Each one reads your sheet, tells you whether you're on pace, and gives you a day-by-day catch-up plan when you're behind.

The teaching loop is adapted from Matt Pocock's `/teach` skill: mission first, trusted sources only, small steps, retrieval practice, spacing and interleaving.

## How a session works

```
"next lesson"
  → pace check (dashboard)
  → warm-up: 3–5 review questions on due topics
  → per topic: hook → short lesson + visual → MCQs + explain-it check
  → log scores to the sheet → spaced review date updates itself
```

Other commands: `quiz me`, `review`, `catch-up`.

## Requirements

- claude.ai with **Projects** (one Project per course)
- **Google Drive** and **Google Sheets** connectors
- Course materials Claude can reach: a folder on your computer linked through the Claude desktop app, files attached to the chat, or files in Drive
- Optional: scheduled tasks, for the check-ins

## Install

**claude.ai:** download [`skills/course-tutor-setup/SKILL.md`](skills/course-tutor-setup/SKILL.md), zip the `course-tutor-setup` folder, then upload it under **Settings → Capabilities → Skills**.

**Claude Code:**

```bash
git clone https://github.com/Jograh/course-tutor-setup.git
cp -r course-tutor-setup/skills/course-tutor-setup ~/.claude/skills/
```

## Use

1. Create a claude.ai Project for the course (e.g. "MATH 204").
2. In a chat inside that Project, say: **"Set up a course tutor for this class"** and point it at your course materials.
3. Answer the intake questions: exam dates, timeline, target grade, check-in days.
4. Review the plan it shows you, then approve it.
5. Paste the one line it gives you into the Project's custom instructions.
6. Start studying with **"next lesson"**.

## What gets created

| Where | What |
|---|---|
| Project docs | `TEACHING-METHOD.md`, `MISSION.md`, `COURSE-MAP.md`, `NOTES.md`, `VISUAL-KIT.md` |
| Google Drive | `<Course>/Lesson Notes/`, `<Course>/Reference/`, `<COURSE> Progress` sheet |
| Sheet tabs | Dashboard · Timeline · Lessons · Topics · Session Log · Check-ins · Assessments |
| Scheduled task | `<COURSE> check-in` (optional) |

## Customizing

All the behaviour lives in `SKILL.md`. Common tweaks:

- **Review intervals:** the `CHOOSE(...,1,3,7,14)` formula in the Topics tab (days until the next review)
- **Mastery thresholds:** 70% / 90% in the logging rules
- **Check-in format:** the prompt template in Phase 5

## License

MIT
