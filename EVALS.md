# Artifact 3: Evals Used to Test Syncly

**What was tested:** the Syncly web prototype (the same pipeline as the Android app). Notices come in, then pass through the group whitelist, the regex extractor, the Gemini fallback, Synced/Pending routing and the Company Vault.
**When:** 26 Nov 2026. Relative words like "tomorrow" and "Friday" are checked against that date.
**How:** automated scripts drive the real app in a headless Chrome browser. Nothing below is estimated; every number comes from a run. The scripts and raw results are in the `evals/` folder so anyone can re-run them.

---

## 1. What "wrong" means for this product

Not every mistake costs the same. A student who shows up an hour late to an OA has lost the opportunity. A student who has to tap "Update" once has lost five seconds. So every failure is scored by what it costs the student:

| Severity | What happened | Cost to the student | Target |
|---|---|---|---|
| **Tier 1, critical** | A card shows the **wrong date or time** | Misses or is late for the round | **0** |
| **Tier 2, wrong state** | An incomplete notice is marked **Synced** instead of Pending | Trusts a card that is missing information | **0** |
| **False positive** | A card is created from chatter ("is the link working?") | Calendar noise; trust drops | 0 |
| **Missed event** | A real event gets no card | Back to scrolling WhatsApp | As low as possible |
| **Safe miss** | A complete notice goes to **Pending** instead of Synced | One tap on Update | Acceptable |

The design rule behind this: **when unsure, send it to Pending, never guess.** A safe miss is fine; a wrong time is not.

---

## 2. Eval 1: Parser accuracy on 40 hand-written notices (held out)

**Why held out:** the "Generate notification" button uses templates I wrote while building the parser, so passing those proves little. This set was written separately, to look like real placement-group messages. It deliberately includes formats the parser was not designed around: slang ("tmrw 3pm"), "12 noon", "10.30 am", "01.12.2026", pipe-separated notices, Hinglish, an invalid date (31/11), brand names inside tool names ("Google Calendar", "Microsoft Teams"), questions, thank-you messages, and results announcements.

Each notice has a human-written expected answer: whether a card should be created, Synced or Pending, the company, date, start/end time, venue and points. Gemini was **off** for this eval, so it measures the offline regex tier alone, which is the worst case.

### Results, before and after fixing what the eval found

| Metric | Round 1 | Round 2 (after fixes) |
|---|---|---|
| Notices fully correct | 34 / 40 | **38 / 40** |
| Routing correct (Synced / Pending / Ignored / Duplicate / Updated / Cancelled) | 36 / 40 | **39 / 40** |
| Company correct | 32 / 32 | 32 / 32 |
| Date correct | 30 / 32 | **31 / 32** |
| Start time correct | 28 / 30 | **29 / 30** |
| Venue correct | 9 / 9 | 9 / 9 |
| **Tier 1 errors (wrong date/time)** | **2** | **0** |
| Tier 2 errors (incomplete marked Synced) | 0 | 0 |
| False positives | 3 of 6 non-events | **0 of 6** |
| Regex extraction time per notice | 0.08 ms | 0.07 ms |

### What the eval caught, and what I changed

1. **Tier 1: two companies, two times** (B37: *"Tomorrow: Ola Electric OA 8pm, PhonePe OA 9pm"*). Both cards got 8 PM, so PhonePe was wrong by an hour. This was the worst failure. **Fix:** when a notice names several companies and gives the same number of times, each company is paired with its own time.
2. **Tier 1: "today… followed by… tomorrow"** (B16). The PPT that is *today* was filed under *tomorrow*, because the parser checked the word "tomorrow" before "today". **Fix:** use the first date mentioned in the message. For a reschedule ("moved from X to Y"), use the last one.
3. **False positives** (B18, B19, B30): *"Is the Jefferies test link working for anyone?"*, *"ICICI Bank interview results are out"* and *"Thanks everyone for attending the EXL PPT today"* each created a Pending card. **Fix:** questions, thank-yous and results announcements that give no time are ignored.
4. **A regression my own fix caused.** The first version of fix 2 broke "30 Nov 11:59 pm" by reading "Nov 11" as a second date. Re-running the eval caught it before release. It is fixed, and all 40 + 210 cases were re-run.

### What still fails (known limits, both safe)

- **B16, two rounds in one message** (*"PPT at 5:30 PM today followed by OA tomorrow at 8 PM"*). The PPT card is now correct, but no second card is created for the OA. This is a *missed event*, not a wrong one. This is the kind of message the Gemini fallback is meant to handle.
- **B24, Hinglish** (*"kal 5 baje Groww ka PPT hai CRC 102 mein"*). Regex can't read "kal" or "5 baje", so the notice goes to **Pending** with date and time missing. A safe miss, and another case for Gemini.

