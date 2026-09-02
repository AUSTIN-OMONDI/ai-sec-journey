# 17 - Disclosed-Report Study (0din + huntr)

Week 16, Monday · 2026-09-01 · (Week 16 Fri task, pulled forward to inform the pivot)

Goal: stop picking targets on intuition. Read what actually gets **accepted and
paid**, and let that choose the target + technique.

---

## 0din.ai — recent disclosures (as of Aug 2026)

Source: [0din.ai/disclosures](https://0din.ai/disclosures)

| Date | Title | Attack class | Technique | Severity |
|---|---|---|---|---|
| Aug 18 | "Umlaut Encoding" | Guardrail jailbreak | umlaut char-substitution + newline injection | Low |
| Aug 13 | "Plagiarism Detection" | Guardrail jailbreak | authority impersonation + task reframing | Low |
| Aug 11 | "LISP Documentation Transform" | Guardrail jailbreak | transform copyrighted text into LISP format w/ char replacement | Low |
| Jul 29 | "Inverted Rules Chemistry Video" | Guardrail jailbreak | inverted-world logic + obfuscated chemical refs | Low |

**Pattern:** 100% guardrail jailbreaks. Creative *linguistic* prompt-craft
(encoding, reframing, format transforms) against multiple models. Reward bands:
guardrail jailbreak $500–1000 (must reproduce across ≥2 categories). Low $/find,
high-volume creative grind. **Skill = prompt artistry, not code.**

## huntr.com — recent disclosures

Source: [blog.huntr.com](https://blog.huntr.com/)

| Vuln type | Project | Technique |
|---|---|---|
| **RCE (deserialization)** | Keras / TensorFlow | malicious model files, unsafe deserialization (CVE-2025-1550) |
| **RCE / path traversal** | model file formats (.nemo/.keras/.gguf/.pth) | archives that `extractall()` blindly ("archive slip") |
| **RCE (pickle)** | PyTorch, general ML | `pickle.load()` on untrusted model = RCE |
| **RCE (code exec)** | Keras Lambda layers | malicious lambda layer in model file |

**Pattern:** traditional appsec (RCE, path traversal, deserialization, SSRF) in
**open-source AI/ML tooling**. Concrete CVEs. **Skill = reading code for unsafe
sinks** (`pickle.load`, `extractall`, `eval`, `os.system`, `yaml.load`,
unauth endpoints, SSRF).

---

## The synthesis (why Week 15 failed + the fix)

The two programs reward **opposite skillsets**, and Week 15 fell in the GAP
between them:

- I was doing **prompt injection against hardened production chatbots** — which is
  neither 0din's payable class (that's *jailbreaks*, a different thing) nor huntr's
  (that's *code vulns in tooling*). No wonder it paid nothing.

| Lane | Program | Payable class | Skill it rewards | Fit for me |
|---|---|---|---|---|
| **A** | 0din | guardrail jailbreaks / content manipulation | linguistic prompt-craft | OK, not my edge; low $/grind |
| **B** | huntr | RCE / path-trav / deser / SSRF in OSS AI/ML tools | **code auditing + ML-stack knowledge** | **STRONG — this is my differentiator** |

**DECISION (confirmed 2026-09-01): Lane B (huntr) is PRIMARY.** It IS the
builder/attacker intersection — someone who builds LangChain/LangGraph/RAG/MCP
systems in Python auditing that same class of tooling for classic vulns. Findings
are unambiguous (RCE is RCE, not a judgment call), the surface (100s of
under-audited OSS ML projects) is vast and thin-defense, my SWE background is a
direct weapon, AND huntr logs in via GitHub (AUSTIN-OMONDI) — no CAPTCHA/login
friction.

**0din (Lane A) SHELVED**, not just secondary: platform friction (heavy CAPTCHAs,
slow password-reset support, failed login) confirms it's not worth the hours.
Revisit only if the login is ever fixed and a target obviously invites a jailbreak.

---

## huntr program structure (confirmed 2026-09-01)
huntr runs THREE things (not just the challenge arena notes/08 feared):
- **OSV (Open Source Vulnerabilities)** — RCE/path-trav/SSRF/deser in OSS AI/ML
  apps & libraries. **125+ in-scope repos, up to $50k critical. → PRIMARY LANE.**
- **MFV (Model File Vulnerabilities)** — deserialization/RCE in model file formats
  (.pkl/.keras/.pth/.gguf). → secondary.
- **Challenge arena** — time-boxed CTF (e.g. "Inside Job — Three Agents, Three
  Secrets", $15k; agentic multi-agent = on-brand). → opportunistic.
Login: GitHub (AUSTIN-OMONDI). Backed by Protect AI; 5th-largest CVE naming authority.

Next: log in → pull OSV in-scope repo list + guidelines → pick first audit target.

## Techniques to try (Lane B starter kit)
Audit an OSS AI/ML project for these sinks:
- **Deserialization:** `pickle.load`, `torch.load`, `joblib.load`, `yaml.load`
  (unsafe), `keras` model load, custom archive extraction (`tarfile.extractall`,
  `zipfile.extractall` without path checks → archive slip / Zip Slip).
- **Code exec:** `eval`, `exec`, `Lambda` layers, template injection, `subprocess`
  with shell=True.
- **SSRF:** any server-side fetch of a user-supplied URL (model download,
  webhook, connector, RAG ingestion of a URL).
- **Path traversal:** file paths from user input in load/save/serve endpoints.
- **AuthN/Z:** unauth admin/ingest endpoints (cf. `/poison` in vuln-rag-001).
