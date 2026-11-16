---
name: course-tutor-setup
description: Set up a full study system for one university course in its own claude.ai Project — a teach-style tutor method, a Google Drive folder, a progress Google Sheet, inline visual lessons, and optional scheduled accountability check-ins.
---

# Course tutor setup

Builds, for one course, a complete study system:

1. **Project docs** that make every chat in the course's Project behave as the same tutor: TEACHING-METHOD, MISSION, COURSE-MAP, NOTES, VISUAL-KIT.
2. **Google Drive**: the course folder with `Lesson Notes/` and `Reference/` subfolders and a `<COURSE> Progress` Google Sheet. The sheet is the learning record and the accountability engine.
3. **Tutor method** adapted from Matt Pocock's `/teach` skill: mission first, trusted sources only, one small idea per lesson, no skipped checks, retrieval practice, spacing, interleaving, and learning records. Everything is native to claude.ai chat: inline Visualizer diagrams and simulations plus the quiz card. Never produce HTML files for the student to open in a browser.
4. **Optional scheduled check-ins** (for example Monday, Wednesday and Saturday) that read the sheet, compare it to the timeline, send the week's to-dos with tips, and build a catch-up plan when the student is behind.

Run it inside the course's own Project. If the session is not in a Project, ask the student to create one for the course and start the chat there first.

## Standing preferences (apply everywhere)
- Concise, direct, practical. Visuals over prose.
- MCQs: vary the position of the correct answer and keep options the same length.
- Accountability matters. When the student is behind, say so plainly and give a concrete plan, without lecturing.
- Native claude.ai visuals and interactivity only. No HTML files to open in a browser.
- Math and symbols render as real math in chat: write them in LaTeX (`$x^2$`, `$$\int 2x\,dx = x^2 + C$$`), never as plain-text pseudo-math like `x^2` or `sqrt(x)`. Full rules are in ground rule 7 of the TEACHING-METHOD template.
- Respect any preferences the student has already saved in memory; they override these defaults.

## Phase 0: Intake

Ask only what can't be found in the materials. Use AskUserQuestion, with at most 4 questions per round.
- Course code and name, term, and the folder with the course materials (connected device folder, attached files, or Drive).
- Time zone (default to the one the session reports).
- Exam and assessment dates: final date, time and room, plus graded quiz dates if they're known.
- Study timeline: an existing one (PDF, calendar, list) or build one backward from the exam dates. Note any blackout periods, such as other finals.
- Mission: why they're taking it and a target grade.
- Check-ins: whether they want them, which days, and what time.

## Phase 1: Inventory the sources

1. List the materials folder recursively (`device_list_dir`). Identify the syllabus or outline, the textbook, and lesson transcripts or slides. Usually there is one file per topic.
2. Stage and extract the syllabus (`pdftotext -layout`). Record:
   - the textbook edition
   - readings per lesson or week
   - the grading breakdown
   - the assessment format: open or closed book, MC or essay, and whether wrong statements lose marks
   - any rule about which sources are accepted (for example, "content outside the lessons and textbook is not accepted")
3. Stage one sample transcript to learn its structure (hook questions, numbered screens, misconceptions it flags).
4. Build the topic list as one row per transcript, with topic ID, lesson number and title, taken from the file names.
5. For a large textbook (hundreds of MB), check whether it's cloud-only. Stage it with `acknowledge_cloud_downloads` only after telling the student. Use `pdfinfo` first, then `pdftotext -f/-l` for page ranges. Map the syllabus readings to lessons.
6. Search past chats in the Project (`conversation_search`) for earlier work on the course.

## Phase 2: Present the plan, then wait for approval

Show, concisely and with one mapping diagram in the Visualizer:
- what the tutor does (the session flow below)
- the Drive layout and the sheet tabs
- the timeline blocks with their dates
- the check-in schedule
- open questions

