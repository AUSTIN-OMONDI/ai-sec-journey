# Week 12 Retrospective

## What I built
- vuln-rag-001: a real, deliberately vulnerable FastAPI RAG service
  (system prompt leak + indirect injection), replacing an empty repo shell.
- A custom PyRIT PromptTarget wrapping an arbitrary HTTP endpoint — the
  skill that lets PyRIT attack any reachable target, not just OpenAI.
- Scored direct-vs-indirect attacks (FAILURE vs SUCCESS) via PyRIT.
- A three-way tool comparison (manual / PyRIT / Garak) against a target
  with known ground truth, including a verified false-positive analysis.
- Blog 3 outline + three notes files. The technical spine is done.

## Most interesting thing I learned
Garak reported a 1.15% "success" rate on apikey.GetKey. On reading the
actual transcripts, all of it was false positives — the detector scored
refusals as leaks because they contained key-vocabulary. The real leak
rate was zero. The lesson generalizes past Garak: a tool's summary metric
is a claim, not a verdict. Read the primary evidence. This applies to any
scanner, any dashboard, any assistant — including the AI advising me.

## What I avoided that I shouldn't have
Committing work promptly. I lost Wednesday's file writes to a reboot
because they were never committed. I built the habit late (commit before
starting the server, not after). Also: I let the parent ~/aisec repo
accumulate untracked junk (pyrit-real/, copytest files) instead of keeping
it clean as I went.

## What I'd do differently
Fix the environment once, up front, instead of re-solving the same
problems each session. This week, roughly half of every session went to
friction: DNS resets, venv confusion, env-var naming (OPENAI_API_KEY vs
OPENAI_CHAT_KEY), git auth, stateful-target contamination. Most were
one-time costs I paid repeatedly because I didn't write the fix down.
An environment-state note and a couple of aliases would have reclaimed
hours.

## On track for the phase?
Behind the calendar by ~1 day, ahead on substance. The schedule's Week 12
deliverable was "Garak comparison notes." I have that plus a working
vulnerable app, a reusable attack harness, scored exploits, and a
verified three-tool comparison — more than the plan asked for. The
roadmap says not to cram to catch up; I'm dropping the Friday PyRIT-Ship/
Burp integration (lowest-value item) and carrying the momentum forward.

## The real skill this week taught
Not "how to use PyRIT" or "how to use Garak" — those were the surface.
The actual skill was distrusting summaries and verifying against ground
truth: confirm a clean baseline before attacking, reset stateful targets
between tests, read transcripts instead of trusting a success-rate column,
and check a library's real interface with inspect.signature instead of
trusting tutorials (or an AI) that describe an older version.
