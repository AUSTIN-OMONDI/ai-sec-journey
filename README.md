# AI Security Journey — Austin Omondi (Austoo)

A public research journal documenting a focused transition from **AI engineer → AI security researcher**: hands-on labs, attack-surface mapping, bug-hunting methodology, tooling, and original research notes.

I build agentic LLM systems (LangChain / LangGraph / RAG) and then try to break them — the builder/attacker intersection. Nairobi-based.

## Repository map

| Path | Contents |
|---|---|
| `notes/` | Research notes — bounty-program scoping, target attack-surface maps, injection probe plans, disclosed-report studies, tooling notes (PyRIT/Garak) |
| `labs/` | Hands-on lab attempts and write-ups |
| `writeups/` | Longer-form write-ups (vuln-langgraph, vuln-rag, RAG methodology) |
| `week12-pyrit/` | PyRIT attack scripts against my own vulnerable apps |
| `owasp-contributions/` | Source for my OWASP GenAI contributions (see below) |
| `tools/`, `vulnerable-apps/`, `screenshots/` | Supporting material |

## Highlights

- **OWASP GenAI contribution** — first LangGraph code sample for the Agentic Top 10 (**ASI02 Tool Misuse & Exploitation**), built from my own vulnerable multi-agent system: [PR #18](https://github.com/GenAI-Security-Project/GenAI-Agent-Security-Initiative/pull/18). Source in `owasp-contributions/`.
- **Vulnerable apps I built to attack** — [`vuln-langgraph-001`](https://github.com/AUSTIN-OMONDI/vuln-langgraph-001) (multi-agent LangGraph) and [`vuln-rag-001`](https://github.com/AUSTIN-OMONDI/vuln-rag-001) (RAG).
- **Published write-ups** — blog at [hashnode.com/@austoo](https://hashnode.com/@austoo); repo [`blog-posts`](https://github.com/AUSTIN-OMONDI/blog-posts).
- **Methodology** — a practical RAG-application pentest methodology mapped across trust boundaries.

## Focus areas

Indirect prompt injection · agentic / multi-agent attack chains · RAG attack surface · tool/function-calling abuse · output-handling & data exfiltration · open-source AI/ML supply-chain security (deserialization / SSRF / path traversal) · red-team tooling (PyRIT, Garak, Promptfoo).

---

*A living log — notes are working documents, not polished papers. Some active vulnerability research is withheld pending responsible disclosure.*