Do not write any docs, Drive files or scheduled tasks until the student approves. Approval can be implicit: answering the open questions and saying go ahead counts.

## Phase 3: Google Drive and the progress sheet

If a Google Workspace skill is available, load it (and its Sheets reference) before the first Sheets call.

1. **Find the existing course folder first.** Search `title contains '<course word>' and mimeType = 'application/vnd.google-apps.folder'`. Many students keep term folders, such as `Fall 2026/<Course>`. Reuse the folder if it exists, and never move or trash their files. Create it only if it is missing, inside the term folder if there is one.
2. Create the subfolders `Lesson Notes` and `Reference`.
3. Create the sheet `<COURSE> Progress` with Drive `create_file` and mime type `application/vnd.google-apps.spreadsheet`, parented to the course folder.
4. In one `update_spreadsheet` call:
   - rename sheet 0 to `Dashboard`
   - `addSheet` tabs with fixed sheetIds: Timeline 1, Lessons 2, Topics 3, Session Log 4, Check-ins 5, Assessments 6
   - set the spreadsheet `timeZone` to the student's time zone
5. Write each tab with `update_formulas`, one call per tab (the calls can run in parallel). Size ranges to the real row counts. Here N = topics + 1 (the last topic row) and L = lessons + 1 (the last lesson row).

**Timeline** (A:G): Block | Start | End | Lessons | Focus | Lessons done by end | Notes
- One row per block: lesson blocks, review passes, pauses, and the exam day.
- Dates go in as `yyyy-mm-dd` strings, and Sheets parses them.

**Lessons** (A:L): Lesson | Title | Textbook reading | Block | Due by | Status | Date completed | Topics studied | Topics total | Avg topic score | Mastery (1-5) | Notes doc
- H2: `=COUNTIFS(Topics!$B$2:$B$N,A2,Topics!$D$2:$D$N,"<>Not started")`
- I2: `=COUNTIF(Topics!$B$2:$B$N,A2)`
- J2: `=IFERROR(AVERAGEIFS(Topics!$H$2:$H$N,Topics!$B$2:$B$N,A2),"")`
- Fill H2:J2 down with `copyPaste` and `PASTE_FORMULA`.

**Topics** (A:K): Topic ID | Lesson | Topic | Status | First studied | Last reviewed | Times reviewed | Last score | Confidence (1-5) | Next review | Misconceptions to watch
- Prefix topic IDs with `'` so they stay text (`'1.3.4`).
- Status starts as "Not started" and Times reviewed as 0. Topics with nothing to check (intro and conclusion videos) get the status "No check" once covered, with score, confidence and review dates left blank.
- J2 holds the spaced-review formula: `=IF(F2="","",F2+IF(AND(H2<>"",H2<0.7),1,CHOOSE(MIN(MAX(G2,1),4),1,3,7,14)))`. Fill it down with `copyPaste` `PASTE_FORMULA`.

**Session Log** (A:F): Date | Type | Lessons / topics | Score | Mistakes and misconceptions | Next step

**Check-ins** (A:E): Date | Day | Pace | Lessons done / due | Plan sent

**Assessments** (A:F): Assessment | Weight | Date | Status | Score | Notes
- One row per graded item, with the weight as a fraction.
- Below the items, add the row "Weighted average of graded items" with `=IFERROR(SUMPRODUCT(Bx:By,Ex:Ey)/SUMIF(Ex:Ey,"<>",Bx:By),"")`.

**Dashboard** (A:B):

