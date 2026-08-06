# Eccles MBA - General Management & Rotational LDP Interview Playbook

A single-file, self-contained HTML interview preparation tool for Eccles Full-Time MBA
students targeting General Management (GM) and Rotational Leadership Development
Program (LDP) interviews — modeled on consumer/retail-focused organizations
(P&G, Target, PepsiCo-style rotational programs).

---

This README is both the **user-facing quick reference** and the **operational guide** for anyone maintaining, editing, or redeploying this tool. 
For a fuller walkthrough of the student-facing experience, see the accompanying `GM_LDP_Playbook_User_Manual.docx`.

---

**▶ Live Tool:** **[Intv Playbook - GM LDP](https://coryjburk.github.io/intv-playbook-gm_ldp/)**
- **▶ User Manual:** **[GM LDP User Manual (DOCX)](https://github.com/coryjburk/intv-playbook-gm_ldp/raw/main/GM_LDP_Playbook_User_Manual.docx)**

---

## 1. What this is
 
- **File:** `Eccles_GM_LDP_Interview_Playbook.html`
- **Type:** Single self-contained HTML file — no build step, no external dependencies,
  no server required.
- **Contents:** 100 interview questions (8 categories), 10 frameworks, an 8-section
  battlecard, 10 red flags, an in-browser practice mode with voice input and
  rules-based (heuristic) AI-style coaching feedback, and a persistent Readiness
  Dashboard that scores each practice attempt and tracks progress over time.
- **Runs:** Directly from disk (`file://`) or from any static web host. No Node,
  Python, or backend of any kind is needed at runtime.
## 2. Repo / file structure
 
```
Eccles_GM_LDP_Interview_Playbook.html   ← the entire tool (HTML + CSS + JS in one file)
GM_LDP_Playbook_User_Manual.docx        ← student-facing Word user manual
README.md                                ← this file
```
 
There is no separate build process. All data (questions, frameworks, battlecard,
red flags) is embedded directly as JavaScript arrays inside a single `<script>` tag
near the bottom of the HTML file.
 
## 3. Tech stack
 
- Vanilla HTML5 / CSS3 / JavaScript (ES6+). No React, no bundler, no npm dependency
  at runtime.
- **Web Speech API** (`SpeechRecognition` / `webkitSpeechRecognition`) powers the
  optional voice-input practice feature. This is a browser-native API — no external
  service or API key is used or required.
- No network calls of any kind are made by the tool itself. It is fully offline-capable
  once the HTML file has loaded.
- Branding uses CSS custom properties defined in `:root` — see Section 6.
## 4. Data schema
 
### 4.1 Questions — `GM_QUESTIONS`
 
Each entry in the `GM_QUESTIONS` array follows this shape:
 
```js
{
  id: 1,                          // sequential integer, unique, never reused
  category: "General Management & P&L Ownership",  // must exactly match one of the 8 categories below
  difficulty: "Foundational",     // one of: "Foundational" | "Core" | "Advanced"
  question: "…",                  // the interview question text
  conversational: "…",            // model spoken-style answer
  deepdive: "…",                  // expanded answer with frameworks/metrics
  coaching: "…",                  // delivery/coaching note
  redFlag: "…"                    // the specific mistake to avoid on this question
}
```
 
**The 8 categories (must match exactly, including punctuation, for filters to group correctly):**
 
1. General Management & P&L Ownership
2. Cross-Functional Leadership & Influence Without Authority
3. Commercial Strategy & Brand/Category Management
4. Supply Chain, Operations & Trade-Off Analysis
5. Business Case & Structured Problem-Solving
6. Change Management & Navigating Ambiguity
7. Consumer Insight & Go-to-Market Strategy
8. Behavioral, Leadership & Career Motivation
Current distribution: 13/12/13/12/13/12/13/12 = 100 questions total.
Difficulty split: 15 Foundational / 64 Core / 21 Advanced.
 
### 4.2 Frameworks — `FRAMEWORKS`
 
```js
{
  title: "P&L / Contribution Margin Waterfall",
  summary: "…",       // one-line description of when to use it
  points: ["…", "…"]  // 3-4 actionable bullet points
}
```
 
### 4.3 Battlecard — `BATTLECARD`
 
```js
{
  title: "Must-Know GM/Rotational Concepts",
  items: ["…", "…"]   // 4-5 bullet points per section
}
```
 
### 4.4 Red Flags — `REDFLAGS`
 
```js
{
  title: "Data Dumping without Strategic Insight",
  desc: "…"  // one-paragraph description
}
```
 
### 4.5 Practice history — `localStorage["gmldp_practice_history_v1"]`
 
Unlike the four datasets above, this isn't baked into the HTML file — it's written by
each student's own browser as they log evaluations, under this key:
 
```js
// Array of attempt records, oldest first
[
  {
    questionId: 1,
    category: "General Management & P&L Ownership",
    difficulty: "Foundational",
    timestamp: 1731000000000,   // ms since epoch
    scores: {                    // each 1-5, entered manually by the student
      acumen: 4,                 // Business Acumen & P&L Judgment
      problemSolving: 4,         // Structured Problem-Solving
      crossFunctional: 4,        // Cross-Functional Leadership & Influence
      strategicJudgment: 4,      // Strategic Judgment & Prioritization
      delivery: 4                // Delivery, Pacing & Presence
    },
    overall: 80,                 // (avg of the 5 scores / 5) * 20 -> scaled to 0-100
    duration: 47,                 // seconds, measured live from first keystroke/recording start
    pace: 132,                    // words per minute, computed from real elapsed time
    fillers: 1
  },
  // …one record per "Save to Dashboard" click tied to a specific question
]
```
 
A new record is appended (never overwritten) each time a student saves an evaluation
on a specific question — so the same question evaluated 3 times produces 3 records,
which is what powers the "progress over time" chart. Free-practice sessions (opened
via the header's "Practice Mode" button with no question selected) can still generate
an evaluation prompt, but **cannot** be saved to this history, since there's no
question ID to attach the record to — the Save button shows an explicit message
instead of silently failing.
 
This is also the exact array that Export downloads as JSON and Import reads back in
— see Section 6.5.
 
## 5. The evaluation workflow (AI-graded, not heuristic)
 
This tool does not call a live AI model to grade answers — it's a static file with no
backend and no API key. Instead, it generates a complete, ready-to-paste evaluation
prompt and asks the student to run it through a real model (Claude, ChatGPT, or
whatever they have access to) themselves, then transcribe the 5 scores back in.
 
**Why this instead of client-side heuristic scoring:** an earlier version of this tool
computed a 0-100 score client-side from word count, filler-word rate, and keyword
overlap with the model answer. That approach is free and instant, but it isn't a real
evaluation of interview quality — testing surfaced cases where a genuinely strong
answer scored lower than a weak one, or vice versa, simply because of vocabulary
overlap quirks. Rather than show students a number that looks authoritative but can
be wrong, the dashboard now only reflects scores a real model actually produced, that
a human read and transcribed. The tradeoff is one extra copy/paste step per
evaluation — worth it for score integrity.
 
### 5.1 What happens automatically (no AI needed)
 
Three metrics are measured live and require no model call, since they're objectively
computable: **Duration** (elapsed time since the student's first keystroke or the
start of recording), **Pace** (words ÷ elapsed minutes — real WPM, not estimated), and
**Filler Words** (see Section 8 for the exact word list). These feed directly into the
generated prompt so the external AI can factor delivery into its feedback.
 
### 5.2 The generated prompt
 
`buildEvaluationPrompt()` in the `<script>` block assembles a prompt from: a fixed
role/instruction preamble, the question's `category` and `question` text, the live
delivery metrics, the student's answer, and a strict output-format instruction listing
all 5 rubric dimensions so the model's response is easy to read scores back from. The
"Copy Prompt" button copies this exact text to the clipboard via
`navigator.clipboard.writeText()`, with a `document.execCommand('copy')` fallback for
browsers that block the Clipboard API in this context.
 
### 5.3 The 5-dimension rubric
 
- **Business Acumen & P&L Judgment**
- **Structured Problem-Solving**
- **Cross-Functional Leadership & Influence**
- **Strategic Judgment & Prioritization**
- **Delivery, Pacing & Presence**
Each is entered on a 1-5 scale via a dropdown (`RUBRIC_DIMENSIONS` in the `<script>`
block — edit the `label` values there to change the rubric; the `key` values are the
storage field names and changing them will orphan any already-saved history). All 5
must be filled before "Save to Dashboard" will accept the entry — partial submissions
are rejected with an inline message rather than silently saved as zeros.
 
`Overall` = `(sum of the 5 scores / 5) * 20`, mapped to the same band system used
throughout the dashboard: Interview Ready (80+), Solid Progress (60-79), Developing
(40-59), Needs Work (<40).
 
## 6. How to make common edits
 
### 6.1 Add or edit a question
 
1. Open the HTML file in a text editor and locate the `GM_QUESTIONS` array inside
   the `<script>` tag.
2. Add a new object following the schema in Section 4.1. Use the next sequential
   `id` (currently 1–100, so a new question would be `id: 101`).
3. The `category` string must exactly match one of the 8 existing category names,
   or it will create an unintended 9th filter chip (the category chip list is
   generated automatically from whatever values appear in the data — see Section 6.3).
4. Save and reopen the file in a browser to confirm it renders correctly.
### 6.2 Add a new framework, battlecard section, or red flag
 
Same pattern — add a new object to `FRAMEWORKS`, `BATTLECARD`, or `REDFLAGS`
following the shapes in Sections 4.2–4.4. These sections render dynamically from
the array, so no other code changes are needed.
 
### 6.3 Important: category chips are auto-generated, hero stats are not
 
The category filter chips and the difficulty filter chips are built dynamically
from whatever values exist in `GM_QUESTIONS` — you do not need to edit the filter
UI separately when adding a category.
 
**However**, the hero section stat tiles (100 / 8 / 10 / 8 / 10) near the top of the
page are hardcoded text, not calculated from the data. If you add or remove
questions, frameworks, battlecard sections, or red flags, update the corresponding
`<div class="num">` values in the hero section manually.
 
### 6.4 Editing the standard footer / branding
 
- Footer text and copyright line live in the `<footer class="site">` block near
  the end of the HTML body — this follows the standard playbook footer used across
  the repo family (Cory Burk / Full-Time MBA Program / David Eccles School of Business).
- Brand colors are defined once in `:root` at the top of the `<style>` block:
  `--red: #CC0000` (primary/Eccles red) and `--gold: #c9a84c` (accent). Change these
  two variables to re-theme the entire tool consistently.
### 6.5 Export / Import / Reset
 
These three buttons live in the Readiness Dashboard header (`.dashboard-actions`):
 
- **Export** (`exportHistory()`) — serializes `practiceHistory` to JSON and triggers
  a browser download named `gmldp-readiness-backup.json`, via a `Blob` + temporary
  `<a download>` element. No server round-trip.
- **Import** (`triggerImport()` / `handleImportFile()`) — opens a hidden
  `<input type="file">`, reads the selected file with `FileReader`, validates it's a
  JSON array, and **replaces** `practiceHistory` entirely (with a confirm dialog if
  there's existing data that would be overwritten). This is a restore/transfer
  operation, not a merge — the framing is "back it up or move it to another device,"
  so replace is the correct semantic; a merge option would need explicit dedup logic
  against `questionId` + `timestamp` if ever added.
- **Reset** (`resetDashboard()`) — clears `practiceHistory` entirely after a confirm
  dialog. Separate from the in-modal "Reset" button, which only clears the current
  practice session (transcript, timer, dropdown selections) and does not touch saved
  history.
## 7. Hosting / deployment
 
This file needs no build step and no server-side logic. Options:
 
- **Direct file share:** send the `.html` file directly — recipients double-click
  to open it locally.
- **Static hosting:** upload as-is to GitHub Pages, Netlify, an S3 static site, or
  any web server. No configuration beyond serving the file is required.
- **HTTPS recommendation:** the voice input feature (Web Speech API) generally
  requires either `localhost` or an HTTPS origin to request microphone access in
  most browsers. Typed practice always works regardless of hosting method.
## 8. Browser compatibility & known limitations
 
| Feature | Chrome / Edge | Safari | Firefox |
|---|---|---|---|
| All content, filters, frameworks, battlecard | ✅ | ✅ | ✅ |
| Typed practice + live metrics + evaluation prompt | ✅ | ✅ | ✅ |
| Voice input (Web Speech API) | ✅ | ⚠️ Limited/inconsistent | ❌ Not supported |
| Copy Prompt (Clipboard API, with execCommand fallback) | ✅ | ✅ | ✅ |
| Readiness Dashboard persistence (localStorage) | ✅ | ✅ | ✅ (fails silently in private/incognito modes on some browsers) |
 
**Known limitations, by design:**
 
- **Grading requires a manual round-trip through an external AI.** This file has no
  backend and no API key, so it cannot evaluate an answer itself. The student copies
  a generated prompt into Claude or ChatGPT, reads 5 scores off the response, and
  types them back in. This is one extra step versus instant scoring, but it means
  every saved score reflects a real model's judgment rather than a keyword-matching
  heuristic — see Section 5 for the full reasoning behind this tradeoff.
- **Nothing stops a student from entering scores without actually running the
  prompt.** The dropdowns accept any 1-5 value regardless of whether an AI was
  consulted. This tool trusts the student to use it honestly, the same way a paper
  self-assessment would; it is a practice aid, not a proctored exam.
- **Duration/Pace timing starts on first keystroke or first recording start, per
  practice session.** Switching questions or clicking the modal's own "Reset"
  restarts the timer; there's no pause/resume.
- **Filler-word detection covers "um," "uh," "like," "you know," "basically,"
  "actually," "kind of," "sort of," and "i mean."** It deliberately does not track
  "so" or "right" as fillers — both are too common as legitimate conjunctions/adjectives
  to detect reliably by keyword matching alone, and would produce far more false
  positives than useful signal.
- **Readiness Dashboard progress is saved via `localStorage`, scoped to one
  browser on one device.** It does not sync across devices automatically — use
  Export/Import (Section 6.5) to move progress between devices or back it up.
  If a browser blocks local storage entirely (e.g. some private/incognito modes),
  the tool detects this and shows a warning directly in the dashboard header so
  students aren't surprised when progress doesn't survive closing the tab.
- **Free-practice mode cannot save to the dashboard.** Opening practice via the
  header's "Practice Mode" button (with no question pre-selected) still generates
  a working evaluation prompt, but "Save to Dashboard" shows an explicit message
  instead of saving, since there's no question ID to attribute the record to.
- **Import replaces, it doesn't merge.** Importing a backup file overwrites the
  current device's saved history entirely (with a confirmation prompt first). If
  you want merge semantics later, that needs explicit dedup logic against
  `questionId` + `timestamp` — see Section 6.5.
## 9. Testing / QA procedure
 
Before shipping any content or code change, run the automated interaction test
suite (Playwright + headless Chromium). Two test files cover this tool:
 
**Core interaction suite** (33 checks):
- Data integrity (correct counts across all 4 datasets; every question has all
  4 non-empty answer fields)
- Sidebar navigation (all 5 panels show/hide correctly, no overlap)
- Category, difficulty, and search filtering (exact expected result counts)
- Question card expand/collapse and all 4 answer tabs
- Practice modal open (both entry points) / close
- Mic button behavior (graceful no-crash fallback when unsupported)
- Zero JavaScript console errors or uncaught exceptions across the full run
**Evaluation/scoring/dashboard suite** (36 checks):
- Empty-state rendering (ring shows "Getting Started," all 8 domains "Not started")
- All 5 rubric dropdowns render with the exact approved labels
- Live Duration/Pace/Filler-word tiles update in real time
- The generated prompt includes the real question, category, live delivery
  metrics, the student's actual answer text, and all 5 rubric dimension lines
- Copy Prompt actually writes the exact prompt text to the clipboard
- Save to Dashboard is rejected with a clear message when any of the 5 scores
  are left blank, and succeeds once all 5 are filled
- Free-practice mode is blocked from saving, with an explicit message
- The dashboard ring, domain breakdown, and Question Bank's top readiness bar
  all update correctly and stay in sync after a save
- The in-modal Reset clears transcript/timer/dropdowns without touching saved history
- **Export produces a valid, correctly-named JSON file containing the real saved
  data** (verified via Playwright's download interception, not just a click)
- **Import correctly restores an exported backup**, including the confirm-before-
  overwrite prompt
- **Practice history persists across a full page reload** (proves `localStorage`
  is actually working, not just in-memory JS state)
- Dashboard-level Reset (with its confirm dialog) clears everything, and the
  cleared state itself persists after another reload
Re-run both suites after any edit to the HTML file's data or script — a single
malformed object (a missing comma, an unescaped quote in question text) is enough
to silently break the whole tool, since it's all one `<script>` block. The
persistence, export/import, and prompt-generation checks are the ones most likely
to break silently during a well-intentioned refactor, so don't skip them for
changes that "only touch the UI."
 
## 10. Version history
 
- **v2.1 (August 2026)** — Fixed a duplicate-save bug: clicking "Save to Dashboard"
  multiple times in a row (e.g. accidental double/triple-click) logged a separate
  attempt record each time, inflating the total evaluations count and corrupting
  the Score Progress Over Time chart with repeated identical data points. The
  button now disables itself and relabels to "Saved ✓" immediately after a
  successful save, and only re-enables when the student clicks the in-modal
  "Reset" button or closes and reopens the practice modal — both of which already
  clear the transcript and timer, so re-enabling save at the same moments is the
  correct behavior, not an extra rule to remember.
- **v2.0 (August 2026)** — Replaced client-side heuristic scoring with a real
  AI-graded evaluation workflow, matching the design of the companion PMM
  interview playbook:
  - Removed the old 3-dimension heuristic score (Content/Structure/Delivery) and
    its qualitative "What Worked / Top 2 Improvements / Power Move" text, both
    computed entirely from keyword overlap and word counts.
  - Added a generated, copy-pasteable evaluation prompt (`buildEvaluationPrompt()`)
    that a student runs through Claude or ChatGPT themselves, plus a 5-dimension
    rubric (Business Acumen & P&L Judgment / Structured Problem-Solving /
    Cross-Functional Leadership & Influence / Strategic Judgment & Prioritization /
    Delivery, Pacing & Presence) entered manually via dropdowns after reading the
    AI's response.
  - Added live Duration and Pace (real WPM from elapsed time, not estimated)
    metric tiles alongside the existing filler-word count.
  - Redesigned the Readiness Dashboard: a circular "Overall Readiness" progress
    ring replaces the old stat-card grid; category bars renamed "Domain Mastery"
    or now driven by real evaluation scores.
  - Added Export (downloads practice history as JSON) and Import (restores a
    backup, with confirm-before-overwrite) to the dashboard header, addressing
    the "doesn't sync across devices" gap from v1.x.
  - Rationale for keeping metrics separate from scoring: duration, pace, and
    filler-word count are objectively measurable without AI and cost nothing to
    show instantly; interview-quality scoring is not, and showing a heuristic
    number next to a real AI-graded one would undermine trust in both.
- **v1.2 (August 2026)** — Added the (now-removed) scoring and Readiness Dashboard
  feature: heuristic 0-100 scores computed client-side, a category breakdown, and
  a score-over-time chart. Superseded by v2.0 above.
- **v1.1 (August 2026)** — Bug-fix pass following an automated regression sweep:
  - Fixed a regex escaping bug that made the filler-word tracker always return 0
    regardless of input.
  - Fixed a false-positive regression from the first fix: filler detection was
    briefly substring-based (no word-boundary check), so words like "unlike,"
    "likely," and "factually" incorrectly counted as containing the filler words
    "like" and "actually." Restored proper word-boundary regex matching.
  - Removed "so" and "right" from the filler-word list entirely. Both are too
    common as legitimate conjunctions/adjectives to detect reliably by keyword
    matching, and Web Speech API transcripts rarely include the punctuation
    that would help disambiguate filler use from normal use.
  - Fixed a mobile layout bug: the sticky header had no responsive handling and
    overflowed horizontally below ~640px width. Added a mobile breakpoint that
    wraps the header, hides the subtitle and track pill on small screens, and
    keeps the practice button accessible.
- **v1.0 (August 2026)** — Initial release. 100 questions / 8 categories / 10
  frameworks / 8-section battlecard / 10 red flags. Built from the same prompt
  template and methodology as the Finance LDP playbook, retargeted for
  consumer/retail GM & rotational leadership programs.
---
 
Developed by Cory Burk, Senior Manager, Program Management · Full-Time MBA Program ·
David Eccles School of Business.
 
© 2026 University of Utah, David Eccles School of Business. All rights reserved.
 
