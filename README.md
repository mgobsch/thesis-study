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
   - **Rate the explanations:** two explanations (A and B) are rated on clarity, understandability, helpfulness, naturalness and trust, followed by a preference and a comprehension question
10. **Final questions:** attention check and optional feedback
11. **End:** answers are sent to the Google Sheet

### The three situations

| Situation | Persona | What happens | Status light |
|---|---|---|---|
| TC1 | Alice | 12:08, rain. A meeting just ended, people are standing in the hallway, the door is closed. | orange |
| TC2 | Bob | 12:10, rain, lunchtime. Bob's "Rain at Lunch" rule should turn the light blue. People are standing in the hallway, the door is closed. | orange |
| TC3 | Alice | 08:00, sunny. Alice arrives in the morning, the door is slightly open, the room is empty. | off |

### The two explanations

In each situation, participants compare two explanations in random order (they only see "A" and "B"):

- **C0:** the original SmartEx template explanation (baseline)
- **WIN:** the explanation from the selected setup, GPT-OSS + prompt P1 (zero-shot) + configuration C2 (LLM-weighted TOPSIS)

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
| `TC1_A_is` | which explanation was shown as "A" (C0 or WIN) |
| `TC1_preferred` | preferred explanation: **C0** or **WIN** |
| `TC1_C0_clear` … `TC1_WIN_trust` | Likert ratings 1–5, already mapped to C0 and WIN |
| `TC1_mcq_correct` | comprehension question correct (TRUE/FALSE) |
| `TC1_time_rank_s`, `TC1_time_rate_s` | seconds spent on the ranking and rating pages |
| `total_time_min` | total duration in minutes |

The same columns exist for TC2 and TC3.

**Rule IDs:** `notoccupied` = Meeting Room Not Occupied, `occupied` = Meeting Room Occupied, `rain` = Rain at Lunch, `sunny` = Sunny at Lunch, `danger` = Danger, `closing` = Closing Time.

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
- which explanation is shown as A and which as B
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

---

## Optional: three explanations in TC2

The page contains an optional third explanation for TC2: **WIN2**, GPT-OSS + P1 + C2 **Trial 2**. It is faithful but uses the foil "Meeting Room Not Occupied" (it explains "why not green"), while WIN (Trial 1) answers "why not blue" but contains an invented priority claim.

- Switch: `TC2_THREE_EXPLANATIONS` in `SETTINGS` (`false` = off, `true` = on).
- When on, TC2 shows explanations A, B and C in random order, with the same ratings for each and a 3-option preference question.
- Extra columns: `tc2_three`, `TC2_C_is`, `TC2_WIN2_clear` … `TC2_WIN2_trust`.
- Switch it on **before** data collection starts, not during it.