| Row | Label | Formula |
|---|---|---|
| A1 | Title | (no formula) |
| 3 | Today | `=TODAY()` |
| 4 | Final exam | date |
| 5 | Days until final | `=B4-B3` |
| 6 | Lessons done | `=COUNTIF(Lessons!F2:FL,"Done")` |
| 7 | Lessons due before today | `=COUNTIF(Lessons!E2:EL,"<"&B3)` |
| 8 | Lessons behind | `=MAX(0,B7-B6)` |
| 9 | Pace | `=IF(B8=0,"On track","Behind by "&B8&" lesson"&IF(B8>1,"s",""))` |
| 10 | Current block | `=IFERROR(LOOKUP(B3,Timeline!B2:Bk,Timeline!D2:Dk),"Not started")` |
| 11 | Current block ends | same `LOOKUP`, returning column C |
| 12 | Topics studied | `=COUNTIF(Topics!D2:DN,"<>Not started")&" of "&COUNTA(Topics!A2:AN)` |
| 13 | Topics due for review | `=COUNTIF(Topics!J2:JN,"<="&B3)` |
| 14 | Average topic score | `=IFERROR(AVERAGE(Topics!H2:HN),"No scores yet")` |
| 15 | Weighted grade so far | `=Assessments!B<avg row>` |
| 16 | Topics left in current block | `=IF(ISNUMBER(B11),SUMPRODUCT((Lessons!E2:EL<=B11)*(Lessons!I2:IL-Lessons!H2:HL)),"")` |
| 17 | Topics per day needed | `=IF(ISNUMBER(B11),ROUNDUP(B16/MAX(1,B11-B3+1),0),"")` |

Row 16 sums topics total minus topics studied for every lesson due on or before the current block's end. Row 17 divides that by the days left in the block, counting today. "Pace" only turns to "Behind" after a due date has passed, so these two rows are the early warning.

6. **Formatting**, in one `update_spreadsheet` batch (one set of requests per tab, merged).
   - Headers: bold white text on `#1F3864`. Freeze row 1 and set column widths.
   - Number formats: dates `ddd mmm d` on Timeline and Lessons due dates, `yyyy-mm-dd` elsewhere; scores `0%`; weights `0%`.
   - **Set `0` as the number format on Dashboard B5:B8, B13 and B16:B17.** Otherwise "Days until final" inherits a date format and shows `1900-02-02`.
   - Dropdowns (`setDataValidation` with `ONE_OF_LIST`):
     - Lessons Status: Not started, In progress, Done
     - Topics Status: Not started, No check, Learning, Reviewing, Mastered
     - Session Log Type: Lesson, Review, Practice quiz, Catch-up, Mock exam
     - Assessments Status: Upcoming, Done, Missed
   - Conditional formats (leave out the `index` field):
     - Pace cell: red fill when the text contains "Behind", green when it equals "On track"
     - Lessons rows: red bold text when `=AND($F2<>"Done",$E2<TODAY())`
     - Topics Next review: yellow when `=AND($J2<>"",$J2<=TODAY())`
   - `addBanding` on Topics.
7. **Verify.** Run `get_values` on every tab and check the Dashboard reads "On track", 0 done, and the right current block. Check that the formulas filled down with a grid read on a couple of cells.

## Phase 4: Project docs

Write them with the Projects tool (`project_write`). Bare names land in `claude/`. The Project's custom instructions can't be set by tool, so ask the student to paste this line into the Project instructions: "Before anything, read claude/TEACHING-METHOD.md and follow it."

- **MISSION.md**: why they're taking the course, hard dates, the exam format, what success looks like (every topic reviewed at least twice, none below 70% on its last check), the target grade, and that accountability is part of the mission. Change it only after confirming with the student.
- **COURSE-MAP.md**:
  - a "where things live" table: device paths, the sheet link and ID, folder links and IDs
  - a table of lessons with topic IDs, readings and due dates
  - the timeline table
  - the grading breakdown and assessment formats
