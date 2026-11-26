# Syncly — Placement Intelligence & Automated Calendar Engine
> **Groww Associate Product Manager (APM) Case Study Submission**  
> Candidate: **Rutwik Nakkalla** (ME23B113, IIT Madras)  
> Live Prototype: **[https://rutwik113.github.io/groww_assmt/](https://rutwik113.github.io/groww_assmt/)**  
> Native App Source: [https://github.com/RUTWIK113/syncly](https://github.com/RUTWIK113/syncly)

---

## 🚀 Live Interactive Prototype
You can test the full prototype directly in your browser with zero installation:
👉 **[Open Live Prototype (GitHub Pages)](https://rutwik113.github.io/groww_assmt/)**  
*(Or clone this repository and double-click `index.html` in Chrome, Edge, or Safari).* 

---

## 📁 Required Submission Artifacts

| Artifact | File | Description |
|---|---|---|
| **1. 1-Page Submission** | [`1_PAGE_SUBMISSION.md`](./1_PAGE_SUBMISSION.md) | Human-written narrative (~644 words) covering Problem Take, In/Out Scope, Solution & Why, and What We Verified vs. Guessed. |
| **2. Prompts Used** | [`PROMPTS_USED.md`](./PROMPTS_USED.md) | Structured LLM extraction prompts, few-shot fixtures, JSON schema, and regex boundary rules. |
| **3. Evals Framework** | [`EVALS_FRAMEWORK.md`](./EVALS_FRAMEWORK.md) | 4-tier error severity taxonomy, 5 evaluation dimensions, 5 benchmark test fixtures, and golden dataset results. |
| **4. Working Prototype** | [`index.html`](./index.html) | Zero-install interactive web application replicating the full Android pipeline. |

---

## 🎯 The Core Problem: The Placement Blindspot
During campus placement season at IIT Madras, communication runs almost exclusively through high-velocity WhatsApp groups (e.g. *MEUG Placements*, *Placement Cell IITM*), generating **300 to 500 messages daily**.

Because **82% of group messages are conversational noise** (*"Has anyone received the link?"*, *"Is CLT open?"*), students mute these groups. Last month in Cauvery Hostel, a wingmate missed a strict 24-hour assessment deadline from a tier-1 firm simply because the announcement was buried under 70 queries while he was in a lab.

**The Insight:** Students don't miss deadlines from negligence; they miss them because chat apps are built for conversation, not state tracking.

---

## ⚡ The Solution: Syncly
Syncly is an ambient placement intelligence utility that intercepts incoming WhatsApp notices, extracts event entities via a **hybrid regex + LLM engine**, and routes them into:
1. **Synced Section:** Notices with verified dates and times are pushed to Google Calendar.
2. **Pending Section:** Notices with missing times (e.g. *"Upload resume by Friday"*) route to Pending for a 1-tap manual timestamp resolution before calendar sync.
3. **Placement Vault:** Auto-generated company dossiers to store Pre-Placement Talk (PPT) notes, attached JDs, and placement incentive points.

---

## 🔬 Why a Web Prototype (Instead of an Android APK)?
Native Syncly is engineered in Kotlin + Jetpack Compose using Android's OS-level `NotificationListenerService`. However, installing an untrusted third-party APK requires evaluators to enable developer options, disable Google Play Protect, and permit unverified installations from Chrome—creating massive review friction.

To make the solution **immediately verifiable in seconds**, we packaged the entire pipeline into this self-contained, responsive web application prototype. All state machines, hybrid parsing logic, and responsive UI faithfully mirror the native app.

---

## 🧪 Quick Test Guide for Evaluators
1. Open [`index.html`](https://rutwik113.github.io/groww_assmt/) in any browser.
2. Click **"Generate Notification"** to cycle through 8 realistic placement notices:
   - **Groww Assignment:** Complete datetime -> Synced immediately.
   - **Google India:** Multi-round split (PPT + OA) -> 2 distinct event cards created.
   - **Goldman Sachs:** Resume upload missing start time -> Routed to Pending.
   - **Chatter Noise:** Group query -> Noise filter rejects (0 calendar clutter).
   - **Morgan Stanley:** Assessment postponed -> Synced with postponement tag.
   - **Hostel Wing:** Non-whitelisted group -> Discarded by whitelist filter.
3. Test full in-app navigation:
   - Tap **"+"** on Home to add an event manually.
   - Tap **"Resolve Time ↗"** on any Pending card to sync it to Calendar.
   - Tap **Vault** -> **"+"** to create a Company Dossier.
   - Tap any company card to open the in-app Dossier (Notes, Docs, Rounds tracker).

---
*Submitted for Groww APM Recruitment 2026 · IIT Madras*