# SmartEx User Study – Smart Office

This repository contains the online user study for my bachelor's thesis at the University of Cologne. The thesis looks at **LLM-generated contrastive explanations** for a smart office: explanations that answer questions like *"Why is the status light orange and not green?"*

The whole study is **one web page** (`index.html`). Participants open the link, take the role of Alice or Bob, and go through three situations in a small smart office.

**Study link:** https://mgobsch.github.io/thesis-study/

---

## What participants do

The study takes about 15–20 minutes and works in English and German (switch at the top right).

1. **Welcome and consent**
2. **A few questions about you:** age, familiarity with smart devices, technical background
3. **Introduction to the smart office:** the meeting room, the status light, the smart office system and SmartEx (the explanation assistant)
4. **The smart devices:** participants click on 7 devices (door sensor, motion sensor, status light, forecast, clock, smoke/CO₂ detector, fan)
5. **Door demo:** a slightly open door shows whether people are inside the meeting room
6. **The office rules:** all 6 rules that change the status light. From here on, the yellow **office note** (cheat sheet) is always available.
7. **Alice and Bob:** the two personas and their technical level
8. **Quick check:** 3 questions to make sure the setup is understood
9. **Three situations (random order),** each with three steps:
   - **Story:** what the persona sees at that moment
   - **Ask SmartEx:** the participant asks why the light looks the way it does, and ranks the rules by what they expected instead (drag and drop, then "Lock in my order")
   - **Rate the explanations:** two or three explanations (A, B and C, depending on `VARIANT`) are rated on clarity, understandability, helpfulness, naturalness and trust, followed by a preference and a comprehension question
10. **Final questions:** attention check and optional feedback
11. **End:** answers are sent to the Google Sheet

### The three situations

| Situation | Persona | What happens | Status light |
|---|---|---|---|
| TC1 | Alice | 12:08, rain. A meeting just ended, people are standing in the hallway, the door is closed. | orange |
| TC2 | Bob | 12:10, rain, lunchtime. Bob's "Rain at Lunch" rule should turn the light blue. People are standing in the hallway, the door is closed. | orange |
| TC3 | Alice | 08:00, sunny. Alice arrives in the morning, the door is slightly open, the room is empty. | off |

### The explanations

In each situation, participants rate the explanations in random order (they only see "A", "B" and "C"):

- **C0:** the SmartEx baseline explanation (reworded for readability)
- **C2:** GPT-OSS + prompt P1 (zero-shot) + configuration C2 (LLM-weighted TOPSIS)
- **C3:** GPT-OSS + prompt P1 (zero-shot) + configuration C3 (LLM foil selection)

The setting `VARIANT` decides which ones are shown (same for all participants):

| `VARIANT` | Explanations per situation |
|---|---|
| `"C2_C3"` (default) | C0 + C2 + C3 (three) |
| `"C2"` | C0 + C2 (two) |

The same trial is used for every participant. Only trials without hallucinations were chosen:

| Situation | C2 | C3 |
|---|---|---|
| TC1 | Trial 2 | Trial 1 |
| TC2 | Trial 2 | Trial 1 |
| TC3 | Trial 1 | Trial 3 |

The German explanations are DeepL translations of the English originals.

---|---|---|
| TC1 | Trial 3 | Trial 2 |
| TC2 | Trial 1 | Trial 1 |
| TC3 | Trial 1 | Trial 2 |

The German explanations are translations of the English originals.

---

## Files in this repository

| File | What it is |
|---|---|
| `index.html` | The complete study: design, texts, logic and saving |
| `README.md` | This file |

The Google Apps Script (`Code.gs`) that receives the answers is not stored here. It lives in the Google Sheet under **Extensions → Apps Script**.

---

## How the answers are saved

```
Participant's browser  ──►  Google Apps Script (web app)  ──►  Google Sheet
     (index.html)             receives the answers             stores one row per person
```

- The page sends the answers to the web-app URL set in `SAVE_URL`.
- A row is written **after each situation** (so partial answers are kept if someone stops), and the same row is **updated at the end**.
- The Google Sheet has two tabs:
  - **Antworten:** one row per participant with all values in columns, ready for analysis
  - **Rohdaten:** the complete raw data as JSON (backup)
- If a participant closes the tab by accident, they can continue where they left off when they open the link again in the same browser.
- If sending fails at the end, a "Try again" button appears.

To analyse the data in Excel: open the Google Sheet → **File → Download → Microsoft Excel (.xlsx)**.

### Important columns (tab "Antworten")

| Column | Meaning |
|---|---|
| `pid` | anonymous participant ID |
| `finished` | TRUE = completed the whole study |
| `order` | order of the situations, e.g. "TC2 TC1 TC3" |
| `quiz_correct` | correct answers in the quick check (0–3) |
| `attention_passed` | attention check passed (TRUE/FALSE) |
| `TC1_rank_locked` | the locked ranking, e.g. "notoccupied > rain" |
| `TC1_rank_first` | the rule ranked first (the expected foil) |
| `TC1_rank_moves` | how often tiles were moved |
| `TC1_A_is`, `TC1_B_is`, `TC1_C_is` | which explanation was shown as A, B and C (C0, C2 or C3) |
| `TC1_C2_trial`, `TC1_C3_trial` | which GPT-OSS trial was used (C3 empty in variant "C2") |
| `variant`, `model` | which variant was running, and the model (GPT-OSS P1) |
| `TC1_preferred` | preferred explanation: **C0**, **C2** or **C3** |
| `TC1_C0_clear` … `TC1_C3_trust` | Likert ratings 1–5, already mapped to C0, C2 and C3 |
| `TC1_mcq_correct` | comprehension question correct (TRUE/FALSE) |
| `TC1_time_rank_s`, `TC1_time_rate_s` | seconds spent on the ranking and rating pages |
| `total_time_min` | total duration in minutes |

