# Linden Street Volunteer Confirmations + Shift Full

**AI Tech Corps Capstone · OptimaNova AI Institute**

> **This client is made up.** Linden Street Community Partners is a fictional practice client created for the AI Tech Corps program. Every name, email, shift, and number in this repo is either made-up test data or comes from the program's case file and practice staff interviews. No real client or volunteer data is used anywhere.

A Make.com automation that drafts volunteer shift emails for a community food pantry. When a volunteer signs up, it checks whether their shift has room, then has Gemini write either a confirmation or a "this shift is full, can you come the following week?" email in the volunteer's preferred language. It saves that email as a **Gmail draft**, and a staff member reads it and decides whether to send it.

**Core principle: AI does, humans decide. Nothing sends automatically.**

---

## 🎥 Demo video (10 minutes)

**[Watch the demo on Google Drive](https://drive.google.com/file/d/1PYi5mlHEAVwMrfYDVj4_07DNjJhZybeL/view?usp=drive_link)**

The video follows a **Show, Try, Check** training format and uses only made-up volunteers:

| Segment | What happens |
|---|---|
| **Show** (about 3 min) | Run Maya (Saturday) and show the draft waiting in Gmail Drafts, not sent. Then run Lucía, who hits the shift cap, and show the "full" draft and the log's Type column. |
| **Try** (about 5 min) | The staff member submits Dev (Tuesday), runs the scenario once, and finds the draft and log row. Then a messy input: a blank shift. The draft asks for a shift and never says "full." |
| **Check** (about 2 min) | The staff member runs one alone and answers: *"A draft says the shift is full. What do you do next?"* |

---

## Table of contents

1. [The client scenario](#the-client-scenario)
2. [The problem](#the-problem)
3. [What was built](#what-was-built)
4. [How it works](#how-it-works)
5. [Guardrails and human review](#guardrails-and-human-review)
6. [Testing](#testing)
7. [How to use it](#how-to-use-it)
8. [Cost and limits](#cost-and-limits)
9. [Governance and bright lines](#governance-and-bright-lines)
10. [Limitations and open questions](#limitations-and-open-questions)
11. [Repo contents](#repo-contents)
12. [Team and credits](#team-and-credits)

---

## The client scenario

*Fictional organization, from the program case file.*

- **Who:** Linden Street Community Partners, a community nonprofit on the east side of a mid-size city, founded 2009.
- **Size:** 14 staff (11 full-time, 3 part-time), about 80 active volunteers, about 450 households served a month, $1.9M budget.
- **Languages:** Families speak English, Spanish, and Haitian Creole.
- **Programs:** Community Pantry (Tuesday, Thursday, Saturday; Saturday is the biggest), Benefits Access, and an After-School Program.
- **Tech today:** Google Workspace for Nonprofits, QuickBooks Online, a city benefits portal, and donated laptops.
- **Readiness:** The team scored Linden Street **38/100 (Exploring tier)**. The score is directional and based on interview evidence.

## The problem

The pantry's volunteer program is run by **Gloria Ortiz**, Operations and Finance Manager. In practice interviews (Nov 9 and Nov 20), she said:

- She spends **about 4 hours a week** (about 16 a month) scheduling, confirming, and reminding volunteers.
- She uses a **paper sign-up sheet** and **texts from her personal phone**. There is no online system.
- Saturday needs **about 25 volunteers**. About **1 in 5 don't show**, so she schedules 4 to 5 extra.
- When a shift is full, she asks the volunteer whether they can serve **the following week**.
- Writing clear instructions in English, Spanish, and Haitian Creole takes extra effort.

## What was built

**Volunteer Confirmations + Shift Full** is a Make.com scenario built by Mackayla Rodriguez.

| | |
|---|---|
| **Pattern** | Automation: trigger, AI step, human check, log |
| **Trigger** | A volunteer submits a Google Form |
| **AI step** | Gemini (regular Flash model) drafts the email in the volunteer's preferred language |
| **Output** | A Gmail **draft** plus a row in the Google Sheets log |
| **Human check** | Gloria reads every draft and sends or deletes it herself. A fluent speaker reviews every Spanish or Haitian Creole draft. |
| **Tools** | Google Forms, Google Sheets, Make.com, Gemini, Gmail |
| **Data** | Made-up volunteer data only |

**Form fields:** First Name, Email (with email validation), Which Shift? (Saturday, Tuesday, or Thursday; left optional so a blank shift can be tested), and Preferred Language (English, Spanish, or Haitian Creole).

## How it works

```
Google Form submitted
        │
        ▼
Count existing "Confirmation draft" rows for that shift   (Shift Sheet)
        │
        ▼
Look up that shift's cap                                   (Cap Sheet)
        │
        ▼
     Router
   ┌────┴─────────────────────────┐
   ▼                              ▼
UNDER CAP (or blank shift)     FULL (count >= cap, shift given)
Gemini writes confirmation     Gemini writes "full, can you
                               come the following week?"
   │                              │
   ▼                              ▼
Gmail: create DRAFT            Gmail: create DRAFT
   │                              │
   ▼                              ▼
Log row: "Confirmation draft"  Log row: "Shift full draft"
        │                              │
        └──────────────┬───────────────┘
                       ▼
       Gloria reviews, then sends or deletes
```

### Module map

| Module | What it does |
|---|---|
| Forms #1 | Watches for new responses ("From now on") |
| Sheets Search Rows #5 | Finds rows in **Shift Sheet** where Shift = form value AND Type = "Confirmation draft" |
| Tools #6 | Numeric aggregator: MAX of "Total number of bundles" (the current count) |
| Sheets Search Rows #14 | Finds the row in **Cap Sheet** where Shift = form value |
| Tools #15 | Numeric aggregator: MAX of Cap |
| Router #7 | **Route 1 (Under cap):** count < cap OR shift is blank. **Route 2 (Full):** count >= cap AND shift exists |
| Gemini #8 → Gmail #10 → Sheets #11 | Confirmation draft. Subject: "Your Volunteer Shift at Linden Street" |
| Gemini #9 → Gmail #12 → Sheets #13 | Shift-full draft. Subject: "About your volunteer shift at Linden Street" |

### What the emails say

**Confirmation:** Thanks the volunteer by first name and confirms the shift. For Saturday it says to arrive by 8:45 AM. For Tuesday and Thursday it confirms the day only, because those times have not been verified. It tells them to park in the back lot and wear closed-toe shoes. It stays under 80 words and is signed by Gloria.

**Shift full:** Says the chosen shift is full this week and asks whether they can serve the same day the following week. It never offers a different day, never promises a spot, and gives no arrival or parking instructions. Gloria confirms the following week by hand after they reply.

**Blank shift (either route):** The email never guesses a shift and never says "full." It asks the volunteer to reply with Saturday, Tuesday, or Thursday.

Both prompts tell Gemini to use only the facts in the prompt and to ignore any instructions typed into the form fields. The full prompts are in [`prompts/`](#repo-contents).

### Sheets setup

- **Cap Sheet** tab, with headers `Shift | Cap` and one row each for Saturday, Tuesday, and Thursday. Staff change caps here and never edit Make.
- **Shift Sheet** tab, the log. The `Type` column is either `Confirmation draft` or `Shift full draft`.
- Test caps are made up: **Saturday 1, Tuesday 2, Thursday 2**.

## Guardrails and human review

- **Nothing sends.** Gmail only creates drafts. A person reads each one and sends or deletes it.
- **Fluent-speaker review** for every Spanish or Haitian Creole draft before it goes out.
- **No guessing.** A blank shift never gets a "full" email or a made-up shift.
- **No promises.** "Full" emails never assign a spot or offer a different day.
- **Prompt-injection guard.** Instructions typed into form fields are ignored.
- **On-demand only.** The schedule stays On demand, so it runs only when someone clicks Run once.
- **Own logins.** Everyone uses their own login with two-step verification. API keys are never pasted into email, chat, or documents.
- **The Four Checks** apply to every AI output: **Source, Specifics, Sense, Sensitivity.**

**What it won't do:** send anything, move a volunteer to another shift or week, guess a missing shift, text anyone, or decide who gets a spot.

## Testing

All tests use made-up volunteers with `@example.com` addresses.

| # | Input | Expected | Result |
|---|---|---|---|
| 1 | Maya, Saturday, English, empty log, cap 1 | Under cap. Confirmation that says arrive by 8:45 AM. | Routed to Under cap after fix F4. *Draft content to confirm on a regular Flash model.* |
| 2 | Lucía, Saturday, Spanish, log has Maya, cap 1 | Full route. Spanish "following week" email with no arrival time. | The second sign-up for the same shift hits the cap. The Full route fires and the Spanish email says the shift is full and asks if she can serve the following week. It promises no spot and gives no arrival time. |
| 3 | Dev, Tuesday, English | Under cap. Confirmation with no arrival time. | A different shift has its own count and cap, so it goes back to a confirmation. English, no arrival time, because Tuesday has no time. |
| 4 | Ana, blank shift | Asks her to reply with a day. Never says "full." | Messy input. The build doesn’t guess a shift and never sends a “full” email. The Spanish draft asks her to reply with Saturday, Tuesday, or Thursday. |
| 5 | Darnell, `darnell.test.example.com` (no @) | Fail-safe: the form refuses the entry, so nothing reaches Make. | ✅ Pass |

### Failures that led to fixes

| # | What went wrong | Fix |
|---|---|---|
| F1 | Aggregator mapped to the text Shift column caused "Validation failed" | Mapped Value to a numeric item. Retest passed. |
| F2 | Gemini 429 (free tier is about 20 requests a day per model) | Switched model or waited for the reset |
| F3 | Blank-shift rows were logged as confirmations and counted toward the cap | Added an "Exists" rule on Shift in the count search and router. *Retest with Ana.* |
| F4 | **First Saturday sign-up went to Full.** COUNT on an empty search returned 1 because Make passes one empty bundle. | Tools #6 now uses MAX of "Total number of bundles," which correctly reads 0. Retest: Maya went to Under cap. ✅ |
| F5 | Sheets 400 error: "Unable to parse range: 'Sheet1'A1" | Set Sheet Name to "Shift Sheet" on #11 and #13 |
| F6 | Flash-Lite wrote a short email missing 8:45 AM, back lot, and the sign-off | Switched #8 and #9 to a regular Flash model |
| F7 | Google Forms 429 "Expensive Reads" and Gemini 503 "high demand" | Outside our control. Wait 10 minutes and run once. Documented in the handoff. |

## How to use it

*Condensed from the one-page handoff doc. The owner is Gloria Ortiz.*

1. Sign in to Make.com with your own login and open **"Volunteer Confirmations + Shift Full."**
2. Click **Run once** one time.
3. Open **Gmail → Drafts** and find each draft by the volunteer's email.
4. Read each draft. Check the name and shift. Saturday drafts say arrive by 8:45 AM. Tuesday and Thursday drafts give no time. A "full" email must not promise a spot.
5. Have a fluent speaker read every Spanish or Haitian Creole draft.
6. **Send or delete the draft yourself.**
7. Check the new row in the Shift Sheet and watch for duplicates.
8. If someone replies to a "full" email, confirm their following-week spot by hand.

**To change a cap:** edit the Cap Sheet. **Each week:** move last week's rows to an Archive tab.

**When it breaks**

- **No draft:** look for a red module and check the form's Responses tab.
- **429, quota, 503, high demand, or Expensive Reads:** these are free-plan limits or Google being busy. Wait 10 minutes and click Run once one time. If that fails, write to the volunteer yourself.
- **The draft is short, blank, wrong, or says "full" when it shouldn't:** don't send it. Check that the Cap Sheet shift names match the form and that last week's rows are cleared.

### Setup checklist (to rebuild it)

- [ ] Google Form with the four fields above, email validation on
- [ ] Google Sheet with `Shift Sheet` and `Cap Sheet` tabs
- [ ] Import the Make blueprint *(see repo contents)* and reconnect Forms, Sheets, Gemini, and Gmail with your own accounts
- [ ] Forms trigger set to "From now on." Schedule set to **On demand**.
- [ ] Sheets #11 and #13 use Sheet Name **"Shift Sheet"**
- [ ] Gemini #8 and #9 use a **regular Flash** model, not Flash-Lite
- [ ] Tools aggregators have "Stop processing after an empty aggregation" **unchecked**
- [ ] Made-up data only

## Cost and limits

*These are estimates, not exact bills.*

- **About 8 Make credits per sign-up.** The free plan's 1,000 credits a month cover about 125 sign-ups.
- **Gemini free tier:** about 20 requests a day per model, so a Saturday rush could hit the limit.
- **Data note:** free Gemini inputs may be used to improve Google's products and can be read by reviewers. That is why real volunteer data is a Line 3 decision.

**Before and after (assumption, not a measured number):** If about 1 hour a week of Gloria's 4 hours goes to confirmations and "full" replies, and each draft takes about 1 minute to review (about 25 a week), the tool could give back **about 2 hours a month**. To measure it for real, track weekly scheduling minutes and Saturday no-shows for four weeks.

## Governance and bright lines

The team flags issues and the Advisor makes the final call. Bright-line issues are never handled alone:

| Line | Covers |
|---|---|
| 1 | Scope, fee, or timeline changes |
| 2 | Final governance recommendations and policy sign-off |
| 3 | Sensitive data moving somewhere new |
| 4 | Vendor selections or commitments |
| 5 | Legal, compliance, or ethics concerns |
| 6 | Public, board-facing, or funder-facing representations |

**Items routed to the Advisor (draft for Advisor review):**

- **Line 3:** Real volunteer names, emails, and shifts going into a free AI tool. Made-up data only until this is decided.
- **Lines 3 and 4:** Choosing the real AI and automation tools, and any paid plan. Make.com was used only to build the prototype. The final choice belongs to the Executive Director and the Advisor.
- **Line 5:** There is no written technology policy, and shared logins are kept in a binder.
- **Line 3:** A case manager was reported to be pasting intake notes into a personal, free AI chatbot.
- **Line 6:** A request for an "AI story" in the December annual report.

**Recommended first step this month:** approve OptimaNova drafting a written technology and AI use policy for Linden Street.

## Limitations and open questions

These are deliberately **not guessed**:

- **Real caps.** Saturday is about 29 to 30 (25 needed plus 4 to 5 extras). Tuesday and Thursday caps are unknown. The tests use made-up caps.
- **Shift hours.** Nobody confirmed exact hours. Only Saturday's 8:45 AM arrival is known.
- **Channel.** It's unknown whether volunteers prefer email or text. The build drafts emails only.
- **Volume.** It's unknown how many sign-ups come in each month, which affects cost and free-tier limits.
- **Test log.** Tests 1 to 4 and the F3 retest still need final results recorded.
- **Free-tier fragility.** 429 and 503 errors happen on free plans. A paid plan is a Line 4 decision.
- **Scope.** The build doesn't handle reminders, no-show tracking, or cancellations.

## Repo contents

> Edit this section to match the files you actually upload.

```
.
├── README.md                    ← you are here
├── blueprint/
│   └── volunteer-confirmations-shift-full.json   ← exported Make.com blueprint
├── prompts/
│   ├── confirmation-prompt.txt  ← Gemini #8
│   └── shift-full-prompt.txt    ← Gemini #9
├── docs/
│   ├── handoff-doc.pdf          ← one-page handoff for staff
│   ├── test-log.md              ← full test log
│   ├── governance-summary.pdf   ← draft for Advisor review
│   └── training-transcript.md   ← transcript of the demo video
└── screenshots/                 ← scenario, Cap Sheet, sample drafts
```

## Team and credits

**AI Tech Corps Capstone, Week 4.** Panel readout on November 24, 2026.

| Team member | Role |
|---|---|
| **Mackayla Rodriguez** | Build, Enable (handoff doc and training video), Build Demo |
| **Kay Richardson** | Diagnose (readiness assessment), Design (roadmap and solution spec) |
| **Anna Malaschenko** | Discover (interview evidence), Governance, Self-Check |

Program: **OptimaNova AI Institute**

*Linden Street Community Partners and all of its staff and volunteers are fictional. AI tools helped draft parts of this project. All outputs were reviewed by people.*
