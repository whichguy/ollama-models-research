# Commit Message Format — Ollama Models Refresh Routine

Git history is the primary planning input for this routine, not just an
audit trail. Every refresh run starts by reading the last 20 `refresh:`
commits to reconstruct current state before doing any new research. That
only works if commit bodies are **structured and greppable**, not prose —
so every refresh commit MUST include a machine-parseable `ROLE PICKS` block
in addition to the human-readable narrative sections.

See `OPERATIONS.md` for environment-specific execution notes (GitHub auth
fallback, local-branch mismatches, the fact that no session-level scheduler
governs this routine) — read it alongside this file, not instead of it.

---

## Step 0 (every run, before research): pull history

Run this before touching the research scope. Filter on the commit
**subject**, not on file paths — path filtering also matches docs-only
commits (e.g. spec updates to this file) that have no `ROLE PICKS` block and
just add noise to the window:

```bash
git log -20 --grep='^refresh:' --format='%H %ad%n%s%n%b%n===END===' --date=short
```

From the returned commits:
1. Take the **most recent commit that actually contains a `ROLE PICKS`
   block** — skip any matched commit that lacks one (defensive: a stray
   commit could match the grep without following the template). That block
   is the current believed state per role.
2. Walk backward through the remaining commits to find, per role, the
   commit where that pick last *changed* (the most recent prior line with a
   different model for the same role) — note that commit's **short SHA**
   (first 7 chars of `%H`) alongside its date and the matching `CONCLUSION`
   line.
3. Build this into the `PRIOR STATE` section of the new commit (below),
   citing the SHA — this is the diff basis, replacing the old "find most
   recent prior ollama-models-*.md file" heuristic, and it is authoritative
   even if a file was later corrected without a matching filename change.
4. If fewer than 20 `refresh:` commits exist (or none have a `ROLE PICKS`
   block), say so explicitly — do not fabricate a baseline. See "Baseline
   anchor" below for this repo's specific history.

**SHA-citation rule:** any time this run's commit references a prior finding,
assumption, or pick as a reason for something (in `PRIOR STATE`, `VALIDATED`,
or `DISAVOWED`), cite the short SHA of the commit that established it —
`(see a1b2c3d)`. That SHA is what lets a future reader run `git show a1b2c3d`
or open `https://github.com/<owner>/ollama-models-research/commit/a1b2c3d`
and land directly on the original reasoning instead of re-deriving it.

---

## Commit message template

```
refresh: YYYY-MM-DD — N picks changed

INTENT
<1–2 sentences: hardware target, research window, scope of roles/families
covered this run.>

PRIOR STATE (from git log, last <=20 refresh commits)
<role>: <model> (since YYYY-MM-DD, see <short-sha>)
<role>: <model> (since YYYY-MM-DD, see <short-sha>)
... one line per role, sourced from Step 0, each citing the commit that set
it. Write "no prior history" if this is the first run.

LEARNED
- <Model> (<Source>, <Date>): <key fact, with benchmark numbers where
  available>
- ... bulleted, one finding per line, always cite source + date

VALIDATED
- <prior assumption or pick that held up this run, with evidence>
  (see <short-sha> for the commit that originally made the claim)
DISAVOWED
- <prior assumption or pick that was proven wrong/superseded, with evidence>
  (see <short-sha> for the commit being superseded — write "none" rather
  than omitting the heading if nothing was disavowed)

ROLE PICKS (this run — authoritative until the next refresh)
<role-slug>: <model> (unchanged|changed from <old model>: <one-line reason>)
<role-slug>: <model> (...)
... exactly one line per role, every role every run, even if unchanged.
Role slugs are fixed: generalist, code-implementer, code-debugger,
plan-orchestrator, judge, document-understanding, vision

CONCLUSION
<2–4 sentences: net effect of this run — what changed and why, what the
reader should act on immediately (new pulls, retirements, caveats).>

Refs: ollama-models-YYYY-MM.md
```

Subject line carries `N picks changed` (count of `changed from` lines in
`ROLE PICKS`) so `git log --oneline` alone shows which runs were consequential
without opening the body.

Process-only commits (fixing this spec, backfilling history, tooling notes —
no new research) still use the `refresh: YYYY-MM-DD` subject prefix so
Step 0's grep finds them, but should say so plainly in `INTENT` and set
`LEARNED`/`VALIDATED`/`DISAVOWED` to describe the process finding rather than
model research, e.g. `LEARNED: - none — process fix only`.

