# 16 - Week 16 Decision: Report or Pivot

Week 16, Monday · 2026-09-01

## Honest assessment: do I have a real finding?

**No.** Week 15 produced zero boundary-crossing findings across two targets:

- **Gemini Workspace (output handling / image exfil)** — [notes/14](14-gemini-output-handling.md): deprioritized, inconclusive-but-mitigated. Explicit refusal to render external images ("custom or unverified URLs"). Class looked mitigated; testing also hit tooling friction.
- **Claude.ai Gmail connector (indirect injection → tool abuse / exfil)** — [notes/15](15-claude-tool-abuse.md): **hardened, evidenced negative.** Model-level provenance awareness (won't follow embedded instructions), source-independent output-handling (won't emit auto-fetch image markdown), and all 23 write tools human-gated ("Needs approval").

There is nothing to write up as a bug report. A fabricated or borderline "finding"
would get auto-closed and burn credibility. Not doing that.

## Decision: TRACK B — PIVOT

**Root cause of the null week: target selection, not skill.** I attacked the two
most-defended AI surfaces in existence (two frontier labs' own flagship products).
Both are defended in layers. Grinding flagship happy-paths is low expected value.

## The pivot (what changes)

1. **Evidence-first, not intuition-first.** Before picking the next target, study
   what actually gets *accepted and paid* on 0din.ai and huntr.com. Let real
   disclosures drive target + technique choice. (Pulling Week 16 Friday's task to
   the front of the week — it should *inform* the pivot, not close it.)
2. **Move to thin-defense surfaces:** third-party / community MCP connectors
   (in scope as the injection vector even when the model resists), custom GPTs /
   GPT Actions, AI features on smaller SaaS, and composition bugs.
3. **Keep banking footprint regardless of bug luck:** post-04 (null-result
   teardown) already drafted; a novel-attack-class writeup is the Week 16 Sat
   deliverable.

## Value already banked this cycle (not a wasted week)
- OWASP PR #18 shipped (public, name-attached contribution).
- Two hardened surfaces characterized with evidence (reusable knowledge + blog).
- post-04 drafted.
- Tradecraft learned: multipart-MIME/RAW inspection, test-validation discipline,
  isolating model-gate vs client-render.

## Resequenced Week 16 plan
- **Mon:** this decision + begin disclosed-report study (0din + huntr).
- **Tue:** finish study → pick ONE thin-defense target + a technique seen working.
- **Wed–Thu:** hunt that target with that technique (Track B).
- **Fri:** more disclosed-report study / technique notes as needed.
- **Sat:** apply one new technique + outline blog post-05.