Full case list: see the Appendix.

---

## 3. Eval 2: Regression on every notification type (210 runs)

The "Generate notification" button can produce 20 kinds of notice. Each kind has a required outcome. All 20 were run **10 times each** with random companies, dates and formats, starting from an empty app each round. A separate check confirmed the first notice is always a Groww PPT that lands in Synced.

| Notice type | Must end up as | Passed |
|---|---|---|
| Clean PPT, OA "tomorrow", resume deadline, GD "16:00 hrs", "28.11.26 EOD", PPI "28/11", Google Meet interview, WhatsApp-invite briefing, PPT with points | Synced, with date and time filled | 90 / 90 |
| One message naming three companies | 3 Synced cards | 10 / 10 |
| Date only ("timings will be shared"), time only, registration link, "time TBA", messy "thursday evening, 6ish" (no Gemini key) | Pending | 50 / 50 |
| "Rescheduled to…" | Updates the existing card, no duplicate | 10 / 10 |
| "Stands cancelled" | Strikes the existing card | 10 / 10 |
| Same notice forwarded to another group | Ignored as duplicate | 10 / 10 |
| Non-placement group (Hostel Mess, CFI), chatter in a placement group | Ignored | 20 / 20 |
| First notification of a fresh app is Groww | Groww PPT, Synced | 10 / 10 |
| **Total** | | **210 / 210** |

---

## 4. Eval 3: End-to-end flows in the app

Scripted clicks through the real UI, checking what appears on screen:

| Flow | Checked | Result |
|---|---|---|
| Generate 6 notices | Cards appear, counters update (3 Synced / 3 Pending) | Pass |
| Tap **Update** on a Pending card, add the missing time, save | Card moves to Synced, counters become 4 / 2 | Pass |
| Vault → **+** → Add company | Opens as a page inside the app (no pop-up); duplicate names are blocked | Pass |
| Company page: add a note, a link and an image | All three saved and shown | Pass |
| Calendar: day strip and month grid | Events on the right days; Pending count shown with "Review" | Pass |
| Settings → turn on **Require Confirmation**, generate a notice | Notification shows Ignore / Add to Syncly; Add creates the card | Pass |
| Delete a card → **Undo** | Card removed, then restored | Pass |
| Paste a notice manually ("Kenvue PPT on 4th Dec at 5 PM, CRC 102, 5 points") | Same pipeline runs; correct Synced card with +5 points | Pass |
| Backup → Reset → Restore | All events and companies come back | Pass |
| Gemini key set but network unavailable | Falls back to regex; trace explains why; no crash | Pass |
| JavaScript errors across all runs | | 0 |

## 5. Eval 4: Screen sizes

Tested at 1920×1080, 1366×768 (laptop), 720 px (narrow browser pane) and 390×844 (phone). The phone keeps true device proportions on every screen, and nothing is cut off. On a real phone the app runs full-screen.

---

## 6. What these evals do *not* prove (honest limits)

- **I wrote the 40 test notices myself.** They are modelled on the kind of messages in our placement groups, but they are not a sample of real messages. The next step is to export ~200 real notices from MEUG Placements and score them the same way.
- **The Gemini tier was not tested with a live key.** Only its fallback path (no key, or no network) was tested. Its accuracy on messy and Hinglish notices is still untested.
- **40 is a small set.** "0 Tier 1 errors" means none were found in 40 cases, not that none exist.
- **The web prototype simulates notifications.** It does not test the Android NotificationListener itself (permissions, battery optimisation, WhatsApp message grouping).

## 7. How to re-run

`evals/golden.js` holds the 40 cases and expected answers. `evals/run.py` opens the prototype in headless Chrome and writes `results.json`. Run `python3 evals/run.py`. The round 1 and round 2 raw results are saved as `results_v1.json` and `results_v2.json`.

---

## Appendix: all 40 held-out cases