---

## Why `ROLE PICKS` exists as a separate block

`CONCLUSION` is for a human skimming one commit. `ROLE PICKS` is for the
*next agent run* doing Step 0 — it needs `role: model (status)` lines it can
extract without re-parsing paragraphs. Never let the two drift: if
`CONCLUSION` says a pick changed, `ROLE PICKS` must show `changed from ...`
on that role's line, and vice versa.

---

## Baseline anchor for this repo

The very first research commit, `09bba13` (2026-07-18), predates this spec
entirely — its message is the bare subject line `refresh: 2026-07-18` with
no body, because `COMMIT_FORMAT.md` didn't exist yet when it was written.
Step 0's grep will match its subject but find no `ROLE PICKS` block in it —
per the skip rule above, ignore it and keep walking.

The commit that added this sentence (see the repo's `refresh:` history
around 2026-07-18, subject containing "baseline backfill") retroactively
supplies the first real, parseable `ROLE PICKS` block, carrying forward the
exact picks from `09bba13`'s research unchanged. Treat **that** commit, not
`09bba13`, as the baseline anchor for `PRIOR STATE` going forward.

---

## Example (2026-07-18 initial research run)

```
refresh: 2026-07-18 — 0 picks changed

INTENT
Initial research run to establish baseline Ollama model recommendations for
Apple M5 Max 128 GB (614 GB/s). Covered 7 agentic roles, surveyed the
2026-05-18 – 2026-07-18 window, sized every candidate against the 128 GB
unified-memory envelope.

PRIOR STATE (from git log, last <=20 refresh commits)
no prior history — first run

LEARNED
- Laguna XS 2.1 (Poolside, 2026-07-02): 33B/3B-active MoE; 70.9% SWE-bench
  Verified — highest coding score per GB in Ollama library at this date
- Kimi K2.7 Code (Moonshot, 2026-06): 1T-param model; no GGUF exists yet;
  cloud-only until a quantized build ships
- Gemma 4 12B Unified (Google, 2026-06): fills mid-size gap; natively
  multimodal; 77.2% MMLU Pro
- Ollama v0.32.0 (2026-07-11): ships native MLX engine on Apple Silicon;
  Gemma 4 generation ~90% faster
- Qwen3.6-27B (Alibaba, 2026-04-22): 27B dense, 256K ctx, dual-mode
  thinking, 77.2% SWE-bench Verified — within 3.7 pts of Claude Opus 4.6
- Llama 4 Scout (Meta, 2026-04): only locally-runnable 10M-context model;
  17B active / 109B total MoE; ~55 GB Q4 on M5 Max
- DeepSeek-R1-Distill-Qwen-32B: highest MATH score of any sub-40 GB local
  model (72%); always-on CoT at ~20 GB Q4
- MLX delivers 40-80% higher tok/s than llama.cpp on M5 Max (LLMCheck,
  2026-07)
- M5 Max ~28% faster than M4 Max across all sizes (614 vs 546 GB/s bw)

VALIDATED
- MoE-A3B pattern: ~17 GB loaded, ~3B active/token, quality competitive
  with dense 30B at faster inference (Laguna XS 2.1, Qwen 3.5 30B-A3B)
- MLX framework advantage: 40-80% throughput gain over llama.cpp confirmed
DISAVOWED
- none — first run, no prior baseline to test against

ROLE PICKS (this run — authoritative until the next refresh)
generalist: Qwen3.6-27B (unchanged)
code-implementer: Laguna XS 2.1 (unchanged)
code-debugger: DeepSeek-R1-Distill-Qwen-32B (unchanged)
plan-orchestrator: Llama 4 Scout (unchanged)
judge: Qwen3.6-27B (unchanged)
document-understanding: Gemma 4 26B MoE (unchanged)
vision: Llama 4 Scout (unchanged)

CONCLUSION
Established a 7-role stack for M5 Max 128 GB. Laguna XS 2.1 is the breakout
new entrant this cycle (top code-implementer pick, released 2026-07-02).
Kimi K2.7 Code is cloud-only — watch for a GGUF build next cycle. All other
picks are first-assignment, not yet tested against a prior baseline.

Refs: ollama-models-2026-07.md
```

(Note: the actual `09bba13` commit predates this spec and does not contain
this body — see "Baseline anchor" above.)
