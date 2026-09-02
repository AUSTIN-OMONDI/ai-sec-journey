# 10 - Target 1 Feature Map: Gemini in Google Workspace

Week 14, Wednesday · 2026-08-09 (catch-up day for Jul 29 slot)

Target selected from Tuesday's shortlist ([notes/09-target-attack-surface-maps.md](09-target-attack-surface-maps.md)): Gemini in Google Workspace, scoped under [Google AI VRP](https://bughunters.google.com/about/rules/google-friends/ai-vulnerability-reward-program-rules), Flagship tier.

Goal today: full feature inventory, every LLM-touching entry point, and — most important per the program's own S1/S2 categories — which connected apps are **read-only** vs **write-capable**, since write-capable is what turns a leak into a rogue action.

---

## Entry points (where a prompt reaches the model)

| Surface | URL / location | Notes |
|---|---|---|
| Standalone app | `gemini.google.com` | Primary chat interface |
| Docs/Sheets/Slides/Drive side panel | embedded in `docs.google.com`, `sheets.google.com`, `slides.google.com`, `drive.google.com` | "continuous, secure experience in the side panel" — same session context can follow across these apps |
| Gmail side panel | embedded in `mail.google.com` | Draft help, summarization, now also pulls in cross-referenced Drive content |
| Meet | embedded in Google Meet | (not deep-dived today — flag for later if pursuing this target further) |
| Gemini in Chrome (browser extension) | Chrome browser, any tab | Reads **page content, URL, and cookies containing site auth info** from the current tab — distinct and broader data-access model than the Workspace side panel |
| Mobile | Android/iOS Gemini app | Device-level access (see Data Access table below) — different attack surface than web, not in scope for this week's plan |

**Read for scoping:** Gemini in Chrome is a materially different (and broader) attack surface than the Workspace side panel — it can see cookies/auth info and arbitrary page content, not just Workspace documents. Worth treating as effectively a 4th sub-target if time allows later, but staying focused on Workspace side panel + standalone app for this week per the plan.

---

## Connected apps — read vs write capability

This is the key table for today. Google's admin docs describe capability **per app**, but the on/off *control* is only at the category level (can't disable Gmail while keeping Drive on) — so once "Workspace apps" is enabled for an account, all of these are live simultaneously in one conversation.

| App | Capability | Read or Write? | Attack-surface relevance |
|---|---|---|---|
| Gmail | Find/summarize messages. **Cannot** create drafts, delete emails, access attachments/images | Read-only | Classic indirect-injection vector: attacker-controlled email body is untrusted content Gemini ingests when asked to summarize inbox |
| Google Calendar | **Create, find, AND edit events** | **Read + Write** | Highest-value target — matches Google's own published S1 example verbatim (indirect injection → unconfirmed calendar/door-unlock action) |
| Google Drive | Find info from Gmail / summarize Drive docs. **Cannot** manage folders, access images/video | Read-only (for documents) | Shared doc with injected instructions is a plausible delivery vector; new Gmail↔Drive cross-referencing (Jun 2026) widens what gets pulled into one context |
| Google Keep | **Create and show** notes/lists | **Read + Write** | Lower blast radius than Calendar, but still a write action — good secondary target for a state-changing PoC if Calendar proves hard |
| Google Tasks | **Add and show** tasks | **Read + Write** | Same category as Keep — write action, low blast radius |
| Google Classroom | Quick answers, stay organized. **Cannot** enter grades, delete assignments, create rubrics | Read-only (guardrailed) | Education-tier only; not relevant unless testing an edu account |
| GitHub | Attach a repo "to better understand the codebase and debug issues" | Read-only (repo content) | Web app only; a malicious README/code comment is a plausible injection vector if pursued |
| Maps / YouTube / Hotels / Flights | Consumer-account only ("Other Google apps" toggle) | Mixed | Not gated by Workspace admin — personal accounts have a different toggle; out of scope for a Workspace-account test plan |

**Priority pick for Thursday's baseline + injection probes: Calendar.** It's write-capable, it's the exact scenario in Google's own S1 reward example, and both Gmail (read, injection delivery) and Calendar (write, action target) can be chained in one flow: poisoned email → Gemini asked to "check my email and schedule anything I need to" → does it create/edit a calendar event without explicit confirmation of the injected instruction?

---

## Data access model (from Gemini Apps Privacy Hub)

What can reach the model, beyond connected-app content:

- **Device data** (Gemini mobile app only): call/message logs, contacts, installed apps, language, screen content
- **Connected Apps**: email, files, events, photos/videos from integrated services (per-app limits per table above)
- **Browser context** (Gemini in Chrome only): cookies incl. site auth info, screen captures, page content/URL of current tab
- **Location**: device location, IP, Home/Work address from Google Account
- **User activity**: Search history, YouTube history, Chrome page URLs
- **Grounding**: some responses grounded on live Search results (a second, less controllable RAG source layered on top of connected-app data)

**Session/data boundary note (important for safe testing):** "the model is structured to only present data from a session to that individual user" — so cross-*user* boundary testing needs a real two-account setup (attacker account + victim account you control), not just single-account prompt games. Plan Thursday/Friday testing around two test accounts you own, consistent with the Google VRP rule to only ever target your own accounts.

**Useful for low-footprint testing:** Temporary Chats mode and "Keep Activity off" (72h retention instead of 18mo) — good default state while running baseline/injection probes so test traffic doesn't pile up in long-term activity history.

---

## File upload mechanics

- Direct file upload in `gemini.google.com` chat (documents, images — exact type/size limits not yet confirmed, check in-product on Thursday)
- Drive-native: "Ask Gemini" over an existing Drive file/folder without uploading (the file is already in Drive) — this is likely the more interesting vector since the file didn't have to pass through an explicit upload/scan step, and ownership/sharing permissions determine what's reachable
- Gmail attachments: explicitly **not** accessible per the docs ("cannot access attachments/images") — worth verifying empirically since docs can lag product behavior

---

## LLM-touching feature checklist (per Wednesday's plan template)

- [x] Tools available to the LLM → connected-app actions (table above); Search grounding
- [x] File uploads → direct chat upload + Drive-native reference (no upload step)
- [x] RAG sources → Gmail, Drive, Calendar, Keep, Tasks, Classroom, GitHub, live Search grounding, (Chrome-only: arbitrary page content)
- [x] Write/action-capable paths → Calendar (create/edit), Keep (create), Tasks (add) — these are the S1 "rogue action" candidates
- [x] Cross-context bridging → Gmail↔Drive cross-referencing in one query (Jun 2026 feature) — increases blast radius of a single injected document
- [ ] Exact upload size/type limits → defer to hands-on check Thursday
- [ ] Whether Gmail attachment-blindness holds up empirically → defer to hands-on check Thursday

---

## Carried into Thursday

Baseline behavior + first injection probes should center on the **Gmail → Calendar chain**: establish normal behavior first (how does Gemini summarize a clean inbox, does it ever touch Calendar unprompted), then introduce a single lightweight, clearly-labeled test payload in a self-sent test email and observe whether any instruction embedded in the email body influences a Calendar action — without ever targeting an account that isn't yours, per both the Google VRP rules and this week's scope discipline.