- **TEACHING-METHOD.md**: the session rules below, filled in with the course's paths, IDs, check-in schedule and exam format. Include a column map for the sheet (tab, column letter, field) and the exact ranges to read at session start and write at session end, so no session has to rediscover the layout. A session start should need only this doc, NOTES.md and the sheet ranges; MISSION, COURSE-MAP and VISUAL-KIT are read when needed. Its first line tells any chat in the Project to read it before teaching, in case the Project instruction was never pasted.
- **NOTES.md**: the student's preferences (see above), plus a line for each time they correct the teaching.
- **VISUAL-KIT.md**: house style for the Visualizer plus a recipe table with one row per concept that has a control (a slider or toggle), for example seasons tilt, Kepler orbits, supply and demand shifts, probability distributions, matrix transforms. Add a row whenever a visual works well. Also record how math is written inside widgets (see ground rule 7 of the TEACHING-METHOD template).

### TEACHING-METHOD template (the tutor loop)

Ground rules:
1. Teach only from the course sources (transcripts or slides, and the textbook). Cite a topic ID or section. Flag anything beyond the course and follow the course.
2. Read the transcript before teaching a topic: extract it in place with `device_bash` and `pdftotext -layout` (stage the file only when there is no device shell). At the start of a lesson, extract all of that lesson's transcripts in one command. If the device is offline, ask for the file; never teach from memory.
3. One topic per step. Never dump a whole lesson.
4. No skipped checks.
5. Visuals are native: the Visualizer (`show_widget`, after `read_me`) and the quiz card (`quiz_display_v0`). Every visual must add something the text doesn't say. Don't put click-to-ask (`sendPrompt`) nodes on a visual shown in the same message as a check.
6. Keep it concise.
7. Math and symbols render as math, not plain text.
   - In chat, write every equation, formula and symbol in LaTeX markdown: inline `$...$`, and standalone equations on their own lines in `$$...$$`. A variable that stands for a quantity is math too: write `$x$`, not x. This covers fractions, exponents, roots, integrals, sums, limits, matrices, vectors, Greek letters, inequalities, set and logic symbols, units with powers (`$\mathrm{m/s^2}$`) and chemical formulas (`$\mathrm{H_2O}$`).
   - Never use plain-text pseudo-math such as `x^2`, `sqrt(x)`, `d/dx`, `>=`, `->` or "integral of". Ordinary numbers and everyday words stay normal text.
   - Don't put math in code blocks or backticks. They show the raw source instead of rendering it.
   - Worked solutions: one step per line inside `$$ \begin{aligned} ... \end{aligned} $$`, with a few words on what changed in each step.
   - Where LaTeX won't render, use real Unicode symbols instead (x², √x, π, ≤, ∫, →, H₂O): quiz cards and flashcards, widget labels (HTML `<sup>` and `<sub>`, or KaTeX from cdnjs.cloudflare.com if the Visualizer allows scripts), Google Sheets cells, Google Docs (lesson notes and reference docs), and check-in messages.
   - If the student says equations show up as raw `$` signs, tell them to turn on LaTeX rendering in claude.ai's settings, use Unicode until they have, and note it in NOTES.md.
8. Grade against the whole course, not only the current transcript. A statement that is correct and within the textbook or a later lesson is neutral: no credit, no penalty. Before calling something "not in the course", search the textbook text or the later transcripts. Penalise only statements that are wrong.

Session "next lesson":
1. Pace check: read the Dashboard and the last Check-ins row. Always state "Topics left in current block" and "Topics per day needed", even when Pace reads "On track". Ask how much time the student has and plan the session's topics to fit it. If behind, follow that plan.
2. Warm-up: 3-5 interleaved questions on topics due for review. Ask at least 2 questions on any topic whose status could change, and re-ask any short answer the student skipped last session.
3. Hook: the transcript's opening questions. Ask the student to guess first.
4. Teach: 3-6 short paragraphs, one visual, one concrete example, and the misconception the transcript flags.
5. Check: 3-4 MCQs (equal length, rotate the correct position) and one "explain in 2 sentences" question, graded like a TA (wrong statements lose credit when the course grades that way). Show the model answer.
   - The quiz card does not report scores back. Ask for one reply in this shape: `3/4, missed 2` followed by the short answer. Shorthand answers are fine. If part is missing, ask once, then score what is there and move on.
   - Scoring is fixed: each MCQ is 1 point, the short answer is 2 points (2 = complete and correct, 1 = right idea but missing the mechanism or vague, 0 = wrong or off-topic; subtract 0.5 per wrong statement, minimum 0). Topic score = points earned / points possible.
   - A skipped short answer is left out of the score, noted as "SA owed" in the topic's Misconceptions cell, and asked again in the next warm-up.
