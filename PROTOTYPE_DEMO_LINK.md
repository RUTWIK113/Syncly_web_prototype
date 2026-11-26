# Artifact 4: Link to the App

- **Live Web Prototype (GitHub Pages):** [https://rutwik113.github.io/groww_assmt/](https://rutwik113.github.io/groww_assmt/)
- **GitHub Repository (Assignment Submission):** [https://github.com/RUTWIK113/groww_assmt](https://github.com/RUTWIK113/groww_assmt)
- **Native Android App Source:** [https://github.com/RUTWIK113/syncly](https://github.com/RUTWIK113/syncly)

Opens immediately in any browser on desktop or phone. Zero installation, zero sign-up.

---

## Why a Web Prototype?

Syncly was originally architected and built as a native Android app in Kotlin and Jetpack Compose. It uses Android's OS-level `NotificationListenerService` to intercept real-time WhatsApp placement notifications without student copy-pasting.

However, installing a third-party APK requires evaluators to enable developer options, disable Google Play Protect, and allow untrusted sources in Chrome. To eliminate this review friction and make the concept instantly verifiable, we packaged the exact same state machine, hybrid parsing engine, and responsive UI into this zero-install web prototype.

---

## 2-Minute Evaluator Walkthrough

1. **Generate notification** (click the button at the top or in the panel): A simulated WhatsApp notification slides in with audio chime, Syncly parses it, and a card appears live.
2. Click **Generate Notification** to test the edge cases:
   - Complete notice (e.g. Groww OA, Google India) $\rightarrow$ routes to **Synced**.
   - Missing time notice (e.g. Goldman Sachs resume deadline) $\rightarrow$ routes to **Pending**.
   - Conversational chatter $\rightarrow$ filtered out as noise (zero calendar clutter).
   - Postponed notice (e.g. Morgan Stanley) $\rightarrow$ marked with postponement tag.
   - Non-whitelisted group (e.g. Hostel Wing) $\rightarrow$ filtered at ingestion.
3. Open **Pending**, click **Resolve Time ↗**, enter the confirmed start time, and watch the card move to **Synced**.
4. **Calendar tab:** View monthly grid and events agenda day-by-day with 1-click Google Calendar export.
5. **Vault tab:** View company dossiers, click any company to open in-app tabs for Notes, Docs & Links, and Interview Rounds.
6. **+ button:** Add a placement event manually or add a new company dossier to the Vault.

*All state is stored client-side in `localStorage`, so every evaluator starts fresh with full interactive capabilities.*
