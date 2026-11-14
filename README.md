# teach-me-everything

**Turn Claude into a personal tutor for any class you're taking.**

Give it your course materials and your exam date. It builds you a study plan, teaches you one topic at a time with diagrams and interactive visuals, quizzes you, and keeps track of how you're doing.

> **Made for [claude.ai](https://claude.ai) (the chat side, using a Project).**
> This is **not** a Claude Code tool. There's nothing to run in a terminal and no code to write. You just chat.

---

## What you get

- **A study plan built around your exam date.** Tell it when your final is and what grade you're aiming for. It works backward and gives you a day-by-day plan. It rebuilds the plan if you fall behind.
- **Lessons you can see and play with.** Diagrams, sliders you can drag, quiz cards and flashcards, all right inside the chat. Nothing to download or open.
- **Teaching that sticks.** It teaches one topic at a time, quizzes you, and brings old topics back for review right before you'd forget them.
- **A progress tracker.** A Google Sheet keeps your scores, what's due for review, and whether you're on pace.
- **Check-ins (optional).** Get a scheduled message a few mornings a week telling you if you're on track, and what to do if you're not.

It teaches only from *your* course materials (syllabus, slides, textbook), so you're studying what your class actually covers.

---

## What you need

- A **claude.ai** account that has **Projects**
- The **Google Drive** and **Google Sheets** connections turned on in Claude
- Your course materials: a folder on your computer, files you attach to the chat, or files in Google Drive

---

## Setup

### The easy way (recommended): just ask Claude

1. In claude.ai, create a **Project** for your class (for example, "MATH 204").
2. Start a chat inside that Project and paste this:

   > Set up a course tutor for this class using this: https://github.com/Jograh/teach-me-everything

3. Claude reads the instructions from the link, asks you a few questions (exam dates, target grade, how often you want check-ins), and shows you the plan.
4. Say go ahead. Claude builds everything for you.

That's it. You don't need to download or install anything.

### The manual way: add it as a Claude skill

If you'd rather have it permanently available in your account:

1. Download [`skills/course-tutor-setup/SKILL.md`](skills/course-tutor-setup/SKILL.md) from this page.
2. Put it in a folder named `course-tutor-setup` and zip that folder.
3. In claude.ai, go to **Settings → Capabilities → Skills** and upload the zip.
4. Create a Project for your class, then say: **"Set up a course tutor for this class."**

---

## Using it

Once setup is done, open your class's Project and just say what you want:

| Say this | What happens |
|---|---|
| **"next lesson"** | Checks your pace, quizzes you on older topics, then teaches the next lesson |
| **"quiz me"** | A mixed quiz on what's due and what you're weakest at |
| **"review"** | Brings back topics that are due for review |
| **"catch-up"** | A shorter, faster path through lessons you've missed |

A typical session: quick warm-up questions, then each topic gets a short lesson with a visual, a few quiz questions and an "explain it in your own words" check. Your scores are saved automatically.

---

## Want the technical details?

How it works under the hood, what gets created, and how to customize it are in [TECHNICAL.md](TECHNICAL.md).

## Credit

The teaching methods are inspired by Matt Pocock's `/teach` skill.

## License

MIT
