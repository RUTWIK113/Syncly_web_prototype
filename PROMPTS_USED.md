# Prompts Used

Syncly started as a small idea and grew one problem at a time. Each iteration below starts with what I noticed, followed by the prompt I gave to fix it.

---

## Part 1: The Android app

### Iteration 1: The core idea
*Starting point: placement notices get lost in busy WhatsApp groups.*

> Our placement WhatsApp groups get 100+ messages a day, mostly people asking questions, and the real notices get buried. I want a simple Android app (Kotlin, Jetpack Compose) that reads WhatsApp notifications in the background using NotificationListenerService and shows any placement-related message as a card with the company, date and time. Just a single list for now.

### Iteration 2: Too much noise
*Problem: it was picking up messages from every chat, including friends and family groups.*

> It's reading every chat, which isn't useful. Add a Settings screen where I can type in group names, like "ME UG Placements", and only read notifications from groups that match. Also add a few keywords (PPT, OA, interview, deadline, registration) so random chit-chat inside those groups doesn't become a card.

### Iteration 3: Dates and times in every format
*Problem: half the notices weren't being read properly because everyone writes dates differently.*

> The parser misses a lot. People write dates as 24/10, 24.10.26, 24 Oct, Oct 24th, tomorrow or this Friday, and times as 6 PM, 6:30 pm, 18:00 hrs, 10.30 am or 12 noon. Use regex to handle all of these, plus the event type (PPT, online test, interview, GD, resume, registration), the venue, and any links in the message.

### Iteration 4: Messages regex can't understand
*Problem: casual messages like "thursday evening around 6ish" break regex.*

> Some messages are too messy for regex. When regex can't find the company, date or time, send the message to Gemini instead. The user adds their own API key in Settings, and if there's no key or no internet, just stay with regex. Important: never let it make up a time that isn't in the message.

### Iteration 5: Wrong events on the calendar
*Problem: messages without a time were showing up with a wrong default time, which is worse than missing them.*

> If a message is missing the date or time, don't guess. Split the home screen into two: Synced, for events with both date and time, and Pending, for anything missing info. Each Pending card should have an Update button that opens the event with date and time pickers so I can fill it in, and then it moves to Synced.

### Iteration 6: Deadlines, duplicates and changes
*Problem: deadlines had no time, forwarded messages created duplicates, and reschedules created a second card.*

> A few things from using it this week. "Register by 24 Oct" has no time, so default deadlines to 11 PM, but don't do this for PPTs or tests. The same notice gets forwarded to several groups and creates duplicates, so check company, type and date before saving. And if a message says something was rescheduled or cancelled, update or strike through the existing card instead of adding a new one.

### Iteration 7: One place per company
*Problem: during PPTs I was writing notes in random apps and losing the JD links.*

> Add a Calendar tab with a day view and a month view. Also add a Vault tab: every company that shows up in a notice should automatically get its own page, where I can write PPT notes and attach photos, PDFs and links.

### Iteration 8: Old events piling up
*Problem: after a few weeks, Home was full of events that were already over.*

> Move past events into a "Past Events" card at the top of Home, like Telegram's archive, that expands to show them greyed out. In the Vault, show photos and PDFs as a 2-column grid of thumbnails instead of file names.

---

## Part 2: A web version anyone can try

### Iteration 9: Making it easy to try
*Problem: to try the APK, people have to turn on developer options and allow unknown apps. Most people won't do that.*

> Build a web version of Syncly that works exactly like the Android app: same flow, screens, cards and light amber theme. A browser can't read WhatsApp, so add a "Generate notification" button that slides in a realistic WhatsApp message and runs it through the same parser. Make the messages varied like real ones: clean notices, missing dates or times, "time TBA", "tmrw 3pm", one message with three companies, reschedules, cancellations, forwards, messages from non-placement groups, and plain chatter. Put the phone in the middle, a short "how it works" on the left, and on the right show what got picked up from each message and why.

### Iteration 10: Making it feel like a real app
*Problem: adding a company opened a big pop-up, and the Synced and Pending counts didn't change when a notification came in.*

> Two things feel off. Adding a company or event opens a big pop-up. Open it as a new page inside the phone instead, with a back button. And when a notification comes in, the Synced and Pending counts on Home should update right away, with the new card highlighted.

### Iteration 11: Fixing the layout
*Problem: the phone looked stretched in a narrow window and squeezed on a laptop.*

> Keep the phone at real phone proportions and scale it to fit whatever screen it's on. When someone opens it on an actual phone, just show the app full screen.

### Iteration 12: Testing it properly
*Problem: I needed to know whether it actually worked on messages it wasn't built around.*

> Write about 40 messages the way they really show up in our groups, including tricky ones: Hinglish, "12 noon", a wrong date like 31/11, questions, thank-you messages, results announcements. Note what should happen for each, then run them all. A wrong date or time is the worst failure, Pending is fine, and a card made from chatter is a fail. Fix whatever breaks and run it again.

*What it found: two companies in one message got the same time, a "today" event was filed under tomorrow, and some chatter became cards. All three were fixed.*

---

## What the app sends to Gemini

Used only when regex can't find the company, date or time.

```text
You extract campus placement events from WhatsApp notices for an Indian engineering college. Today is {today} ({weekday}).
Return ONLY JSON: {"action":"create"|"reschedule"|"cancel"|"ignore","events":[{"company":string,"type":one of {types},"date":"YYYY-MM-DD" or null,"start":"HH:MM" (24h) or null,"end":"HH:MM" or null,"deadline":boolean,"venue":string or null,"points":number or null}]}
Resolve relative dates and fuzzy times (e.g. "thursday evening around 6ish" -> that Thursday, 18:00). Use null when the notice truly does not give a value. "ignore" if it is chit-chat.
Notice:
"""{notice}"""
```