6. Repeat steps 3-5 for each topic.
7. Log once, at the end of the lesson or when the student stops (see Logging). Ask for confidence once per lesson, as one list of topics, not after every topic.

Other session types:
- **"Quiz me" / "review"**: 10-15 interleaved questions on due and weakest topics.
- **Voice sessions** (the student is talking, not typing, or visuals and tools aren't available on the call): ask one hook question at a time. MCQs have 3 short options, read once in full; never change a question when repeating it. Skip visuals and say they'll come in text. Teach only topics whose transcript was read before the call; if the next one can't be read, stop and say so. Keep a running tally of scores and misconceptions, and log everything as soon as the session is back in text, then show the visuals that were skipped.
- **"Catch-up"**: per topic, the hook, the core idea, a visual and 2 MCQs. Topics stay at Learning status.
- **Review passes / mock exams**: match the final's format, re-teach anything under 70%, and use flashcards (`initial_mode: "flashcards"`) for glossary drills.

Logging (every session; read the sheet once at the start, write once per tab at the end of each lesson or when the student stops, never after each topic):
- **Topics tab**: Status, First studied, Last reviewed, Times reviewed +1, Last score as a fraction, Confidence (ask once per lesson), Misconceptions. Never overwrite the Next review formula.
  - Status on first study: under 70% Learning, 70% or more Reviewing (never Mastered on first study).
  - Status on a review: a review counts only if the topic got at least 2 questions. Under 70% Learning, 70-89% Reviewing, 90% or more Mastered. With fewer than 2 questions, update Last reviewed and Times reviewed but leave Status and Last score alone.
  - Intro and conclusion videos: "No check".
- **Lessons tab**: Status Done, date, Mastery, notes doc link.
- **Session Log**: append one row.
- **Assessments**: update when the student reports a score or a date.
- **NOTES.md**: add a line whenever the student corrects the teaching, asks for something later ("review Moon phases again") or changes how they want sessions run (voice, shorter checks).
- **VISUAL-KIT.md**: add a row for each visual that worked this session (concept, control, what it showed).

Saved material (Drive, only things worth rereading):
- **Lesson notes**: one Google Doc per finished lesson in `Lesson Notes/`, named "L04 - Title". Contents: key ideas per topic, key terms, misconceptions hit, 5 self-test questions with answers at the end, readings.
- **Reference docs**: Glossary, Cheat sheet and Final review in `Reference/`. Append to these; don't make duplicates.
- Never save quizzes or transcripts.

## Phase 5 (optional): Scheduled check-ins

1. List the existing scheduled tasks (`list_triggers`) to avoid duplicates and clashing times.
2. Create the task with `create_trigger`:
   - name `<COURSE> check-in`
   - cron `CRON_TZ=<time zone> <m> <h> * * <days>`, for example days `1,3,6` for Monday, Wednesday and Saturday. Move an on-the-hour time 1-29 minutes earlier (letters in the name mod 29, plus 1). Avoid busy morning hours (9:00-12:00 US Eastern).
   - `requires_local_device: false`, `notifications: {push: true}`, `initiation: human_request`
   - leave `permission_mode` unset, and tell the student which approval setting the task got
3. The prompt must be standalone, because every run is a fresh session. Fill in this template:

```
You are <NAME>'s study accountability coach for <COURSE> (<course name>, <term>). This runs <days> mornings (<time zone>). Tell them exactly what they have to get done, check whether they are on pace with their timeline, and if they are behind, give a concrete catch-up plan that gets them back on it. Direct and specific, never preachy.

Data (only source of truth): Google Sheet "<COURSE> Progress", ID <id>. Read with get_values: Dashboard!A1:B17, Timeline!A1:G<k>, Lessons!A1:L<L>, Topics!A1:K<N>, Check-ins!A:E (last 2 rows, don't repeat yourself), Assessments!A1:F<r>. Use only numbers from the sheet. Get today's date in <time zone> first.

Timeline (the tab wins if it differs): <blocks with dates>. FINAL: <date, time, room, format, weight>.

Work out: lessons done vs due before today and vs pro-rated position inside the current block (behind if either is short); what's left in the block and when it ends; topics due for review and the 3 weakest; assessment dates in the next 7 days.

Catch-up rule: spread overdue plus remaining block lessons over the days left in the block as a day-by-day plan (~10 min per topic + 10 min warm-up). If it doesn't fit, spill over and name the day they're back on schedule; never run lessons past <last lesson cutoff>; never cut the final review pass. 2+ lessons behind: say this week needs extra sessions and name the days.

By day: Monday = week plan Mon-Sun + 2-3 tips; Wednesday = progress vs plan, what remains, catch-up if needed + 1-2 tips; Saturday = weekly scorecard, what the weekend must finish, next week preview. Review passes: daily focus, weakest first. Pauses: one short optional-review note. The day after the exam: one closing message; after that send nothing, and disable this task if a tool allows it.

Tips must name the actual upcoming lessons and weak topics. End with: open the <Course> project in Claude and say "next lesson" (or "catch-up", "quiz me").

Format: ONE SendUserMessage, phone-length markdown: bold line "<COURSE> · {Day Mon d} · {On track | Behind by N lessons} · {D} days to final", a text progress bar "Lessons ■■■□□□ 3/13 (due by today: 4)", the plan list, Tips, Start line. Write any symbols in plain Unicode (x², √, π), not LaTeX: a phone notification won't render it.
Then append a row to Check-ins: date, day, pace, "done / due", one-line plan summary.
If the sheet can't be read, send only a short message saying the progress sheet couldn't be read, to turn the Google Sheets connector back on, and the current block's lessons and end date.
Final reply after tools: one short line or empty.
```

4. Add the schedule to TEACHING-METHOD.md ("Accountability" section).

## Phase 6: Verify and hand off

- Run `get_values` on every tab. Check there are no `#` errors, the Dashboard is sensible, and the formulas filled down.
- The Project lists all 5 docs (`project_info`). If its `instructions` field is empty, remind the student once to paste the one-line instruction.
- The scheduled task shows the right cron and next run time.
- Send the student links to the sheet and folder in the sentence that names them, the first check-in time, the one-line Project instruction to paste, and how to start ("next lesson").

## Gotchas

- Both Sheets write tools parse input like the UI. Date strings become dates, and a formula cell next to a date column can inherit a date format. Set number formats explicitly.
- Fill formulas down with `copyPaste` and `PASTE_FORMULA` instead of sending dozens of formula strings.
- Leave `index` out of `addConditionalFormatRule`. An index past a tab's rule count can fail.
- Drive search uses `title`, not `name`.
- A 600 MB+ textbook may be cloud-only on the device. Tell the student before downloading it, or ask them to download it locally first.
- Project docs written with bare names go to `claude/<name>`. Rewriting a doc replaces it whole, so send the full content.
- The check-in prompt must carry every ID and date it needs. It cannot see the setup chat.
- The quiz card never reports a score to the tutor. Plan for the student to type it.
- "On track" lags: it only turns to "Behind" after a due date passes. Use the topics-per-day rows for pacing.
- Some sessions happen by voice, where there are no visuals, quiz cards or tools.
- Transcripts contain errors (an old instructor's name, a misspelled star or person). Follow the outline and textbook and tell the student about the mismatch.
