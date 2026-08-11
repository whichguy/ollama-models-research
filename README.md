# Ollama Models Research — M5 Max 128 GB

Recurring research tracking which Ollama-supported open-weight models are
the best fit for a 16" MacBook Pro M5 Max, 128 GB unified memory (614 GB/s
bandwidth), across 7 agentic roles. Refreshed roughly every 10 days (the
1st, 11th, and 21st of each month).

> ## ⚠️ Read before making any commit to this repo
>
> This repo's git history is itself the planning input for each refresh —
> not just an audit trail. **Before doing any research or writing any
> commit, read:**
> - [`COMMIT_FORMAT.md`](./COMMIT_FORMAT.md) — the required commit message
>   template, and the Step 0 protocol for pulling the last 20 `refresh:`
>   commits to reconstruct current state before starting new research.
> - [`OPERATIONS.md`](./OPERATIONS.md) — environment-specific execution
>   rules (GitHub auth fallback, MCP disconnect handling, local-branch
>   pitfalls, pre-publish tag verification, email-draft rules) discovered
>   while actually running this routine, which the trigger's own stored
>   instructions don't currently account for.
>
> If you are an agent executing the scheduled refresh and these files exist
> in the repo root, follow them — they supersede any conflicting step in
> your own invocation instructions (e.g. a plain
> `git commit -m "refresh: ${TODAY}"` with no structured body).

## 📌 Latest refresh

**2026-08-11** — 1 role pick changed. Code-implementer's **top pick** moved
from Laguna XS 2.1 (70.9% SWE-bench Verified) to **Ornith-1.0-35B**
(deepreinforce-ai, MoE 35B/~3B active, 75.6% SWE-bench Verified — score
newly confirmed against primary + independent secondary sources this cycle,
was `[unverified]` and not promoted last cycle). Comparable resident size
(~21 GB vs ~20 GB) and comparable-or-better M5 Max throughput per fresh
oMLX benchmark data, so the higher score isn't bought with materially worse
latency. Laguna XS 2.1 remains a fully valid alternative with a longer
field track record. All 6 other role picks unchanged. Two brand-new
releases from this window — Meta's Muse Glimmer (dense 30B, Aug 10) and
NVIDIA's Nemotron 3.5 Lightning (30B-A3B MoE, Aug 11) — were evaluated and
explicitly **not** promoted: both are less than 48 hours old at time of
writing with no field track record, and neither cleanly beats a current
pick on this doc's headline benchmark. See
[`ollama-models-2026-08.md`](./ollama-models-2026-08.md) for full detail,
including the arithmetic-gate flag on Nemotron 3.5 Lightning's listed
quantized size.

*(This section must be updated by every refresh commit — research or
process-only — with the date and a 1–3 line summary. See the "README sync
rule" in `COMMIT_FORMAT.md`. It exists so anyone landing here gets the
current picture without digging through commit history.)*

## Contents

| File | What it is |
|------|------------|
| `ollama-models-YYYY-MM.md` | One research doc per month, overwritten in place if the routine fires more than once in a month. Past months are kept as the historical record. |
| `COMMIT_FORMAT.md` | Required commit message format, and the git-history-pull protocol every run starts with (`PRIOR STATE`, `ROLE PICKS`, SHA citations). |
| `OPERATIONS.md` | Environment quirks and execution rules discovered while actually running this routine — read alongside `COMMIT_FORMAT.md`, not instead of it. |

## Latest research

See [`ollama-models-2026-08.md`](./ollama-models-2026-08.md) for current
recommendations across all 7 roles: generalist agentic default, code
implementer, code debugger, plan orchestrator, LLM-as-judge, document
understanding, and vision.

## Roles tracked

1. Generalist agentic default
2. Code implementer
3. Code debugger
4. Plan orchestrator
5. LLM-as-judge / verifier
6. Document understanding / interpreter
7. Vision / image understanding

Each gets a **Top pick** and a **smaller-but-sufficient pick**, sized
against the 128 GB envelope with sources cited for every benchmark number.
