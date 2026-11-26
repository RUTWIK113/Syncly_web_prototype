# The Placement Blindspot: Why Chat Apps Fail at High-Stakes Deadlines

### A. My Take on the Problem Statement
During placement season at IIT Madras, communication runs almost exclusively through WhatsApp. In our hostel wing, students belong to multiple official and informal placement groups (e.g. *MEUG Placements*, *Placement Cell IITM*), receiving 300 to 500 messages daily. The vast majority is conversational noise: *"Has anyone received the link?"*, *"Is CLT open?"*, or casual chatter. Consequently, almost every student mutes these groups.

Last month in Cauvery hostel, a close wingmate missed a strict 24-hour assessment window for a tier-1 firm. The notification arrived at 10:15 AM, was buried under 70 follow-up queries, and by the time he checked after lab the next morning, the test link had expired at 10:00 AM. He lost the opportunity without writing a single line of code.

Students don't miss deadlines from negligence; they miss them because chat apps are built for conversation, not state tracking. While Google Calendar exists, manually translating ten unstructured notices every week—extracting venues, dates, cutoffs, and links—creates unsustainable friction during peak preparation.

---

### B. What is In Scope
1. **Targeted Group Filtering:** Intercepting incoming WhatsApp notifications exclusively from whitelisted placement groups, ignoring personal chats.
2. **Two-Tier Hybrid Parsing Engine:**
   - **Regex-First (Offline):** Deterministic pass extracting standard dates, 12h/24h times, campus venues (CLT, CRC), and assessment links in under 15ms.
   - **Gemini AI Fallback (BYO-Key):** Routes unstructured or ambiguous notices to Gemini 1.5 Flash using the student's personal API key.
3. **Deterministic State Routing (Synced vs. Pending):**
   - **Synced:** Notices with verified dates and times are organized chronologically and synced to Google Calendar.
   - **Pending:** Notices with missing times (e.g. *"Upload resume by Friday"*) route to Pending for a 1-tap manual timestamp resolution before calendar sync.
4. **Calendar View & Company Vault:** A monthly calendar grid with event markers, plus an automated Company Vault per firm to store Pre-Placement Talk (PPT) notes, JD PDFs, portal links, and attendance points.

---

### C. What is Out of Scope
1. **No In-App Chat Client:** We do not replace WhatsApp; communication remains native.
2. **No Placement Portal Functionality:** We do not handle resume screening, student verification, or job applications.
3. **No Preparation Content:** This is an operational workflow utility, not an ed-tech or interview-prep platform.
4. **No Centralized Data Scraping:** All parsing and vault storage remain 100% on-device in local storage.

---

### D. The Solution & Why
The solution is **Syncly**, an ambient placement intelligence and calendar automation utility.

**Why this architecture:**
We originally architected Syncly as an Android app because Android natively manages OS-level notifications via `NotificationListenerService`, enabling background parsing without copy-pasting. 

However, evaluating an Android APK requires enabling developer options, disabling Play Protect, and permitting untrusted APKs—creating massive friction for evaluators. To make the concept instantly verifiable, we packaged this exact pipeline into a **responsive web application prototype**. Evaluators can test the complete end-to-end flow—triggering simulated WhatsApp push notices, running the Regex/Gemini parser, routing events to Synced vs. Pending, and exploring the Company Vault—directly in any browser with zero setup.

---

### E. Evals & Testing: What We Verified vs. Guessed
**What We Verified:**
- **Hostel Audit:** Across Cauvery Hostel (IITM), 94% of placement notices flow through 4 core WhatsApp groups with an 82% noise ratio.
- **Severity-Tiered Evals (120-Notice Golden Dataset):**
  - *Tier 1 (Date/Time Precision):* 99.2% accuracy (zero catastrophic misses; wrong time has 0% tolerance).
  - *Tier 2 (State Routing):* 100% precision (every missing-time notice correctly isolated to Pending).
  - *Chatter Rejection:* 99.1% specificity (zero false-positive calendar clutter).
  - *Latency:* 14.2ms local regex tier (offline); 1.1s Gemini 1.5 Flash fallback.

**What We're Still Guessing:**
- The optimal threshold for auto-archiving past events without user anxiety.
- Whether students prefer automatic Google Calendar push or a manual "Review before Sync" confirmation.

*(Detailed test fixtures, error taxonomy, and benchmark matrices are documented in **Artifact 3: EVALS_FRAMEWORK.md**).*
