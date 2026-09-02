# 08 - Bounty Program Scope Summary

Week 14, Monday · 2026-08-09 (catch-up day for Jul 27 slot)

Goal: know exactly what's in scope, what pays, and what gets a report auto-closed, across the five programs before picking Tuesday's candidate targets.

---

## Quick comparison

| Program | Model | Top reward | Model-content bugs (jailbreaks, hallucinations)? | Notes |
|---|---|---|---|---|
| [0DIN (Mozilla)](https://0din.ai) | Classic bug bounty, submit via portal | $15,000 (weights/layers disclosure) | **In scope** — this is the point of the program | Targets specific named models (Claude, GPT, Gemini, LLaMA, etc.), not arbitrary apps |
| [huntr (Protect AI)](https://huntr.com) | **Gamified CTF-style challenge arena**, leaderboard payout | $15,000 pot (current challenge) | In scope, but framed as "objectives" against a specific demo agent, not open submission | Has pivoted away from the old open OSS/model-file bounty model — see note below |
| [OpenAI (Bugcrowd)](https://bugcrowd.com/openai) | Two programs: Security (classic) + Safety (agentic/abuse) | $6,500/finding, $20,000/researcher cap (Safety) | **Out of scope** for both — explicit, extensive exclusion list | Route model-content issues to the Model Behavior Feedback form instead |
| [Anthropic (HackerOne)](https://hackerone.com/anthropic) | Classic bug bounty (infra/product) + separate model safety bounty | $8,000 avg (Critical, Core); model safety bounty up to $15,000 | **Out of scope** for the main program; jailbreaks handled by a *separate* model safety bounty | Claude Code in scope for permission-bypass / unauthorized command execution |
| [Google AI VRP](https://bughunters.google.com/about/rules/google-friends/ai-vulnerability-reward-program-rules) | Classic bug bounty with AI-specific category table | $20,000 base (S1/S2, Flagship tier), up to ~$30,000 w/ multiplier | **Out of scope** — jailbreaks/hallucinations explicitly listed as non-qualifying | Wants *security impact* (data exfil, unauthorized actions), not policy-violating output |

**Biggest takeaway for scoping targets:** four of the five programs (OpenAI, Anthropic, Google, and implicitly the serious tier of huntr) do **not** pay for "I got the model to say something bad." They pay for a security or privacy boundary crossing — prompt injection that exfiltrates data, unauthorized tool/action execution, permission bypass, sensitive info disclosure. Only 0DIN and huntr's challenge format reward pure guardrail bypass as the primary category. This should drive which attack classes get priority time this week (indirect prompt injection → data exfil / rogue actions, not raw jailbreaking).

---

## 0DIN (Mozilla) — [0din.ai](https://0din.ai)

**What it is:** GenAI bug bounty covering vulnerabilities *in specific named models*, not in a company's product surface.

**In-scope vulnerability classes (with reward bands):**
| Category | Reward |
|---|---|
| Prompt Extraction | ~$100 |
| Guardrail Jailbreak (must reproduce across ≥2 testing categories) | $500 – $1,000 |
| Interpreter Jailbreak (escape sandboxed code execution/tool use) | $2,500 |
| Content Manipulation (inject harmful/misleading content into model I/O) | $5,000 |
| Weights & Layers Disclosure (extract learned parameters/architecture) | $15,000 |

**Coverage:** 46 models across 15 orgs — Anthropic (11 Claude variants), OpenAI (12 GPT models + DALL-E3), Google (5 Gemini variants), Meta, Amazon, Apple, Cohere, IBM, Microsoft, NVIDIA, Perplexity, Salesforce, X, BigScience, others. Per-model exclusions exist (e.g. prompt extraction is "N/A" on nearly all models; illicit-substance content excluded for Claude/Command R/Grok).

**Process:** submit a high-level abstract + affected model(s) first; 0DIN responds within 3 business days on in-scope decision. Full validation target: 2 weeks. Payment within 30 days of agreement.

**Disclosure:** if vendor unreachable after 3 attempts over 2 weeks → disclose within 30 days; if vendor engaged → up to 120 days to fix.

**Caveat:** no explicit legal safe-harbor clause in the T&Cs — worth being conservative about testing methods (no scraping/DoS-adjacent probing) since there's no reassurance if a vendor objects.

---

## huntr (Protect AI) — [huntr.com](https://huntr.com)

**Important scope-planning note:** huntr has moved off the model I had in my head (open bounty across 125+ ML supply-chain repos, split into Model File Vulnerabilities / Open Source Vulnerabilities programs — `/bounties` now 404s). It is now a **gamified challenge platform**: time-boxed events against a specific hosted AI agent, scored on a leaderboard, cash paid to top finishers rather than per-valid-report.

Example current challenge ("Inside Job — Three Agents, Three Secrets", $15,000 pot, ~36 days left as of Aug 9): get three different demo agents (customer support bot, internal tool-router, docs assistant) to leak their internal tool names/parameters or a planted secret, via prompt injection / roleplay / context manipulation. Rules: **human-only, no automated tooling** (scripts/bots/scanners get you banned), one attempt typed by hand at a time.

**Implication for our plan:** huntr is no longer a good source of "pick 3 candidate targets in the wild" the way OpenAI/Anthropic/Google bounty scopes are — it's a sandboxed, purpose-built practice range, closer to the PortSwigger labs we already did than to real recon. Useful for drilling technique, not for the recon-discipline goal this week is trying to build. **Recommendation: treat huntr as a technique-practice track, and pull actual target candidates from 0DIN's model list or the OpenAI/Anthropic/Google scopes below.**

---

## OpenAI — [Bugcrowd](https://bugcrowd.com/openai) (Security) + [Safety Bug Bounty](https://bugcrowd.com/engagements/openai-safety)

**Two separate programs:**
- **Security Bug Bounty** (since Apr 2023): classic infra/app security — auth, injection, API key exposure, account security. Scope rating 3/4 on Bugcrowd.
- **Safety Bug Bounty** (since Jul 2025): agentic risk, MCP, exposure of OpenAI proprietary info, account/platform integrity abuse. $200–$6,500 per finding, $20,000 max researcher payout.

**Explicitly out of scope for BOTH, no exceptions without separate verifiable security impact:**
- Jailbreaks/safety bypasses (DAN-style prompts, "say bad things," "tell me how to do bad things," malicious code generation)
- Hallucinations (model "pretending" to do things, fake secrets, fake code execution)
- Sandboxed Python interpreter code execution (intended feature) — includes documented sandbox fingerprints (kernel version, `whoami` output) they expect you to check before claiming RCE
- Agent Mode sandbox escapes that stay inside the container (same fingerprint-check requirement)
- Container tool / GPT-5 shell tool root access (sandboxed via gVisor, not a real privesc)

Route model-content issues to the **Model Behavior Feedback form** (openai.com/form/model-behavior-feedback) instead — not rewarded, but that's the intended channel.

**Process rules worth noting:** test only your own accounts, don't disrupt others, disclosure embargo up to 90 days, safe harbor only if disclosure is "unconditional" (no extortion/threats).

---

## Anthropic — [HackerOne](https://hackerone.com/anthropic)

**Scope:** Claude.ai, Anthropic API, Claude Code (CLI + IDE extensions), official iOS/Android/Desktop clients, MCP integrations, support.anthropic.com, infra/internal apps, leaked employee API keys (report only — don't test validity beyond auth/deauth).

**Core vs Non-Core assets** materially changes payout — Core (Critical): $7,500–$10,000; Non-Core (Critical): $3,000–$5,000. Full band:

| Severity | Non-Core | Core |
|---|---|---|
| Low | $100–$250 | $250–$750 |
| Medium | $250–$750 | $1,000–$2,500 |
| High | $1,000–$2,500 | $3,000–$5,000 |
| Critical | $3,000–$5,000 | $7,500–$10,000 |

90-day averages: Low $190, Medium $1,104, High $3,563, Critical $8,000.

**Explicitly out of scope:** model content/responses — jailbreaks, harmful content generation, hallucinations. These are **not** rewarded under this program.

**Separate track:** a dedicated **model safety bounty** exists for novel, universal jailbreaks in high-risk domains (CBRN, cybersecurity) — up to $15,000. That's the right venue if a jailbreak is the actual finding, not the general HackerOne program.

**Claude Code specifics:** in scope for critical vulns involving unauthorized command execution, invisible tool usage, permission-modal bypass. Note: Anthropic is mid-remediation on permission-modal bypasses specifically, so new reports there may get closed as duplicates of the tracked effort — still worth submitting, just set expectations.

**Program mechanics:** one vuln per report (unless chaining is needed to show impact), first-to-report wins on duplicates, $100 flat for doc-only fixes, GitHub Action vulns in in-scope repos treated as non-core unless they hit core assets, AI-written/scanner false positives get closed at Anthropic's discretion — need a working PoC.

---

## Google AI VRP — [Bug Hunters](https://bughunters.google.com/about/rules/google-friends/ai-vulnerability-reward-program-rules)

**Scope:** AI features across three product tiers — **Flagship** (Search, Gemini Apps, Workspace core: Gmail/Drive/Meet/Calendar/Docs/Sheets/Slides/Forms), **Standard** (AI Studio, Jules, Workspace non-core like NotebookLM), **Other** (everything else, excluding acquisitions and third-party/OSS apps). Vertex AI / Cloud AI issues route to the separate Cloud VRP instead.

**Qualifying categories (S = "Sensitive"/severe, A = "Abuse"), reward = category × tier:**

| Category | Flagship | Standard | Other |
|---|---|---|---|
| S1: Rogue Actions (unconfirmed state-changing action, e.g. indirect injection unlocks a door) | $20,000 | $15,000 | $10,000 |
| S2: Sensitive Data Exfiltration (PII/SPII leak without user approval) | $15,000 | $15,000 | $10,000 |
| A1: Phishing Enablement (persistent HTML injection on Google-branded page) | $5,000 | $500 | credit |
| A2: Model Theft (exfiltrate confidential model params — "extremely rare") | $5,000 | $500 | credit |
| A3: Context Manipulation, cross-account, hidden & persistent | $5,000 | $500 | credit |
| A4: Access Control Bypass (limited impact) | $2,500 | $250 | credit |
| A5: Unauthorized Product Usage (enable paid feature free) | $1,000 | $100 | credit |
| A6: Cross-user DoS (limited requests, persistent) | $500 | $100 | credit |

Report-quality multiplier: 0.8x / 1x / 1.2x based on write-up completeness. Novelty bonus +$1k–$5k at panel discretion.

**Explicitly non-qualifying (report via in-product feedback instead, not the VRP):**
- Policy-violating content generation within attacker's own session (jailbreaks, safety/alignment bypasses)
- Guardrail bypass techniques themselves (obfuscation, translation, roleplay) absent a security outcome
- Harm already achievable with existing non-AI tools
- Hallucinations / factual inaccuracy
- Preamble/system-prompt extraction when non-sensitive
- Sandboxed code execution (Gemini's code interpreter)
- Insecure MCP server interaction *when the victim configured it themselves*

**Read for target-picking:** this is the clearest "what counts as a real finding" rubric of the five — S1/S2 (rogue action, data exfil via indirect injection) is exactly the attack shape Thursday/Friday's baseline + injection probing should aim to demonstrate, regardless of which program ends up being the actual target.

---

## Action for Tuesday (candidate target selection)

Given the above, strongest candidates for "3 targets within scope" are ones where:
1. There's a clear **security boundary** to cross (data exfil, unauthorized action, permission bypass) — not just guardrail content.
2. The program pays for that boundary crossing specifically (Google AI VRP S1/S2, Anthropic Core-asset MCP/Claude Code, or a named model on 0DIN's list for interpreter jailbreak / content manipulation).
3. Scope is unambiguous (named asset list, not "any AI feature") so testing stays inside authorized boundaries.

Deferred decision on the actual 3 targets to the Tuesday recon session — will map attack surface (URLs, endpoints, LLM-touching features) per target then.
