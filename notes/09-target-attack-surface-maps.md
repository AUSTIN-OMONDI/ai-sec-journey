# 09 - Target Attack Surface Maps

Week 14, Tuesday · 2026-08-09 (catch-up day for Jul 28 slot)

Goal: 3 candidate targets, each with a first-pass attack surface map, ready for Wednesday's deep feature inventory on whichever one gets picked first.

---

## Selection rationale

Picked from the programs reviewed Monday ([notes/08-bounty-programs.md](08-bounty-programs.md)), filtered by the finding that mattered most: OpenAI, Anthropic, and Google all pay for a **security boundary crossing** (data exfil, unauthorized action, permission bypass) not a bare jailbreak. So the 3 targets are chosen for having real *tool-use / agentic / data-connector* surface — the shape of feature that produces S1/S2-style findings — rather than being picked for jailbreak-friendliness.

| # | Target | Program / scope basis | Asset tier | Top relevant reward |
|---|---|---|---|---|
| 1 | **Claude.ai** (web) — Projects, Connectors/MCP, Artifacts | [Anthropic × HackerOne](https://hackerone.com/anthropic) | Core | $7,500–$10,000 (Critical) |
| 2 | **Gemini in Google Workspace** (Gmail/Drive side panel) | [Google AI VRP](https://bughunters.google.com/about/rules/google-friends/ai-vulnerability-reward-program-rules) | Flagship | $20,000 (S1 Rogue Actions) / $15,000 (S2 Data Exfil) |
| 3 | **ChatGPT** (web) — Agent Mode, Connectors, Custom GPTs/Actions | [OpenAI × Bugcrowd](https://bugcrowd.com/openai) (Security + Safety) | N/A (single program) | $6,500/finding, $20,000 researcher cap |

All three are explicitly in scope, all three have named, unambiguous asset entries (not "any AI feature"), and all three now ship an external data-connector or agentic-action layer as of mid-2026 — that's the feature class most likely to produce a real S1/S2/rogue-action/data-exfil finding via indirect prompt injection.

---

## Target 1 — Claude.ai

**Program basis:** Anthropic HackerOne, "Official Clients" + Claude.ai domain, Core asset. Model-content jailbreaks are explicitly out of scope here — findings need to demonstrate unauthorized action, data exposure, or permission bypass.

**Surface inventory (public/feature-level, pre-login):**

| Area | Detail |
|---|---|
| Primary URL | `claude.ai` (web app) |
| API surface | `api.anthropic.com` (in scope per Anthropic HackerOne policy — verify exact asset entry when logged into H1 dashboard) |
| File upload | Documents, images, PDFs uploadable directly into chat and Projects |
| RAG / external data sources | **Connectors (MCP)** — official + community MCP servers (200+ as of May 2026) can pull from Google Drive, Gmail, Calendar, Slack, GitHub, Jira, databases, etc. Multiple connectors can be active in one conversation |
| Tools available to the LLM | Web search (toggle, all tiers), code execution inside Artifacts, MCP tool calls (read + write depending on connector), Artifacts can now call Claude's API directly and persist storage across sessions |
| Memory/persistence | Projects persist context across sessions; Artifacts can now save/retrieve data between visits |
| Agentic surface (adjacent, separate H1 asset) | Claude Code (CLI + IDE extensions) — explicitly in scope for permission-bypass / unauthorized command execution; desktop app supports MCP too |

**Where the interesting attack surface is:** any point where a connector pulls untrusted external content (an email, a shared doc, a Slack message, a GitHub issue) into context — that's the indirect-injection vector. A connector with *write* capability (send email, create calendar event, push to Slack/GitHub) is what turns a successful injection into an S1-shaped "rogue action" finding rather than just leaked text.

**Open questions for Wednesday's deep dive:** exact list of connectors available on a free/Pro account (some may be Team/Enterprise-gated), whether Artifacts' "call Claude's API directly" feature can be reached from injected content, whether persistent Artifact storage crosses conversation/user boundaries.

---

## Target 2 — Gemini in Google Workspace

**Program basis:** Google AI VRP, Flagship tier (Workspace core apps: Gmail, Drive, Calendar, Docs, Sheets, Slides, Forms + Gemini Apps). Highest base rewards of all three ($20k S1 / $15k S2). Jailbreaks/hallucinations explicitly excluded — Google's own S1 example is *exactly* "indirect prompt injection → unconfirmed action," which is a strong signal for what to test.

**Surface inventory:**

| Area | Detail |
|---|---|
| Primary URLs | `gemini.google.com`; Gemini side panel embedded in `mail.google.com`, `drive.google.com`, `docs.google.com`, `calendar.google.com`, `keep.google.com`, Tasks |
| File upload | Direct upload + "Ask Gemini" over Drive files/folders |
| RAG / external data sources | Gmail threads pulled directly into Drive-side Gemini context (new as of June 2026 rollout); cross-referencing between emails and Drive files in one query |
| Tools available to the LLM | "Workspace apps" (formerly "Extensions") — when enabled, Gemini can interact with and take actions across connected Google apps, not just read them |
| Agentic surface | Summarizing threads, pulling related docs, identifying action items across apps in a single query — the multi-app chaining is the novel part vs. 2025 |
| Rollout status | Open beta as of Jun 3, 2026; tiers: Business Standard/Plus, Enterprise, select Education, Google AI Pro/Ultra — confirm which tier the account testing from actually has |

**Where the interesting attack surface is:** an attacker-controlled email or shared Drive doc/comment is untrusted content that lands directly in Gemini's context when a victim asks it to "summarize my inbox" or "pull related docs." Google's own S1 example (calendar invite → unlocks a door) confirms this exact shape is what they reward. Action-taking "Workspace apps" (not just read) is the part that turns a leak into a rogue action.

**Open questions for Wednesday:** which specific actions Gemini can execute vs. only read in the enabled Workspace apps; whether Gmail→Drive context bridging can be triggered by content the victim didn't directly ask about (i.e., surprise inclusion); scope boundary between this (AI VRP) and Google Cloud VRP if any tested surface touches Vertex AI.

---

## Target 3 — ChatGPT (web)

**Program basis:** OpenAI × Bugcrowd, both Security and Safety bug bounty apply. Safety program explicitly covers "agentic risks (including MCP)" — best fit of the three OpenAI programs for this kind of testing. Sandboxed code execution (Python interpreter, Agent Mode container, GPT-5 container tool) is explicitly out of scope with documented fingerprints to rule out before claiming RCE — worth re-reading before touching Agent Mode.

**Surface inventory:**

| Area | Detail |
|---|---|
| Primary URL | `chatgpt.com` (`chat.openai.com` redirects) |
| File upload | Direct upload in chat; Code Interpreter / Advanced Data Analysis sandbox |
| RAG / external data sources | **Connectors**: Gmail, Google Calendar, Google Drive, SharePoint, Slack, Salesforce, Notion, Atlassian — read-only by default, admin-gated |
| Tools available to the LLM | **Agent Mode**: virtual computer, visual browser, terminal, file access, plans and executes multi-step tasks, asks permission for "consequential" actions, connectors are read-only but the agent's own browsing/terminal actions are not |
| Agentic surface | Custom GPTs + Actions (call external APIs via OpenAPI schema) — being deprecated for business accounts but still live; Workspace Agents (new tier: persistent, schedulable, shareable, connect to apps/tools, run via Slack/API) |
| Developer/beta surface | "Developer mode and full MCP connectors in ChatGPT" (beta) — direct MCP support, worth checking current rollout status before testing |

**Where the interesting attack surface is:** Agent Mode's browser + terminal combo is the highest-value surface — it's an LLM taking autonomous, permission-gated actions on the open web, which is precisely the "agentic risk" the Safety program calls out. Custom GPT Actions (arbitrary OpenAPI-defined external calls) is a second distinct vector: a malicious or compromised GPT could chain an Action to exfiltrate data pulled from a Connector. Read-only Connectors reduce direct rogue-action risk but don't eliminate data-exfiltration risk if Agent Mode can then post that data somewhere via the browser.

**Open questions for Wednesday:** whether "Developer mode / full MCP connectors" beta is enabled on the test account; exact definition of "consequential action" that triggers a permission prompt in Agent Mode (this is the boundary a bypass would need to cross); whether a Custom GPT Action can be pointed at an attacker-controlled endpoint without the platform flagging it before use.

---

## Burp Suite project setup (per target)

One Burp project per target, per the plan. Since I can't drive Burp directly, here's the setup to run manually before Wednesday's session:

**General config (all 3 projects):**
- New project file per target: `burp-claude.burp`, `burp-gemini-workspace.burp`, `burp-chatgpt.burp` — keep separate, don't mix scope across programs (accidental cross-program testing is a fast way to void safe harbor).
- Proxy listener on `127.0.0.1:8080` (default), browser configured to route through it (or use Burp's embedded Chromium).
- Install/trust Burp's CA cert in the testing browser profile — use a **dedicated browser profile per target**, logged in with a test/throwaway account, not your daily-driver Google/OpenAI/Anthropic account.

**Target-specific scope (Target > Scope > Add):**

| Target | Include in scope | Explicitly exclude |
|---|---|---|
| Claude.ai | `claude.ai`, `api.anthropic.com`, relevant MCP connector callback domains you enable | Anything under a third-party connector's own domain you don't own (e.g. don't scan Slack's or GitHub's infra — only the data flow through Claude) |
| Gemini Workspace | `gemini.google.com`, `mail.google.com`, `drive.google.com`, `docs.google.com` (only while testing the Gemini side panel flows) | Rest of Workspace unrelated to Gemini; Vertex AI / Cloud endpoints (different program) |
| ChatGPT | `chatgpt.com`, `chat.openai.com` | Sandboxed interpreter/Agent Mode container internals — out of scope per policy, don't spend Burp traffic there |

**Practical note:** for all three, most of the interesting traffic (connector data pulls, tool calls) happens server-side between the AI vendor and the third-party connector (Gmail, Drive, Slack) — Burp on your browser won't see that hop directly. What Burp *will* show: the request/response carrying your injected payload into the model, and the client-rendered result of any action the model takes back through the web UI. Plan Thursday's baseline-behavior pass around what's actually observable at the proxy layer vs. what you'll have to infer from UI behavior.

---

## Carried into Wednesday

Per the week plan, Wednesday goes deep on **one** target only (feature inventory, LLM-touching feature list, tools/file-uploads/RAG sources). Given the reward ceiling (Google's S1 = $20k, highest of the three) and the unusually explicit "here's exactly what we pay for" S1 example Google gives (indirect injection → unconfirmed action), **recommend starting with Target 2 (Gemini in Workspace)** unless you have a strong reason to prefer the Anthropic or OpenAI ecosystem instead (e.g. more familiarity with Claude/MCP from other project work).
