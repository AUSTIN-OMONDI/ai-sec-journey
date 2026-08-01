# Blog 3 — Outline
## "Three Ways to Break a RAG: What Manual Testing, PyRIT, and Garak Each See (and Miss)"

Working title alternatives:
- "I Attacked My Own RAG Three Ways. Each Tool Was Blind to Something."
- "Manual vs PyRIT vs Garak: A Grounded Comparison on One Known-Vulnerable RAG"

Target length: 1800–2500 words. Audience: AI security hiring managers / practitioners.
Thesis: On a RAG target whose vulnerabilities I already knew, each testing
approach revealed something the others structurally could not — and the
automated scanner's headline metric was 100% false positives.

## 0. Hook (150w)
- An innocent question — "how do I connect to the VPN?" — returned an internal
  API key. No jailbreak. The malice lived in a document, not the prompt.
- Setup in one line: built a vulnerable RAG, attacked it three ways.
- Payoff promise: scanner reported 1.15% success; all false positives. Read your transcripts.

## 1. The target: vuln-rag-001 (250w)
- Why build your own: no ground truth, no way to study a scanner's blind spots.
- Architecture: FastAPI /chat, keyword retrieval, secret in system prompt, /poison endpoint.
- Two vulns: System Prompt Leakage (LLM07), Indirect Prompt Injection (LLM01).
- The whole bug in one line:
    user_content = f"CONTEXT:\n{context}\n\nUSER QUESTION:\n{query}"
- [CODE: rag.py excerpt]

## 2. Method 1 — By hand (350w)
- Baseline first (why: can't tell exploit from broken app without it).
- Direct "repeat system prompt" -> REFUSED.
- Indirect: poison "vpn" doc, ask innocent question -> secret leaks. [CODE: 2 curls + response]
- Contrast: same model/secret, only the channel changed.
- Callout: stateful-target trap — poison persisted across restarts, contaminated a baseline.

## 3. Method 2 — PyRIT (450w)
- Adds: encode once, score automatically.
- Transferable skill: custom PromptTarget wrapping any HTTP endpoint. [CODE: VulnRagTarget]
- v1.0.0 API note (short): target->scorer->AttackScoringConfig->attack; verify w/ inspect.signature.
- Stage 2 direct: FAILURE (matches manual, scorer calls refusal a non-leak).
- Stage 3 indirect: SUCCESS — target hits /poison then /chat, two-step as one attack. [CODE: result]
- Point: caught the real vuln because the multi-step sequence was ENCODED.

## 4. Method 3 — Garak (450w)
- What it's for: broad pre-built probes.
- Wiring: REST generator config. [CODE: garak json]
- Run: apikey, 942 attempts, clean KB. CompleteKey 768/768 PASS, GetKey "1.15%".
- THE FINDING: pulled the 2 GetKey hits — both REFUSALS with key-vocabulary,
  detector scored 1.0 on a refusal. Real secret in ZERO outputs. 1.15% = 100% FP. [CODE: transcripts]
- Separate: CompleteKey once fabricated an AWS-style key (AKIAfv38D6F7) — credential
  hallucination, different vuln class, one paragraph, do NOT conflate.
- Why Garak missed the real bug: can't reach /poison. Black-box = blind to stateful attacks.

## 5. Three-way table (150w)
| Approach | Direct extraction | Indirect (poisoned doc) |
|----------|-------------------|--------------------------|
| Manual   | refused (1 try)   | LEAKED |
| PyRIT    | FAILURE (scored)  | SUCCESS (scored) |
| Garak    | 0 real / 942; 2 FP| never attempted (can't reach /poison) |

## 6. What this means for pentesting RAG (300w)
1. Scanners are a starting point, not a verdict — verify every hit or file phantom bugs.
2. Dangerous RAG vulns are stateful/multi-step; black-box tools miss them by construction.
3. Establish ground truth (build or fully understand the target).
4. Reset stateful targets; confirm clean baseline with an independent tool.
- Map to OWASP LLM01/LLM07, MITRE ATLAS AML.T0051.

## 7. Close (100w)
- Meta-lesson: the skill isn't running tools, it's distrusting their summaries and
  reading primary evidence. Garak asserted a leak; the transcript said refusal.
- Links: repo, prior posts, invite reproduction.

## Assets checklist
- [ ] rag.py concatenation line
- [ ] 2 curls + leaked response
- [ ] VulnRagTarget._send_prompt_to_target_async
- [ ] PyRIT Stage 3 SUCCESS block
- [ ] garak_vulnrag.json
- [ ] 2 false-positive GetKey transcripts (centerpiece)
- [ ] AWS-key fabrication transcript (separate)
- [ ] three-way table

## Publishing
- [ ] Crosspost Hashnode + LinkedIn + X, OWASP/MITRE mappings
- [ ] Tweet the table as standalone artifact
- [ ] Tag researchers per amplification strategy