| ID | What it tests | Notice (shortened) | Expected | Round 1 | Round 2 |
|---|---|---|---|---|---|
| B01 | Missing time | All shortlisted candidates for Goldman Sachs Engineering Analyst, upload yo… | pending | ✅ | ✅ |
| B02 | Points + relative date | Google India Pre-Placement Talk today at 5:30 PM in CLT. Attendance carries… | synced | ✅ | ✅ |
| B03 | Chatter | Guys, can someone please share the syllabus for tomorrow's technical test? | ignored | ✅ | ✅ |
| B04 | Whitelist | Don't forget to submit mess rebate form today 5 PM. | ignored | ✅ | ✅ |
| B05 | Standard | Flipkart PPT at 6:30 pm on 2nd Dec, venue CRC 101 | synced | ✅ | ✅ |
| B06 | Standard range | Meesho OA: 3/12/2026, 7:00 PM – 8:30 PM. Link will be mailed. | synced | ✅ | ✅ |
| B07 | Dot time | Reminder: Swiggy interview slots tomorrow, 10.30 am onwards at ICSR 302 | synced | ✅ | ✅ |
| B08 | Weekday | Razorpay coding round on Saturday 9 PM on HackerEarth | synced | ✅ | ✅ |
| B09 | Noon | Bajaj Auto GD – 12 noon, 1 Dec, RMN 101 | synced | ✅ | ✅ |
| B10 | Deadline | PhonePe registration closes 30 Nov 11:59 pm. Apply on Superset | synced | ✅ | ✅ |
| B11 | Reschedule w/o original | Tata Steel PPT postponed to Monday, 4 PM | synced | ✅ | ✅ |
| B12 | 24h range | Texas Instruments test is from 10:00-11:00 hrs on 29/11 | synced | ✅ | ✅ |
| B13 | No date | Ather Energy shortlist is out. Interviews will be scheduled next week. | pending | ✅ | ✅ |
| B14 | Slang | Accenture PPT tmrw 3pm | synced | ✅ | ✅ |
| B15 | Dotted date | Honeywell online test on 01.12.2026 at 2 PM | synced | ✅ | ✅ |
| B16 | Two rounds, one message | Caterpillar PPT at CLT from 5:30 PM today followed by OA tomorrow at 8 PM o… | synced | ❌ Missed event; Tier 1 | ❌ Missed event |
| B17 | Standard | Kenvue Supply Chain Intern PPT - Dec 4th, 6 PM - Venue: CRC 102 | synced | ✅ | ✅ |
| B18 | Result, not event | ICICI Bank interview results are out. Congratulations to all selected! | ignored | ❌ False positive | ✅ |
| B19 | Question, not event | Is the Jefferies test link working for anyone? | ignored | ❌ False positive | ✅ |
| B20 | Short alias | HUL PPT on Friday at 11 AM in OAT | synced | ✅ | ✅ |
| B21 | Extension | Amex OA deadline extended to 2nd Dec 11:59 PM | synced | ✅ | ✅ |
| B22 | Window | DE Shaw - Online assessment link: https://hackerrank.com/xyz . Window: 9 AM… | synced | ✅ | ✅ |
| B23 | Pipe format | Infosys PPT: 28-11-2026 \| 16:30 \| Online (MS Teams) | synced | ✅ | ✅ |
| B24 | Hinglish | kal 5 baje Groww ka PPT hai CRC 102 mein | synced | ❌ Missed field; Safe miss | ❌ Missed field; Safe miss |
| B25 | Ambiguous time | Mahindra PPT at 7, 1st Dec | pending | ✅ | ✅ |
| B26 | Invalid date | Cummins interview on 31/11 | pending | ✅ | ✅ |
| B27 | Bare range | ZS Associates PPT 27th Nov 6-7 PM CRC 203 | synced | ✅ | ✅ |
| B28 | Standard | Qualcomm test: 2nd December, 9:00 AM | synced | ✅ | ✅ |
| B29 | Brand inside tool name | Microsoft Teams link for tomorrow's Deloitte interview at 3 PM: https://tea… | synced | ✅ | ✅ |
| B30 | Thank-you, not event | Thanks everyone for attending the EXL PPT today 🙏 | ignored | ❌ False positive | ✅ |
| B31 | Duplicate | Flipkart PPT at 6:30 pm on 2nd Dec, venue CRC 101 | duplicate | ✅ | ✅ |
| B32 | Forwarded duplicate | Fwd: Meesho OA: 3/12/2026, 7:00 PM – 8:30 PM. Link will be mailed. | duplicate | ✅ | ✅ |
| B33 | Cancel | Flipkart PPT stands cancelled. | cancelled | ✅ | ✅ |
| B34 | Reschedule | Meesho OA rescheduled to 4th Dec, 7 PM | updated | ✅ | ✅ |
| B35 | Brand inside tool name | Google Calendar invite has been sent for the Goldman Sachs interview on 2 D… | synced | ✅ | ✅ |
| B36 | TBA | Schneider Electric PPT will be on 5th Dec. Time: TBA | pending | ✅ | ✅ |
| B37 | Two companies, two times | Tomorrow: Ola Electric OA 8pm, PhonePe OA 9pm | synced | ❌ Tier 1 | ✅ |
| B38 | Standard | Jefferies PPT 28 Nov 5 PM | synced | ✅ | ✅ |
| B39 | Short alias | ITC PPT Friday 4 pm | synced | ✅ | ✅ |
| B40 | Chatter | ok | ignored | ✅ | ✅ |
