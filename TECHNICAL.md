# Technical details

This page covers how `course-tutor-setup` works and how to customize it. For the plain-language overview and setup, see the [README](README.md).

## Platform: claude.ai chat, not Claude Code

The skill is written for the **claude.ai chat interface inside a Project**. It depends on tools that exist there:

- Claude's built-in Visualizer (`show_widget`) and quiz card (`quiz_display_v0`) for inline visuals
- Project file tools (`project_write`) for the tutor docs
- Google Drive and Google Sheets connectors
- Scheduled tasks (`create_trigger`), for the optional check-ins
- `conversation_search` and device-folder tools (`device_list_dir`) for finding materials

These aren't available in Claude Code, so installing the skill there isn't supported. It never produces HTML files for you to open in a browser; all visuals are native to the chat.

## What it sets up

Given your syllabus and lesson files, the skill builds:

- **A tutor** that teaches only from your course materials, one topic at a time, with inline diagrams, interactive simulations and quizzes. It never skips the comprehension check.
- **A study timeline you control.** Set the final date, quiz dates and target grade, or bring an existing plan (PDF, calendar, list). It schedules lessons backward from your exam and rebuilds the plan when you fall behind.
- **A Google Sheet progress tracker** with a dashboard (pace, days to the final, topics due for review), lessons, topics with spaced-repetition review dates, a session log and grade tracking.
- **A Google Drive folder** for lesson notes and reference docs (glossary, cheat sheet, final review).
- **Optional check-ins** on a schedule, e.g. Mon/Wed/Sat mornings. Each one reads your sheet, tells you whether you're on pace, and gives you a day-by-day catch-up plan when you're behind.

## Requirements

- claude.ai with **Projects** (one Project per course)
- **Google Drive** and **Google Sheets** connectors
- Course materials Claude can reach: a folder on your computer linked through the Claude desktop app, files attached to the chat, or files in Drive
- Optional: scheduled tasks, for the check-ins

## Install (manual)

Download [`skills/course-tutor-setup/SKILL.md`](skills/course-tutor-setup/SKILL.md), zip the `course-tutor-setup` folder, then upload it under **Settings → Capabilities → Skills** in claude.ai.

Alternatively, paste the repo link into a chat inside the course's Project and ask Claude to set up a course tutor from it. Claude reads `SKILL.md` and follows it without a separate install.

## Setup flow

1. Create a claude.ai Project for the course.
2. In a chat inside that Project, ask for a course tutor and point it at the course materials.
3. Answer the intake questions: exam dates, timeline, target grade, check-in days.
4. Review the plan it shows, then approve it. Nothing is written to Drive or the Project until you do.
5. Paste the one line it gives you into the Project's custom instructions (the Project instructions can't be set by tool): `Before anything, read claude/TEACHING-METHOD.md and follow it.`
6. Start with **"next lesson"**.

## Session flow

```
"next lesson"
  → pace check (dashboard)
  → warm-up: 3–5 review questions on due topics
  → per topic: hook → short lesson + visual → MCQs + explain-it check
  → log scores to the sheet → spaced review date updates itself
```

Other commands: `quiz me`, `review`, `catch-up`.

The full tutor loop, logging rules and session types live in the `TEACHING-METHOD` template inside [`SKILL.md`](skills/course-tutor-setup/SKILL.md).

## What gets created

| Where | What |
|---|---|
| Project docs | `TEACHING-METHOD.md`, `MISSION.md`, `COURSE-MAP.md`, `NOTES.md`, `VISUAL-KIT.md` |
| Google Drive | `<Course>/Lesson Notes/`, `<Course>/Reference/`, `<COURSE> Progress` sheet |
| Sheet tabs | Dashboard · Timeline · Lessons · Topics · Session Log · Check-ins · Assessments |
| Scheduled task | `<COURSE> check-in` (optional) |

## Learning methods

Mission first, trusted sources only, one small idea per lesson, no skipped checks, retrieval practice, spaced repetition, interleaving and learning records. Adapted from Matt Pocock's `/teach` skill.

## Customizing

All the behavior lives in `SKILL.md`. Common tweaks:

- **Review intervals:** the `CHOOSE(...,1,3,7,14)` formula in the Topics tab (days until the next review)
- **Mastery thresholds:** 70% / 90% in the logging rules
- **Check-in format:** the prompt template in Phase 5

## Skill phases

For orientation, `SKILL.md` runs in these phases:

0. **Intake**: course, dates, timeline, mission, check-ins
1. **Inventory the sources**: syllabus, textbook, lesson files, topic list
2. **Present the plan**, then wait for approval
3. **Google Drive and the progress sheet**: folders, tabs, formulas, formatting
4. **Project docs**: the five tutor docs
5. **Scheduled check-ins** (optional)
6. **Verify and hand off**

The file ends with a list of gotchas (date-format inheritance in Sheets, `copyPaste` for fill-down formulas, Drive search using `title`, and so on).
