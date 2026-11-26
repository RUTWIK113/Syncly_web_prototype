# Prompts Used

**PRD:** [View the Syncly PRD (Google Drive)](https://drive.google.com/file/d/1SfTJpCNIa3kDJREcVyjqM8MKNqZlSzdP/view?usp=sharing)

## How I built it

Before writing any code, I realised how much a clear PRD matters: it keeps every prompt pointed at the same problem. So I followed a structured process. I recorded all my rough thoughts as voice notes, had ChatGPT turn them into one structured PRD, and gave that PRD to Claude to build the first version. After that I kept testing it on real messages from our placement groups, and in each iteration I added, fixed, refactored or removed one thing.

---

## Step 1: From voice notes to a PRD (ChatGPT)

> I'm going to ramble about an app idea, so bear with me. Our placement WhatsApp groups get 100+ messages a day, mostly doubts and follow-ups, and the actual notices about PPTs, tests and deadlines get buried. My wingmate missed a 24-hour test because of this. I want an app that reads these messages and turns them into calendar events by itself… *(about 10 minutes of voice notes)*. Turn all of this into a proper PRD: the problem, features, how parsing should work, edge cases, and what's out of scope.

## Step 2: The skeletal structure from PRD to begin with (Claude)

> Here's my PRD for Syncly. Build the first version as an Android app in Kotlin with Jetpack Compose. Start with the core loop only: read WhatsApp notifications, pull out the company, date and time, and show them as cards. Use regex first, and only send a message to Gemini (with the user's own API key) when regex misses something. Never make up a time.

---

## Iterations: test, find the problem, fix

### 1. Added: group whitelist
*Problem: it was reading every chat, even family groups.*
> Only read groups whose names I add in Settings, like "ME UG Placements". Ignore everything else.

### 2. Fixed: date and time formats
*Problem: half the notices were missed because everyone writes dates differently.*
> Handle 24/10, 24.10.26, Oct 24th, tomorrow, this Friday, 6 PM, 18:00 hrs, 10.30 am, 12 noon and EOD.

### 3. Refactored: Synced and Pending
*Problem: events with no time were showing up at a wrong default time, which is worse than missing them.*
> Split Home into Synced (date and time both known) and Pending (something missing). Replace the Dismiss and Add buttons on Pending cards with a single Update button that opens date and time pickers.

### 4. Fixed: deadlines, duplicates and reschedules
*Problem: forwards created duplicates and "rescheduled to" created a second card.*
> Default deadlines with no time to 11 PM (not PPTs or tests). Check company, type and date before saving. Update or strike through the old card when a message says rescheduled or cancelled.

### 5. Added: Calendar and Company Vault
*Problem: my PPT notes and JD links were scattered across apps.*
> Add a Calendar tab, and a Vault where every company automatically gets a page for notes, photos, PDFs and links.

### 6. Refactored: Past Events archive
*Problem: Home filled up with old events.*
> Move past events into a collapsible "Past Events" card at the top, like Telegram's archive.

### 7. Added: web version
*Problem: to try the APK, people need developer options and unknown apps turned on.*
> Build the same app for the browser. Since it can't read WhatsApp, add a "Generate notification" button that sends realistic messages, including the messy ones (missing dates, "tmrw 3pm", forwards, chatter), through the same parser.

### 8. Removed: pop-ups
*Problem: "Add company" opened a big pop-up, and the counts didn't update.*
> Open everything as a page inside the phone, and update the Synced and Pending counts live.

### 9. Fixed: phone layout
*Problem: the phone looked stretched or squeezed depending on the screen.*
> Keep real phone proportions on every screen; full screen on actual phones.

### 10. Fixed: bugs found by testing
*Problem: I needed proof it worked on messages it wasn't built around.*
> Write 40 real-style messages (Hinglish, "12 noon", 31/11, questions, thank-yous), note the right answer for each, and score the app. A wrong time is the worst failure.

*Found and fixed: two companies in one message got the same time, a "today" event was saved as tomorrow, and chatter became cards.*