The same columns exist for TC2 and TC3.

**Rule IDs:** `notoccupied` = Meeting Room Not Occupied, `occupied` = Meeting Room Occupied, `rain` = Rain at Lunch, `sunny` = Sunny at Lunch, `danger` = Danger, `closing` = Closing Time.

---

## Quality and bot checks

The page measures how long each screen takes and adds warning flags to every row. **Nobody is excluded automatically**; the flags only help you decide later.

| Column | Meaning |
|---|---|
| `time_<screen>_s` | seconds on each screen, e.g. `time_devices_s`, `time_TC1_rank_s`, `time_TC1_rate_s` (adds up if someone goes back) |
| `TC1_rate_min_reading_s` | minimum time needed to read the explanations on that rating page at `READING_WPM` |
| `TC1_straightlining` | TRUE = all ratings in this situation have the same value |
| `flag_too_fast` | whole study faster than `MIN_TOTAL_MINUTES`, or a rating page faster than its minimum reading time |
| `flag_straightlining` | straightlining in at least one situation |
| `flag_no_pointer` | no mouse, touch or scroll activity at all |
| `flag_webdriver` | the browser reports that it is remote-controlled (typical for bots) |
| `flag_honeypot` | an invisible field on the "About you" page was filled in (people can't see it) |
| `pointer_moves` | number of mouse/touch/scroll events |
| `bot_score` | number of flags that apply (0 = unremarkable) |

Thresholds are set in `SETTINGS`: `MIN_TOTAL_MINUTES` (default 5) and `READING_WPM` (default 300). Decide before data collection which rule leads to exclusion (for example: attention check failed or `bot_score` ≥ 2) and report it in the method section.

---

## How to change things

All content is in clearly marked blocks at the top of the `<script>` part of `index.html`. To edit on GitHub: open `index.html` → click the **pencil icon** → make the change → **Commit changes**. The live page updates after about 1 minute.

| Block | What you can change there |
|---|---|
| `SETTINGS` | test mode, save URL, which rule context is shown |
| `STATS` | how often each rule was fired / explained (past 90 days) |
| `RULES` | rule names, "Starts when" / "Only if" texts, owners |
| `SCENARIOS` | stories, time, weather, door, candidate foils, the explanations, comprehension questions |
| `T` | every other text on the page, in English (`en`) and German (`de`) |

### Settings

| Setting | Values | Meaning |
|---|---|---|
| `TEST_MODE` | `true` / `false` | `true` shows the collected data on the end screen. **Set to `false` before the real study.** |
| `SAVE_URL` | web-app URL | where answers are sent (Google Apps Script) |
| `SURVEY_RETURN_URL` | URL or `""` | optional button on the end screen, e.g. to a follow-up survey |
| `VARIANT` | `"C2_C3"` / `"C2"` | three explanations (C0 + C2 + C3) or two (C0 + C2) |
| `MIN_TOTAL_MINUTES` | number | minimum total duration; faster → `flag_too_fast` |
| `READING_WPM` | number | reading speed used for the rating-page check |
| `CONTEXT_MODE` | `"basic"` / `"all"` / `"split"` | `basic` = owners only in the office note; `all` = also how often rules were fired and explained; `split` = random 50/50 per participant |

### Editing tips

- Only change text **inside the quotation marks** `"…"`. Keep every comma, quote mark and bracket.
- Quotation marks inside a text must be typographic („…“ or “…”), not plain `"`.
- After every change, open the study link once and click through the changed part.
- Avoid changing texts during data collection. If you have to, write down the date so you know which participants saw which version.

---

## Before the real study (checklist)

- [ ] Google Sheet and Apps Script set up, web-app access set to **"Anyone"**
- [ ] One complete test run through the study link, row appears in the Google Sheet
- [ ] Test rows deleted from both tabs (keep the header row)
- [ ] `TEST_MODE` set to `false`
- [ ] Data protection and consent text checked with the supervisor
- [ ] Pilot with 1–2 people, feedback included
- [ ] Link sent to participants, with a note that a **laptop** works best

---

## Randomisation

To avoid order effects, the page randomises per participant (and saves it):

- the order of the three situations
- which explanation is shown as A, B and C
- the starting order of the rule tiles
- the order of the comprehension-question answers

---

## Data protection

- No names, e-mail addresses or IP addresses are collected by the page. Each participant gets a random ID.
- Answers are stored in a Google Sheet (Google servers). This should be mentioned in the consent text (to do, after checking with the supervisor).
- The page itself is hosted on GitHub Pages and loads its font from Google Fonts.

---

## Technical notes

- One self-contained HTML file: plain HTML, CSS and JavaScript, no build step, no framework.
- Works in current versions of Chrome, Firefox, Safari and Edge, on laptops and phones.
- The office drawing is inline SVG, so no image files are needed.
- Progress is stored in the browser's `localStorage` and removed after the study is finished.

