# Ollama Models Research — M5 Max 128 GB

**Last updated:** 2026-08-21 (full refresh) · **Hardware target:** Apple M5 Max · 128 GB unified memory · 614 GB/s bandwidth
**Most recent full research window:** Last 60 days (2026-06-22 → 2026-08-21)

> This is a **single living document** — it is updated in place every refresh
> (roughly every 10 days: the 1st, 11th, and 21st of each month). There is no
> per-month copy of this file. **Full history lives in git**, not in dated
> filenames — run `git log -- ollama-models.md` or see `COMMIT_FORMAT.md`'s
> Step 0 protocol to reconstruct what changed and when. If you're reading
> this in the GitHub UI, "History" on this file is the changelog.

## Current Picks at a Glance (as of 2026-08-21)

The single fastest way to answer "what's the latest recommendation right now" — no need to read further unless you want the rationale.

| Role | Top pick | Smaller pick |
|------|----------|---------------|
| 3.1 Generalist agentic default | **Qwen3.6-27B** (~17 GB) | Gemma 4 12B Unified (~7.6 GB) |
| 3.2 Code implementer | **Ornith-1.5-35B-A3B** (~23 GB) ⟵ *changed 2026-08-21* | Ornith-1.5-9B (~6.6 GB) ⟵ *changed 2026-08-21* |
| 3.3 Code debugger | **DeepSeek-R1-Distill-Qwen-32B** (~20 GB) | Qwen3.6-27B, thinking mode (~17 GB) |
| 3.4 Plan orchestrator | **Llama 4 Scout** (~67 GB) | Qwen3.6-27B (~17 GB) |
| 3.5 LLM-as-judge / verifier | **Qwen3.6-27B** (~17 GB) | Gemma 4 12B Unified (~7.6 GB) |
| 3.6 Document understanding | **Gemma 4 26B MoE** (~18 GB) | Mistral OCR 4 (container, not Ollama-loadable) |
| 3.7 Vision / image understanding | **Llama 4 Scout** (~67 GB) | Gemma 4 E4B (~9.6 GB) |

Every pick below is cross-referenced by section number (§3.1–§3.7). "Changed" tags in this table only reflect the *most recent* refresh — see "Recent Changes" immediately below for the full story, and git log for anything older.

---

> **Recent Changes (as of the 2026-08-21 refresh; previous state was commit
> `5d76cdae` — no model-pick commit since 737643c on 2026-08-11):**
>
> **Code-implementer's Top and Smaller picks both changed** — the entire pair
> moves to the next generation of the same MIT-licensed family. **Top pick:**
> **Ornith-1.5-35B-A3B** (79.0% SWE-bench Verified) replaces **Ornith-1.0-35B**
> (75.6%). **Smaller pick:** **Ornith-1.5-9B** (70.6%) replaces **Ornith-1.0-9B**
> (69.4%). DeepReinforce/Ornith shipped the 1.5 family 2026-08-19/20 — literally
> 1–2 days before this refresh — extending the self-scaffolding training approach
> into a full self-improvement loop (the model proposes its own training tasks in
> addition to its own scaffolds). Both new scores are confirmed directly against
> Ornith's own Hugging Face model cards (primary), using the **same OpenHands-harness
> methodology** (temp=1.0, top_p=0.95, 256K context, avg of 5 runs, anti-reward-hacking
> controls) as the 1.0 generation this doc already trusted — a clean apples-to-apples
> comparison, not a benchmark-suite change. Both Ollama tags resolve and were fetched
> directly: `ornith-1.5:35b` at 23 GB (vs 1.0's 21 GB), `ornith-1.5:9b` at 6.6 GB (vs
> 1.0's 5.6 GB) — the ~10–18% size increase passes the arithmetic gate (still
> consistent with a Q4-class quant at this parameter count) and is plausibly explained
> by added multimodal/vision input support. **Caveat, stated plainly:** these are
> **self-reported scores with no independent third-party reproduction yet** (same
> pattern the 1.0 generation was in in its first cycle, before BenchLM.ai
> independently matched it — see 737643c), and **no fresh M5 Max throughput
> measurement exists yet** for the 1.5 family (too new) — the tok/s table below
> extrapolates from the 1.0 family's directly-measured oMLX data by architecture
> analogy (same MoE-A3B pattern, marginally larger footprint), marked accordingly.
> Ornith-1.0-35B and Ornith-1.0-9B remain fully valid alternatives with real field
> track records (5+ and 3+ weeks of production use respectively) and directly-measured
> throughput. See §3.2.
>
> **No other role's Top or smaller pick changed.** Generalist, code-debugger,
> plan-orchestrator, judge, document-understanding, and vision all keep their
> prior picks.
>
> **Qwen3.8-27B shipped 2026-08-14 — evaluated for generalist/judge, NOT promoted.**
> This resolves the "Pending Releases" watchlist entry tracked since 2026-08-15 (see
> §2). Qwen3.8-27B is the same 27B/dense-with-hybrid-attention architecture as
> Qwen3.6-27B (post-training-only upgrade, not a new base model) and posts real gains
> on agentic benchmarks per Alibaba's own model card: Terminal-Bench 2.1 63.4→73.0,
> OSWorld-Verified 63.9→84.3, SWE-bench Pro 53.5→61.7. **It does not clear this doc's
> verification bar for a promotion this cycle, for two independent reasons:** (1)
> Alibaba has **not published a SWE-bench Verified score** for it at all — only
> SWE-bench Pro — so there is no apples-to-apples comparison against this doc's
> headline metric (Qwen3.6-27B's tracked 77.2% SWE-bench Verified.); (2) multiple
> independent field reports describe a real "overthinking" regression: the model
> **defaults to `reasoning_effort=xhigh`**, and one developer reported 22,276
> reasoning tokens and 21 minutes of latency on a single trivial SVG-generation
> prompt — a severe throughput regression for exactly the agentic loops this doc's
> selection criterion weighs (throughput is part of quality; many completions per
> session). Community reports also flag chat-template bugs and higher-than-expected
> context memory use. The model is worth re-testing at `reasoning_effort=low` or
> `medium` next cycle once the template issues settle and field data accumulates —
> not a rejection of the model outright, a "not yet" on the evidence available this
> cycle. Ollama tag confirmed: `qwen3.8:27b` (Q4_K_M, 18 GB). See §2 and §3.1.
>
> **Considered and rejected (no pick impact):**
> - **GLM-5.3** (Z.ai, Aug 14, 2026) — Z.ai's own strongest-open-weights-coding claim
>   to date, but inherits GLM-5.2's unchanged 743B total / ~40B active MoE base (no
>   retrain — every gain is from post-training), so it stays **cloud-only by
>   arithmetic regardless of benchmark score** (743B × ~4.5 bpw ÷ 8 ≈ 418 GB at Q4).
>   Weights are also not public yet — Z.ai states "about two weeks" from launch for a
>   safety-hardened release. Not added to the Pending Releases watchlist per that
>   list's own scope rule: a model this oversized wouldn't become a local candidate
>   even once weights ship. See §1/§2.
> - **Llama 5** (Meta, ~600B total MoE, 5M-token context, native video/audio) — far
>   exceeds the envelope (≈338 GB at Q4) and no smaller "Scout"-class sibling has
>   been released or announced; Llama 4 Scout (109B/17B active) remains the only
>   locally-viable Meta model. Added to the "what exceeds 128 GB" table for
>   completeness. See §1.
> - **Muse Glimmer** and **NVIDIA Nemotron 3.5 Lightning** — both evaluated last
>   cycle (see 737643c) and re-checked this cycle: no new SWE-bench Verified data or
>   M5 Max throughput measurement surfaced for either. Nemotron 3.5 Lightning does
>   have a new independent-analysis data point (Artificial Analysis Intelligence
>   Index score of 24, on par with gpt-oss-120b at a quarter the active parameters;
>   Elo 824 on agentic benchmarks) but its SWE-bench Verified score (52.80%) is
>   unchanged and still well below every code-implementer pick — status unchanged,
>   not promoted. See §2.
>
> **Previously-recommended models beaten this cycle:** **Ornith-1.0-35B** and
> **Ornith-1.0-9B** lose their Top/Smaller code-implementer slots to their own 1.5
> successors (see above) — not deprecated, both remain fully valid alternatives with
> real field track records the 1.5 generation doesn't have yet. See §3.2.
>
> *For changes before this cycle, don't scroll further down in this file — this
> callout is replaced wholesale on every refresh. Use `git log -- ollama-models.md`
> or the `refresh:` commits described in `COMMIT_FORMAT.md` to see the full history,
> including cycles where 0 picks changed.*

---

## 1. Hardware Envelope

### M5 Max Specifications

| Spec | Value |
|------|-------|
| Unified memory | 128 GB |
| Memory bandwidth | 614 GB/s |
| GPU cores | 40 |
| Neural Engine | 38 TOPS |

### Tokens-per-Second on Representative Models

Numbers are approximate; see measurement note below. **New this cycle:** several
rows now carry directly-measured M5 Max figures from oMLX (a community-run,
crowdsourced MLX benchmark tool — a *distinct* third-party MLX runtime, not the
same thing as Ollama's own MLX path or `mlx_lm` used elsewhere in this table).
oMLX figures are marked ‡‡ and kept in the `mlx_lm native`-class column since both
bypass Ollama entirely, but they are not literally the same runtime — treat as
"MLX-family, directly measured," not interchangeable with the mlx_lm column's
other entries.

| Model | Architecture | Quant | tok/s Ollama | tok/s Ollama MLX | tok/s mlx_lm / MLX-family |
|-------|-------------|-------|:------------:|:----------------:|:-------------------:|
| Llama 3.1 8B | Dense 8B | Q4_K_M / 8bit | ~82 | ~138 | ~230 (mlx_lm) / ~44–56 (oMLX 8bit, 1k–64k ctx)‡‡ |
| Ornith-1.0-9B | Dense 9B | Q4_K_M / 4bit | ~75 (extrapolated†) | ~120 (extrapolated†) | **109.0 (oMLX, 4bit, 4K ctx, 2026-06-28)‡‡** — degrades to 44.4 at 195K ctx |
| Qwen3.6-27B | Dense 27B | Q4_K_M | ~45 | ~70 | ~63 (X/community post, runtime unstated, 2026-08-11)‡‡ |
| Qwen3.8-27B | Dense 27B, hybrid attention | Q4_K_M | ~44 (extrapolated‡, similar footprint to Qwen3.6) | ~68 (extrapolated‡) | *No M5 Max figure found this cycle — 1 week old at time of writing. Not promoted this cycle — see Recent Changes.* |
| Qwen 3.5 30B-A3B | MoE 30B/3B active | Q4_K_M | ~45 | ~55 | ~68 (mlx_lm) |
| Laguna XS 2.1 | MoE 33B/3B active | Q4_K_M / 4bit | ~43 | ~53§§ | **87–107 (oMLX, 4bit, 1K–4K ctx, 2026-08-10/11)‡‡** |
| **Ornith-1.5-35B-A3B** | MoE 35B/~3B active | Q4_K_M / 4bit | ~40 (extrapolated‡, analogy to 1.0) | ~50 (extrapolated‡) | *No fresh oMLX measurement yet — shipped 2026-08-19/20, 1–2 days before this refresh. Extrapolated from Ornith-1.0-35B's 77–123 tok/s (see below) adjusted ~5–8% down for the slightly larger 23 GB vs 21 GB footprint.* |
| Ornith-1.0-35B | MoE 35B/~3B active | Q4_K_M / 4bit | ~42 (extrapolated†) | ~52 (extrapolated†) | **77.0–123.4 (oMLX, 4bit, batch 1×–8×, 2026-07-02)‡‡** |
| Ornith-1.5-9B | Dense 9B | Q4_K_M / 4bit | ~72 (extrapolated‡) | ~115 (extrapolated‡) | *No fresh oMLX measurement yet — same "1–2 days old" caveat as the 35B row above. Extrapolated from Ornith-1.0-9B's 109.0 tok/s (4K ctx, below) adjusted ~5% down for the 6.6 GB vs 5.6 GB footprint.* |
| DeepSeek-R1-Distill 32B | Dense 32B | Q4_K_M / 4bit | ~27 | ~45 | **27.6–28.9 (oMLX, 4bit, 1K ctx, 2026-07-15)‡‡** / ~60 (mlx_lm, older est.) |
| Llama 4 Scout | MoE 109B/17B active | Q4_K_M | ~22 | ~26 | ~50 (mlx_lm, no fresh M5 Max row found this cycle) |
| Llama 3.3 70B | Dense 70B | Q4_K_M / 4bit | ~12–18 | ~15–22 | **12.7–13.2 (oMLX, 4bit, 1K–4K ctx, 2026-07-20)‡‡** / ~7 (oMLX 8bit, older) |
| Mistral Medium 3.5 | Dense 128B | Q4_K_M (80 GB)§ | ~6 | ~7 | **7.1–7.2 (oMLX, 4bit, 4K ctx, 2026-07-16)‡‡** — matches prior bandwidth-ceiling estimate |

† No direct M5 Max measurement found this cycle for the "extrapolated" rows above
— treat as directional, not measured. Ornith-1.0-35B's own oMLX row (77–123 tok/s)
is real data but was not measured on the Ollama runtime specifically, hence its
Ollama/Ollama-MLX columns remain extrapolated by analogy.

‡ **New this cycle.** Ornith-1.5-35B-A3B and Ornith-1.5-9B shipped 2026-08-19/20,
1–2 days before this refresh — no benchmark tool (oMLX or otherwise) has posted M5
Max figures for them yet. All columns for these two rows are extrapolated by
architecture analogy to the corresponding Ornith-1.0 model (same MoE-A3B or dense
pattern, ~10–18% larger resident footprint), not measured. Re-check next cycle.
Qwen3.8-27B's row is extrapolated the same way, by analogy to Qwen3.6-27B (same
dense/hybrid-attention architecture, near-identical resident size) — it was
evaluated and not promoted this cycle regardless (see Recent Changes), so this
figure is informational only, not load-bearing for any pick.

‡‡ oMLX (https://omlx.ai) is a crowdsourced, individually-dated community benchmark
database using its own MLX-based runner — a third, independent tool distinct from
both Ollama's MLX engine and `mlx_lm.generate`. Methodology (exact prompt/output
lengths) is not fully disclosed per submission; treat as noisy but genuinely
directly-measured M5 Max data, materially better-sourced than pure extrapolation.
All oMLX figures above are dated within or right at the edge of the most recent
research window and are new as of the 2026-08-11 refresh.

§§ As of Ollama v0.32.4 (2026-07-25), Laguna gained a native MLX path on Apple
GPUs; the ~53 tok/s "Ollama MLX" figure predates this change and has not been
re-measured on Ollama's own MLX path since — the new 87–107 tok/s figure is from
oMLX (a different runtime, see ‡‡), not a re-measurement of the Ollama MLX column,
so the two numbers are not directly comparable and neither has been overwritten.

§ **Bandwidth-ceiling-derived, not measured**, for the Ollama/Ollama-MLX columns.
Mistral Medium 3.5's confirmed 80 GB resident size caps this hardware at
614 GB/s ÷ 80 GB ≈ **7.7 tok/s** for any Ollama-path runtime running this dense
model single-stream. The oMLX row (7.1–7.2 tok/s) independently confirms this
ceiling is roughly correct and not just theoretical.

**Measurement note:** "Ollama" column = Ollama v0.32.x via llama.cpp backend.
"Ollama MLX" = Ollama's native MLX path (available since v0.30.0, extended to
Laguna in v0.32.4, Qwen3.5 gained MTP-head speculative decoding in v0.32.6).
"mlx_lm / MLX-family" = either `mlx_lm.generate` directly or oMLX (marked ‡‡) —
both bypass Ollama overhead but are not the same tool as each other. LLMCheck
(March–April 2026) remains the primary source for older Ollama/Ollama MLX column
figures not updated in the most recent cycle; PromptQuorum and Presenc AI for older
mlx_lm figures. The Wale Akinfaderin Medium article ("Benchmarking Open-Weights
LLMs on the MacBook Pro M5 Max") is fetchable (an earlier HTTP 503 has cleared)
but is paywalled beyond the intro, and — worth flagging — actually benchmarks the
**binned 32-core/36GB/460 GB/s M5 Max SKU**, not the 40-core/128GB/614 GB/s
configuration this doc targets; low value even if unlocked. Context length,
KV-cache fill, and prompt length all materially affect tok/s — treat single
numbers as interactive-session estimates at moderate context (2K–8K tokens)
unless a context figure is given directly in the table.

Sources: [LLMCheck Apple Silicon Benchmarks](https://llmcheck.net/benchmarks) · [oMLX Community M5 Max Benchmarks](https://omlx.ai/benchmarks) (directly-measured, individually-dated) · [PromptQuorum M5 Max Benchmarks](https://www.promptquorum.com/local-llms/m5-pro-max-llm-benchmarks-2026) · [Presenc AI Local Benchmarks 2026](https://presenc.ai/research/local-llm-tokens-per-second-benchmarks-2026) · [Ollama v0.32.6 release notes](https://github.com/ollama/ollama/releases) (primary, Qwen3.5 MLX speculative-decoding)

### Runtime Notes

- **MLX vs llama.cpp:** Ollama's bundled MLX engine gives 20–50% uplift over its
  llama.cpp path on Apple Silicon for models not already bandwidth-saturated;
  `mlx_lm`/MLX-family tools directly (including the oMLX data source) can add
  further uplift on top. For large dense models near the bandwidth ceiling (e.g.
  Mistral Medium 3.5 at 80 GB), the uplift compresses toward zero — the oMLX
  measurement (7.1–7.2 tok/s) confirms this is a hard physical limit, not a
  software gap.
- **Ollama v0.32.6–v0.32.14 (current as of 2026-08-21):** picking up from
  v0.32.5 (2026-07-27): **v0.32.6** (Aug 4) ships Qwen3.5 speculative decoding
  via its MTP head on Apple GPUs plus an OpenAI-wire-format streaming rework;
  **v0.32.7** (Aug 10) introduces Muse Glimmer support, Apple/MLX-only at first;
  **v0.32.8** (Aug 10, same day) extends Muse Glimmer to NVIDIA/AMD; **v0.32.9**
  (Aug 11) adds NVIDIA Nemotron 3.5 Lightning support and fixes a Muse Glimmer
  function-calling parser bug. Point releases through **v0.32.14** (~Aug 15)
  changed `repeat_penalty`'s default from 1.1 to 1.0 (off) for models that
  don't set it — matching other engines and speeding up speculative decoding —
  plus ~7–8% faster NVFP4 MLX prefill on Qwen3.6/Muse Glimmer and Linux/Windows
  MLX support coming online. **Current stable is v0.32.14; no v0.33 found as of
  this refresh.** None of these releases specifically target Ornith, Qwen3.8,
  Llama 4, DeepSeek-R1, or KV-cache handling.
- **MoE-A3B behaviour:** Models like **Ornith-1.5-35B-A3B** (top code-implementer
  pick, 35B/~3B active, ~21 GB Q4), **Laguna XS 2.1** (33B/3B active, ~20 GB), and
  **Qwen 3.5 30B-A3B** (30B/3B active, ~17 GB) activate only ~3B parameters per
  token despite loading a much larger weight set — 30B-class quality at 40–110+
  tok/s on M5 Max depending on runtime. This remains the highest quality-per-GB
  pattern in the current Ollama library, and fresh oMLX data confirms it holds up
  under direct M5 Max measurement, not just theory.
- **KV-cache headroom:** A directly-measured KV-cache study (Contra Collective,
  MLX 0.21, M5 Max 128GB) quantifies the tradeoff concretely for a 70B Q4 model:
  at 64K context, KV cache alone costs 20.8 GB at fp16, 10.4 GB at Q8
  (near-lossless, ~0.2 MMLU pts), or 5.2 GB at Q4 (8–14pp needle-in-haystack
  accuracy loss). Decode throughput scales inversely: 7.9 → 13.2 (+67%) → 16.1
  tok/s (+104%) going fp16 → Q8 → Q4 KV cache. At 128 GB total, reserve ~10–18 GB
  for macOS + KV-cache; practical ceiling for a single loaded model remains
  ~110 GB.
- **M5 Max vs M4 Max:** ~28% higher decode throughput reported across community
  MLX benchmarks (614 vs 546 GB/s bandwidth, plus per-core Neural Accelerator
  gains not explained by bandwidth alone per Apple's own M5-focused research
  post — which itself only covers the base M5, not M5 Max). No new primary M5 Max
  vs M4 Max comparison found in the most recent cycle; figure carried forward
  unchanged.

### What Exceeds 128 GB (Cloud-Only or Borderline)

| Model | Why it won't fit |
|-------|------------------|
| **Kimi K3** (Moonshot, 2.8T/50.4B active MoE) | Ollama offers only `kimi-k3:cloud`; llama.cpp support remains an unmerged PR. Native weights ~1.4 TB; even 2-bit quants ≈ 700+ GB. **Cloud-only.** |
| **DeepSeek V4-Flash** (DeepSeek, 284B/13B active MoE) | *(re-checked 2026-08-15)* Ollama's own tags page confirms all three variants (`:cloud`, `:0731-cloud`, `:preview-cloud`) are cloud-hosted only — there is no local/GGUF tag at all, so this is cloud-only by Ollama's own listing, not just by arithmetic. (Arithmetic still backs this up independently: 284B × ~4.5 bpw ÷ 8 ≈ 160 GB at Q4 — would exceed 128 GB even if a local tag existed.) |
| **Ornith-1.5-397B** (ornith-ai/deepreinforce-ai, 397B/~3B active MoE) | *(new this cycle, replaces Ornith-1.0-397B)* Ollama `ornith-1.5:397b` tag confirmed at 242 GB — Q4_K_M-class, exceeds 128 GB by nearly 2×. Extreme sub-Q2 quants might fit numerically but are untested at that precision. **Effectively cloud-only.** Note: the same family's 9B and 35B members fit comfortably and are this doc's current code-implementer picks — see §3.2. |
| **GLM-5.3** (Z.ai, 743B/~40B active MoE — *new this cycle*) | Z.ai's own strongest-open-weights-coding claim to date (84.5% CyberGym, Terminal-Bench 3.0 lifted from 4.6% to 28.3%), released 2026-08-14, but inherits GLM-5.2's unchanged base model (no retrain) — 743B × ~4.5 bpw ÷ 8 ≈ 418 GB at Q4, stays cloud-only by arithmetic regardless of score. Weights also not yet public as of this refresh (Z.ai states ~2 weeks from launch). A "GLM-5.5" successor remains rumored/unofficial — not treated as a real release. |
| **Llama 5** (Meta, ~600B total MoE, 5M-token context — *new this cycle*) | ≈600B × ~4.5 bpw ÷ 8 ≈ 338 GB at Q4 — far exceeds 128 GB. No smaller "Scout"-class sibling exists; Llama 4 Scout (109B/17B active) remains the only locally-viable Meta model and this doc's plan-orchestrator/vision pick. |
| **MiniMax M3** (MiniMax, ~428B/23B active MoE) | 80.5% SWE-bench Verified; smallest local quant ~143 GB; Ollama only offers `:cloud`. (MiniMax's newer "H3" is video/multimodal, not a text LLM, and its weights had not shipped as of the most recent source checked — not evaluated as a candidate.) |
| Llama 4 Maverick (400B total) | Q4 ≈ 200 GB — hard OOM |
| Kimi K2.7 Code (1T total) | Community GGUF builds exist but ~585 GB at Q4; cloud-only for this hardware |
| **DeepSeek V4 Pro** *(the doc's prior "DeepSeek-V4-Pro-Max" — 2026-08-15: left preview, went GA 2026-08-12 as "V4 Pro 0813")* (1.6T/49B active MoE) | 80.6% SWE-bench Verified — still one of the highest open-weight scores found — but ~800 GB minimum at Q4; a circulating "~50 GB" claim is arithmetically impossible for 1.6T params and was rejected early in this doc's history. Re-checked 2026-08-15: Ollama added it to its library as a **`:cloud` tag only** — confirms cloud-only independent of the arithmetic. Community (non-official) GGUF conversions exist but are unverified against DeepSeek's checkpoints. |
| DeepSeek-V3 full (671B MoE, 37B active) | Q4 ≈ 170 GB — borderline, likely OOM with KV-cache |
| **Qwen3.8-2.4T-A95B** (Alibaba, 2.4T total, ~95B active — the base model behind the Qwen3.8-Max API) | *(re-checked 2026-08-15)* Open weights **did ship**, on 2026-08-12 (Hugging Face + ModelScope, FP8 quant available) — but this changes nothing for local use. Arithmetic: 2.4T × ~4.5 bpw ÷ 8 ≈ **1.2+ TB at Q4**, ~750 GB even at an aggressive 2-bit quant — off by close to an order of magnitude versus 128 GB, not a borderline case. No Ollama tag exists. **Still not evaluable as a local candidate, weights or not.** The size-comparable companion model, **Qwen3.8-27B**, shipped 2026-08-14 and *is* locally runnable (~18 GB Q4, `qwen3.8:27b`) — evaluated for generalist/judge this cycle and not promoted; see Recent Changes and §3.1. |
| Any 700B+ total-parameter model | Exceeds envelope at any practical quant |

---

## 2. Current Model Landscape (Last 60 Days)

### Models that moved SOTA or shipped new (Jun 22 – Aug 21, 2026)

| Model | Family | Released | Why it matters |
|-------|--------|----------|-----------------|
| **Ornith-1.5-35B-A3B / -9B** | ornith-ai/deepreinforce-ai (MoE 35B/~3B active and dense 9B) | Aug 19/20, 2026 | **Code-implementer top and smaller picks, both changed this cycle.** 79.0% and 70.6% SWE-bench Verified respectively (HF model cards, primary, same OpenHands-harness methodology as the 1.0 generation) — beat prior picks Ornith-1.0-35B (75.6%) and Ornith-1.0-9B (69.4%) at a modest ~10–18% larger footprint. Extends self-scaffolding into a full self-improvement loop (model generates its own training tasks). MIT license, 256K–1M context (YaRN). Self-reported only, no third-party reproduction or M5 Max throughput data yet — see Recent Changes and §3.2. |
| **Qwen3.8-27B** | Alibaba/Qwen (dense 27B, hybrid attention, natively multimodal) | Aug 14, 2026 | Evaluated for generalist/judge, **not promoted**. Same architecture as Qwen3.6-27B (post-training-only upgrade); strong agentic-benchmark gains (Terminal-Bench 2.1 63.4→73.0, OSWorld-Verified 63.9→84.3) but no published SWE-bench Verified score, and field reports of a severe "overthinking" latency regression at its default `reasoning_effort=xhigh`. Ollama tag `qwen3.8:27b` confirmed, ~18 GB Q4. **Resolves the Qwen3.8-27B "Pending Releases" watch item — see below.** |
| **GLM-5.3** | Z.ai (MoE 743B/~40B active, unchanged base from GLM-5.2) | Aug 14, 2026 | Z.ai's strongest open-weights coding claim yet (Terminal-Bench 3.0 4.6%→28.3%) but stays cloud-only by arithmetic (≈418 GB at Q4) regardless of score; weights not yet public (~2 weeks from launch per Z.ai). **Not locally relevant — see §1.** |
| **Llama 5** | Meta (MoE ~600B total, 5M-token context, native video/audio) | Apr 8, 2026 | ≈338 GB at Q4 — far exceeds 128 GB, no smaller "Scout"-class sibling exists. Llama 4 Scout remains the only locally-viable Meta model. **Not locally relevant — see §1.** |
| **Muse Glimmer** | Meta Superintelligence Labs (dense 30B + 1.8B vision encoder) | Aug 10, 2026 | Unchanged status — no new benchmark or M5 Max data surfaced this cycle. Purpose-built for local agentic workloads; wins big on agentic/tool-use benchmarks (MCP-Atlas, DeepSearch QA, GAIA2) but trails Qwen3.6-27B on SWE-bench Verified (76.0 vs 77.2), TerminalBench 2.1, and OSWorld-Verified. Apache 2.0, ~18 GB Q4. **Evaluated, not promoted — see §3.1.** |
| **NVIDIA Nemotron 3.5 Lightning** | NVIDIA (hybrid Mamba+MoE, 30B total/3B active) | Aug 11, 2026 | Unchanged status — new independent data point this cycle (Artificial Analysis Intelligence Index 24, on par with gpt-oss-120b; Elo 824 on agentic benchmarks) but SWE-bench Verified still only 52.80%, well below every code-implementer pick. Ollama's `q4_K_M` tag lists 25 GB for a nominally-4-bit 30B model (~6.7 bpw implied) — arithmetic flag, not fully explained. **Not promoted to any role.** |
| **DeepSeek V4-Flash / V4 Pro** | DeepSeek (MoE 284B/13B active and 1.6T/49B active respectively) | Jul 31, 2026 (Flash); V4 Pro went GA 2026-08-12 | Both confirmed cloud-only **by Ollama's own tag listing** (`:cloud`-suffixed tags only, tag-level check as of 2026-08-15, not independently re-fetched this cycle), not just by size arithmetic — see §1. |
| **Laguna XS 2.1** | Poolside (MoE 33B/3B active) | Jul 2, 2026 | Remains a fully valid alternative to the current Ornith-1.5 code-implementer picks — longer field track record and its own directly-measured M5 Max throughput (87–107 tok/s, oMLX). Not deprecated. |
| **Kimi K3** | Moonshot AI (MoE 2.8T/50.4B active) | Jul 16, 2026 (announced); open weights Jul 26, 2026 | Still no local GGUF/Ollama/MLX support (`kimi-k3:cloud` only); llama.cpp PR #26185 remains unmerged (re-checked this cycle). **Cloud-only — see §1.** |
| **Mistral Medium 3.5** | Mistral AI (dense 128B) | Apr 29, 2026 | 77.6% SWE-bench Verified; ~80 GB Q4; oMLX data (7.1–7.2 tok/s) independently confirms the bandwidth-ceiling tok/s estimate is accurate, not just theoretical. Still too slow for agentic loops — quality-critical one-shot use only. |
| **Ollama v0.32.14** | Platform release | ~Aug 15, 2026 | Current stable, up from v0.32.9. Notable changes since v0.32.9: `repeat_penalty` now defaults to 1.0 (off) instead of 1.1 for models that don't set it, matching other engines and speeding up speculative decoding; ~7–8% faster NVFP4 MLX prefill on Qwen3.6/Muse Glimmer; Linux/Windows MLX support coming online. No v0.33 release found as of 2026-08-21. |

### Notable models evaluated and excluded (checked repeatedly, still excluded)

| Model | Family | Released | Why noted |
|-------|--------|----------|-----------|
| **GLM-5.2** | Z.ai (MoE 744B/40B active) | Jun 13, 2026 | Doesn't fit 128 GB; `:cloud` only. A "GLM-5.5" successor is analyst speculation, not an official release. |
| **MiniMax M3** | MiniMax (MoE ~428B/23B active) | Jun 1, 2026 | Doesn't fit; `:cloud` only. Newer "H3" is multimodal/video, not a text LLM, and unshipped as of the latest source checked. |
| **NVIDIA Nemotron 3 Super** | NVIDIA (hybrid Mamba-Transformer, 120B/12B active) | Mar 11, 2026 | ~60 GB at Q4, but 60.47% SWE-bench Verified, below all current picks. Not selected. |

### Pending Releases Worth Re-Checking Next Cycle

**Purpose of this list:** models that are (a) publicly promised or previewed
but not yet locally usable, and (b) would actually fit the 128 GB envelope
and plausibly challenge a current pick *if* they ship — as opposed to the
much longer list of promised-but-still-oversized models (Kimi K3, GLM-5.5,
MiniMax H3, etc.) that wouldn't change a verdict even once released, and so
aren't worth a standing re-check. Every item here should be explicitly
re-verified at the start of the next refresh (see `COMMIT_FORMAT.md`'s
"Step 0.5") — added when first spotted, removed once it ships or is
confirmed dead, never left to silently go stale in old prose.

| Model | Why it's on this list | Last checked |
|-------|------------------------|---------------|
| *(none)* | **Qwen3.8-27B shipped 2026-08-14** and was evaluated this cycle as a normal candidate against Qwen3.6-27B — not promoted (see Recent Changes and §3.1). No new promised-but-unshipped, would-actually-fit, would-actually-matter model was identified this cycle. Table intentionally empty; will be re-populated the moment a genuine candidate is spotted. | 2026-08-21 |

### SWE-bench Verified Snapshot (Aug 21, 2026)

For context on where role picks sit in the broader leaderboard. Frontier
(non-open-weight, API-only) scores remain omitted from precise citation —
trackers still disagree on exact figures for the current frontier and none of it
changes a local pick. **Qwen3.8-27B is intentionally absent** — Alibaba has not
published a SWE-bench Verified score for it (only SWE-bench Pro, a different
benchmark), so it cannot be placed on this table without conflating the two — see
Recent Changes and §3.1.

| Model | Score | Locally runnable? |
|-------|-------|--------------------|
| Kimi K3 | 93.4% (vals.ai) | No (`:cloud` only, 2.8T params) |
| Ornith-1.5-397B | 86.0% | No (242 GB Q4) |
| Ornith-1.0-397B | 82.4% | No (200 GB Q4) — beaten by its own 1.5 successor above |
| DeepSeek V4 Pro | 80.6% | No (`:cloud` tag only, 800 GB Q4) |
| MiniMax M3 | 80.5% | No (143 GB min) |
| **Ornith-1.5-35B-A3B** | **79.0%** | **Yes (~23 GB Q4) — current top local coding pick, changed 2026-08-21** |
| Mistral Medium 3.5 | 77.6% | **Yes (80 GB Q4 — slow, ~7 tok/s)** |
| Qwen3.6-27B | 77.2% | **Yes (~17 GB Q4)** |
| **Muse Glimmer** | 76.0% | Yes (~18 GB Q4) — **evaluated, not promoted, see §3.1** |
| Ornith-1.0-35B | 75.6% | Yes (~21 GB Q4) — beaten as top pick, still a valid alternative |
| Laguna XS 2.1 | 70.9% | Yes (~20 GB Q4) — still a valid alternative to the Ornith-1.5 picks |
| **Ornith-1.5-9B** | **70.6%** | **Yes (~6.6 GB Q4) — current smaller coding pick, changed 2026-08-21** |
| Ornith-1.0-9B | 69.4% | Yes (~5.6 GB Q4) — beaten as smaller pick, still a valid alternative |
| Devstral Small 2 | 68.0% | Yes (~15 GB Q4) — still a valid alternative |
| Cohere North Mini Code 1.0 | 67.6% | Yes (~17 GB Q4) |
| NVIDIA Nemotron 3.5 Lightning | 52.80% | Yes (~25 GB) — too low-scoring for any current pick |

Source: [vals.ai SWE-bench leaderboard](https://www.vals.ai/benchmarks/swebench) and [BenchLM.ai](https://benchlm.ai/benchmarks/sweVerified) (both primary-adjacent/secondary, cross-checked); Ornith-1.5-35B-A3B, Ornith-1.5-9B, Ornith-1.5-397B scores verified directly against Hugging Face model cards (primary) this cycle — no third-party reproduction yet, see Recent Changes.

---

## 3. Recommendations by Role

### 3.1 Generalist Agentic Default
*One model that does most jobs adequately.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Qwen3.6-27B | ~17 GB | SWE-bench Verified 77.2%; Terminal-Bench 2.0 59.3%; 256K ctx | `qwen3.6:27b` |
| **Smaller pick** | Gemma 4 12B Unified | ~7.6 GB | MMLU Pro 77.2%; natively multimodal (text + image) | `gemma4:12b` |

**Rationale:** Qwen3.6-27B (Alibaba, Apr 22, 2026) is a 27B dense model with a built-in dual-mode thinking toggle: `/think` on for deep reasoning, off for fast chat. At 77.2% SWE-bench Verified it sits well within range of the frontier API models while running entirely locally. The 256K context window handles most agentic sessions without truncation. Gemma 4 12B Unified is the right answer when memory is at a premium or multimodal input is needed at low cost.

**Evaluated, not selected (2026-08-21): Qwen3.8-27B.** Alibaba's next-generation model in the same size class shipped 2026-08-14 (`qwen3.8:27b`, ~18 GB Q4, Apache 2.0). It's the same 27B dense/hybrid-attention architecture as Qwen3.6-27B — the upgrade is entirely post-training (stronger RL on agentic tasks), not a new base model — and posts real gains per Alibaba's own model card: Terminal-Bench 2.1 63.4→73.0, OSWorld-Verified 63.9→84.3, SWE-bench Pro 53.5→61.7. Two things keep it from displacing Qwen3.6-27B this cycle: **(1)** Alibaba has not published a SWE-bench Verified score for it at all, so there's no apples-to-apples read against this doc's tracked headline metric; **(2)** independent field reports describe the model **defaulting to `reasoning_effort=xhigh`**, with one developer clocking 22,276 reasoning tokens and 21 minutes of latency on a single trivial SVG-generation prompt — a serious throughput regression for the many-completions-per-session agentic-loop use case this doc optimizes for (this doc's own selection criterion: throughput is part of quality). Community reports also note chat-template bugs and higher context-memory use than 3.6. Worth re-testing at `reasoning_effort=low`/`medium` once field data accumulates and template fixes land — not a rejection of the model, a "not yet" given what's confirmable this cycle. See §2.

**Evaluated, not selected:** *Muse Glimmer* (Meta Superintelligence Labs, dense 30B, released Aug 10, 2026, Apache 2.0, `muse-glimmer:30b`, ~18 GB Q4) is a genuine new contender purpose-built for agentic workloads. Meta's own benchmark table shows a **mixed** result against Qwen3.6-27B: Glimmer wins decisively on tool-use/agentic benchmarks (MCP-Atlas 75.5 vs 62.5, DeepSearch QA 74.6 vs 71.1, GAIA2 43.3 vs 40.0, WildClawBench 47.6 vs 43.2) but loses on this doc's headline benchmark, SWE-bench Verified (76.0 vs 77.2), and on TerminalBench 2.1 (51.7 vs 60.7) and OSWorld-Verified (65.9 vs 75.6). The secondary source reproducing these numbers (kingy.ai) explicitly flags a methodology concern — Meta "selected the more favorable of a competitor's self-reported score or its own reproduction" — which is reason for caution on the comparison numbers themselves, separate from the mixed result. No new data surfaced this cycle (re-checked 2026-08-21); still a **"watch next cycle" candidate, not a promotion**. If MCP-style tool-calling and general agentic reliability matter more to a given workflow than raw SWE-bench score, Muse Glimmer is worth trying now despite not being the default pick here.

**Sizing note:** ~16.8 GB at Q4_K_M; ~19.5 GB at Q5_K_M; ~28.6 GB at Q8_0 for Qwen3.6-27B. Q5_K_M is recommended if the extra 3 GB is available.

### 3.2 Code Implementer
*Writes code, multi-file edits, agentic coding loops.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | **Ornith-1.5-35B-A3B** | ~23 GB | SWE-bench Verified 79.0%; Terminal-Bench 2.1 68.5%; 256K–1M ctx (YaRN) | `ornith-1.5:35b` |
| **Smaller pick** | Ornith-1.5-9B | ~6.6 GB | SWE-bench Verified 70.6%; Terminal-Bench 2.1 46.2–47.0; 256K–1M ctx (YaRN) | `ornith-1.5:9b` |

**Rationale — both picks, as of 2026-08-21:** Ornith-1.5-35B-A3B and Ornith-1.5-9B (ornith-ai/deepreinforce-ai, shipped 2026-08-19/20) extend the Ornith-1.0 family's self-scaffolding approach into a full self-improvement loop — the model now generates its own training tasks in addition to its own scaffolds and rollouts, MIT license throughout. Both scores are confirmed directly against Ornith's own Hugging Face model cards (primary) using the **same OpenHands-harness methodology** (temp=1.0, top_p=0.95, 256K context, averaged over 5 runs, anti-reward-hacking controls: git history stripped, network disabled) that this doc already trusted for the 1.0 generation — a genuine apples-to-apples comparison, not a benchmark-suite change. The 35B variant gains 3.4 points over Ornith-1.0-35B (79.0% vs 75.6%); the 9B variant gains 1.2 points over Ornith-1.0-9B (70.6% vs 69.4%). Both Ollama tags were fetched directly and confirmed: `ornith-1.5:35b` at 23 GB, `ornith-1.5:9b` at 6.6 GB — a ~10–18% size increase over the 1.0 generation that passes the arithmetic gate (still consistent with a Q4-class quant at these parameter counts) and is plausibly explained by added multimodal/vision input support. **Two caveats stated plainly, per this doc's epistemic rules:** these are **self-reported scores with no independent third-party reproduction yet** (the same position Ornith-1.0's scores were in for one cycle before BenchLM.ai independently matched them), and **no fresh M5 Max/oMLX throughput measurement exists** for the 1.5 family — it shipped 1–2 days before this refresh. §1's tok/s table extrapolates by architecture analogy to the 1.0 family (same MoE-A3B/dense pattern, ~5–8% slower estimated for the larger footprint), explicitly marked as extrapolated, not measured. Given the clean win on the exact headline metric via a primary source, from an already-vetted family with an unbroken evaluation methodology, promotion is made on that basis while flagging both gaps for next cycle's re-check.

**Also evaluated, not selected:**
- *Ornith-1.0-35B* and *Ornith-1.0-9B* (deepreinforce-ai; 75.6% and 69.4% SWE-bench Verified; `ornith:35b`/`ornith:9b`, ~21 GB/~5.6 GB, MIT): **beaten by their own 1.5 successors** (see above), not deprecated. Both remain fully valid alternatives with real field track records (5+ and 6+ weeks of production use respectively) and directly-measured M5 Max throughput (77–123 tok/s and 109 tok/s respectively, oMLX) that the 1.5 generation doesn't have yet. Teams wanting a benchmarked-in-production model today, not a 2-day-old release, have no urgent reason to switch.
- *Laguna XS 2.1* (Poolside, Jul 2, 2026; MoE 33B/3B active; 70.9% SWE-bench Verified; `laguna-xs-2.1:q4_K_M`, ~20 GB, Apache-compatible): still a fully valid alternative — longer field track record than either Ornith generation, and its own very competitive measured M5 Max throughput. Teams already standardized on Laguna's tool-calling behavior have no urgent reason to switch.
- *Muse Glimmer* (Meta, dense 30B, Aug 10, 2026; SWE-bench Verified 76.0%; SWE-bench Pro 51.2 vs Qwen3.6-27B's 50.2 — narrowly ahead there but behind on Verified, TerminalBench, and OSWorld): re-checked this cycle, no new data. Doesn't clearly beat the current picks on the headline metric. See §3.1 for the full comparison.
- *NVIDIA Nemotron 3.5 Lightning* (30B-A3B MoE, released Aug 11, 2026): 52.80% SWE-bench Verified — well below every pick and alternative in this section, unchanged this cycle. A new independent data point (Artificial Analysis Intelligence Index 24, Elo 824 on agentic benchmarks) doesn't move the coding-quality verdict. Fast but not competitive on quality for this role.
- *Devstral Small 2* (Mistral, 24B dense; 68.0% SWE-bench; `devstral-small-2`, ~15 GB Q4, Apache 2.0): still a valid alternative to the Ornith-1.5-9B smaller pick, especially where Devstral's longer production track record in agentic tool-calling scaffolds (OpenHands, SWE-agent) matters more than footprint.
- *Mistral Medium 3.5* (128B dense; 77.6% SWE-bench Verified; `mistral-medium-3.5:128b`, 80 GB Q4): fits, but at ≈7.1–7.2 tok/s (confirmed by direct oMLX measurement, not just bandwidth arithmetic) a 1K-token completion takes roughly 140–150 seconds versus Ornith-1.5-35B-A3B's estimated ~8–14 second range — the latency penalty rules it out for agentic loops. Quality-critical one-shot tasks only.
- *Ornith-1.5-397B* (86.0%), *DeepSeek V4 Pro* (80.6%), *MiniMax M3* (80.5%), *Kimi K3* (93.4% per vals.ai) top the global leaderboard but none fit 128 GB at practical quants — see §1.

### 3.3 Code Debugger
*Reasoning / chain-of-thought, root-cause analysis, math-heavy debugging.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | DeepSeek-R1-Distill-Qwen-32B | ~20 GB | MMLU 83%; MATH 72% (highest of any sub-40 GB local model) | `deepseek-r1:32b` |
| **Smaller pick** | Qwen3.6-27B (thinking mode on) | ~17 GB | SWE-bench 77.2%; switchable CoT; same model as §3.1 | `qwen3.6:27b` |

**Rationale:** DeepSeek-R1-Distill-Qwen-32B provides always-on chain-of-thought reasoning and holds the highest MATH benchmark score of any locally-runnable model under 40 GB. Distilled from the 671B R1 teacher; generates full reasoning traces before answering. oMLX M5 Max data (2026-07-15, 4bit) directly measures 27.6–28.9 tok/s at short context — consistent with this doc's Ollama-MLX estimate. Qwen3.6-27B with `/think` is the pragmatic alternative if you want to avoid loading a second model. No challenger has beaten this pick.

### 3.4 Plan Orchestrator
*Long context + reliable tool calling + structured output for multi-agent coordination.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Llama 4 Scout | ~67 GB | 10M-token context; 17B active / 109B total MoE; natively multimodal; strong tool calling | `llama4:scout` |
| **Smaller pick** | Qwen3.6-27B | ~17 GB | 256K context; reliable structured output; strong tool fidelity | `qwen3.6:27b` |

**Rationale:** Llama 4 Scout (Meta, Apr 2026) still offers the longest locally-runnable context window at 10M tokens by a wide margin. Its 17B active-parameter MoE runs at ~22–26 tok/s via Ollama's MLX engine on M5 Max — no fresh M5 Max data point exists for Scout as of the most recent cycle (a gap in the community benchmark sources checked, not a regression). At ~67 GB Q4 it still fits alongside a 17 GB model with room to spare. Qwen3.6-27B covers 256K context at a quarter the size.

**New candidate considered, not selected:** NVIDIA Nemotron 3.5 Lightning (30B-A3B MoE, released 2026-08-11, 1M-token context on its standard variant, claimed high throughput for "always-on agent execution layers") is architecturally interesting for this role — long context plus speed is exactly the orchestrator profile — but no tool-calling-specific benchmark exists for it yet, and its SWE-bench Verified score (52.80%) suggests weaker code-comprehension than Scout or Qwen3.6-27B for orchestration tasks that involve reasoning about code. Worth re-evaluating once field-tested; not promoted given how recently it shipped.

**Checked this cycle:** Meta shipped **Llama 5** (~600B total MoE, 5M-token context, native video/audio) but it exceeds 128 GB by nearly 3× (≈338 GB at Q4) with no smaller "Scout"-class sibling — not a candidate for this hardware. Llama 4 Scout remains Meta's only locally-viable model. See §1/§2.

**MCP tool-calling context:** No update to the MCP-Mark leaderboard (last dated Jun 2026: Kimi K2.7 Code leads at 0.811) has been found. Neither Scout nor Qwen3.6-27B appear in that evaluation.

**Also evaluated, not selected:** NVIDIA Nemotron 3 Super (Mar 11, 2026; 120B/12B-active, 1M context, ~60 GB at Q4) — scores 60.47% SWE-bench Verified, below Qwen3.6-27B and Scout's orchestration-specific strengths.

### 3.5 LLM-as-Judge / Verifier
*Calibrated scoring against rubrics, pairwise preference evaluation, output verification.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Qwen3.6-27B | ~17 GB | Strong instruction following; consistent in non-thinking mode; 256K ctx | `qwen3.6:27b` |
| **Smaller pick** | Gemma 4 12B Unified | ~7.6 GB | Fast; MMLU Pro 77.2%; low latency for high-volume scoring | `gemma4:12b` |

**Rationale:** For judging, **disable thinking mode** — reasoning traces inflate token cost and can introduce self-consistency bias in scores. Qwen3.6-27B's instruction fidelity and calibration make it the recommended local judge. Inject a scored reference example at the top of the judge prompt to anchor the scoring scale. **Staleness flag, unchanged:** no fresher RewardBench 2 data has been found — last confirmed data remains the Jun 2025 submission behind RewardBench 2's ICLR 2026 paper. For high-stakes judging, frontier API models remain preferred; use local judges for cost-sensitive or privacy-constrained pipelines.

**Evaluated, not selected (2026-08-21):** Qwen3.8-27B (see §3.1 for the full writeup) is **especially unsuited to this role right now**, not just "not yet promoted" — its reported default `reasoning_effort=xhigh` behavior (22K+ reasoning tokens on trivial prompts in field reports) is directly at odds with high-volume, low-latency scoring, which is exactly what judging workloads need. Worth revisiting once `reasoning_effort=low` behavior is field-validated.

### 3.6 Document Understanding / Interpreter
*PDFs, OCR, table and equation extraction.*

| | Model | Q4 resident | Key capability | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Gemma 4 26B MoE (E26B-A4B) | ~18 GB | Natively multimodal; strong table/equation/diagram extraction; 89% AIME 2026 | `gemma4:26b` |
| **Smaller pick** | Mistral OCR 4 | *(container, not GGUF)* | Purpose-built OCR; structured Markdown/JSON output; 170 languages, bounding-box/block classification | *Not Ollama-loadable — see below* |

**Rationale:** Gemma 4's 26B MoE variant (Google, Apr 2026) understands images, text, tables, and LaTeX natively in a single forward pass. It handles complex PDF layouts, nested tables, and mixed-language documents reliably. No update has moved this pick. Mistral OCR 4 remains a real, purpose-tuned upgrade for dedicated OCR pipelines run *outside* Ollama (still no evidence of a GGUF/Ollama-loadable form); within the Ollama-only constraint of this document, Gemma 4 12B Unified (§3.1) remains the fallback for lighter-weight document tasks.

### 3.7 Vision / Image Understanding
*VLMs for image comprehension only — not generation.*

| | Model | Q4 resident | Key capability | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Llama 4 Scout | ~67 GB | Natively integrated vision; coherent multi-image + long-text reasoning | `llama4:scout` |
| **Smaller pick** | Gemma 4 E4B | ~9.6 GB | Natively multimodal (image, audio, video); best small VLM available | `gemma4:e4b` |

**Rationale:** Llama 4 Scout integrates vision natively, resulting in more coherent multi-modal reasoning when combining images with long context. Supports up to 5 input images per prompt. Gemma 4 E4B is the go-to when you need a capable VLM in a small footprint. **Under consideration:** Muse Glimmer (§3.1) ships with its own dedicated 1.8B perception/vision encoder and is natively multimodal, but no vision-specific benchmark (image QA, multi-image reasoning) exists for it yet — not evaluated as a vision-role candidate yet given the same "too new, unvalidated" caveat as §3.1. Cloud-only alternatives (Qwen3-VL-235B, InternVL3-78B) lead open-weight VLM benchmarks globally but don't fit locally.

---

## 4. The "Sufficient but Smaller" Angle

Four patterns consistently pay off on M5 Max 128 GB:

### 4.1 MoE-A3B Models
**Ornith-1.5-35B-A3B** (top code-implementer pick, 35B/~3B active, ~23 GB at Q4_K_M), **Laguna XS 2.1** (33B/3B active, ~20 GB), and **Qwen 3.5 30B-A3B** (30B/3B active, ~17 GB) all activate only ~3B parameters per token despite loading a much larger weight set. Directly-measured oMLX M5 Max data confirms the pattern holds up for the still-current-generation members: 77–123 tok/s for Ornith-1.0-35B, 87–107 tok/s for Laguna XS 2.1 — both firmly in "interactive agentic loop" territory despite 30B+ class quality. Ornith-1.5-35B-A3B's own throughput is extrapolated by analogy (no fresh measurement yet — see §1) but expected to hold the same pattern given its near-identical architecture. This remains the highest quality-per-GB pattern in the current Ollama library.

### 4.2 Distilled Reasoning Models
**DeepSeek-R1-Distill-Qwen-32B** transfers chain-of-thought behaviour from a 671B teacher into a 32B student (20 GB at Q4). It achieves MATH scores that beat many naive 70B models, at twice the speed and half the memory. For debugging and reasoning tasks, the distill is sufficient — you don't need to reach for a 70B.

### 4.3 Tiny Purpose-Trained Coding Models
**Ornith-1.5-9B** is a *dense* 9B model, trained specifically around self-generated agentic-RL scaffolds (now extended to a full self-improvement loop as of the 1.5 generation), that beats larger competitors at a fraction of their resident memory. Its 35B sibling's continued lead as top pick across two generations (1.0→1.5) suggests the self-scaffolding/self-improvement training approach scales cleanly across sizes within the same family, not just as a one-off small-model trick.

### 4.4 Purpose-Built Specialized Small Models
A handful-of-GB specialist routinely beats a 70B generalist on its target domain:
- **Mistral OCR 4**: purpose-tuned OCR with structured output — runs outside Ollama (container/API, not GGUF — see §3.6)
- **Gemma 4 E4B** (~9.6 GB): best small VLM; handles image/audio/video natively
- **nomic-embed-text**: best-in-class embeddings for RAG pipelines; negligible resident memory

The 128 GB envelope lets you keep several specialists resident simultaneously.

---

## 5. Suggested Concrete Stack

Ollama loads and unloads models from unified memory on demand. The full inventory fits on disk; concurrent-pair examples show what stays live together without eviction.

### Full Inventory (Disk)

| Model | Ollama tag | Q4 resident | Primary role |
|-------|-----------|------------|---------------|
| Llama 4 Scout | `llama4:scout` | ~67 GB | Orchestrator + vision |
| Qwen3.6-27B | `qwen3.6:27b` | ~17 GB | Generalist + judge |
| DeepSeek-R1-Distill-Qwen-32B | `deepseek-r1:32b` | ~20 GB | Code debugger / reasoning |
| **Ornith-1.5-35B-A3B** | `ornith-1.5:35b` | ~23 GB | Code implementer (top pick, higher SWE-bench) |
| Ornith-1.5-9B | `ornith-1.5:9b` | ~6.6 GB | Code implementer (smaller pick) |
| Gemma 4 26B MoE | `gemma4:26b` | ~18 GB | Document understanding |
| Gemma 4 12B Unified | `gemma4:12b` | ~7.6 GB | Fast general + doc assistant |
| Gemma 4 E4B | `gemma4:e4b` | ~9.6 GB | Small vision / audio / video |
| *Laguna XS 2.1 (optional, alternative)* | `laguna-xs-2.1:q4_K_M` | ~20 GB | *Code implementer alt — longer field track record, very competitive speed* |
| *Ornith-1.0-35B / -9B (optional, alternative)* | `ornith:35b` / `ornith:9b` | ~21 GB / ~5.6 GB | *Code implementer alt — real field track record, directly-measured throughput, vs. the 1.5 picks' 2-day-old self-reported scores* |
| *Devstral Small 2 (optional)* | `devstral-small-2` | ~15 GB | *Code implementer alt — battle-tested tool-calling track record* |
| *Mistral Medium 3.5 (optional)* | `mistral-medium-3.5:128b` | ~80 GB | *Quality-critical one-shot coding, latency-tolerant only* |

**Total core inventory size on disk (excluding optionals):** ~169.2 GB. Laguna XS
2.1 adds ~20 GB, the Ornith-1.0 pair adds ~26.6 GB, Devstral Small 2 adds ~15 GB,
Mistral Medium 3.5 adds ~80 GB if all optionals are also pulled. Ollama evicts from
RAM on demand.

### Recommended Concurrent Working Sets (each under ~100 GB)

| Set | Combined RAM | Use case |
|------|-------------|----------|
| Scout + Qwen3.6 | ~84 GB | Orchestrator + workhorse; long-context agentic sessions |
| Ornith-1.5-35B-A3B + Ornith-1.5-9B | ~29.6 GB | Same-family coding pair: escalate from smaller to top pick within one session without a family-behavior mismatch |
| Ornith-1.5-9B + DeepSeek-R1 32B | ~26.6 GB | Coding session: implement → debug loop |
| Qwen3.6 + Gemma 4 12B | ~24.6 GB | Lightweight dual-model chat + vision |
| Scout + Ornith-1.5-35B-A3B | ~90 GB | Orchestrated multi-file coding with the top code pick |
| Mistral Medium 3.5 + Qwen3.6 | ~97 GB | Quality-first coding + fast judge — fits but with limited KV-cache headroom |

Laguna XS 2.1 and the Ornith-1.0 pair both remain drop-in substitutes for the
Ornith-1.5 picks in any pairing above (same ~20-23 GB class) if a longer field
track record is preferred over the newest benchmark score.

### What to Exclude from Local Use

| Model | Reason |
|-------|--------|
| Kimi K3 (2.8T total) | `:cloud` only; ~700+ GB even at 2-bit |
| DeepSeek V4-Flash / V4 Pro (284B / 1.6T total) | `:cloud` tags only on Ollama — no local variant exists |
| Ornith-1.5-397B (397B total) | Q4 ≈ 242 GB (OOM); sub-Q2 quants might fit numerically but are untested |
| GLM-5.3 (743B total) | Weights not yet public; ≈418 GB at Q4 even once released — `:cloud` only |
| Llama 5 (~600B total) | ≈338 GB at Q4 — no smaller "Scout"-class sibling exists |
| Llama 4 Maverick (400B total) | Q4 ≈ 200 GB — hard OOM |
| Kimi K2.7 Code (1T total) | Community GGUF exists but ~585 GB at Q4 |

---

## 6. Verification Checklist

```bash
# Ensure Ollama is up-to-date (v0.32.14 as of 2026-08-15; no v0.33 as of 2026-08-21)
ollama version

# --- Pull core models ---
ollama pull qwen3.6:27b              # ~17 GB  — generalist / judge / debugger fallback
ollama pull llama4:scout              # ~67 GB  — orchestrator + vision
ollama pull deepseek-r1:32b           # ~20 GB  — reasoning / debugging
ollama pull ornith-1.5:35b            # ~23 GB  — code implementation (top pick)
ollama pull ornith-1.5:9b             # ~6.6 GB — code implementation (smaller pick)
ollama pull gemma4:26b                # ~18 GB  — document understanding
ollama pull gemma4:12b                # ~7.6 GB — fast general
ollama pull gemma4:e4b                # ~9.6 GB — small vision/audio/video

# --- Optional alternatives ---
# ollama pull laguna-xs-2.1:q4_K_M      # ~20 GB  — code implementation alt (longer track record)
# ollama pull ornith:35b                # ~21 GB  — code implementation alt (Ornith-1.0, beaten as top pick, real field track record)
# ollama pull ornith:9b                 # ~5.6 GB — code implementation alt (Ornith-1.0, beaten as smaller pick, real field track record)
# ollama pull devstral-small-2          # ~15 GB  — code implementation alt (battle-tested tool-calling)
# ollama pull mistral-medium-3.5:128b   # ~80 GB  — 77.6% SWE-bench but ~7 tok/s (confirmed via direct M5 Max measurement)
# ollama pull qwen3.8:27b               # ~18 GB  — evaluated, NOT promoted this cycle (overthinking/latency regression at default reasoning_effort — see §3.1); pull only to test reasoning_effort=low/medium yourself

# --- Verify resident memory after load ---
# (run `ollama ps` while the model is active)
# qwen3.6:27b           → 16–18 GB
# llama4:scout          → 65–70 GB
# deepseek-r1:32b       → 19–21 GB
# ornith-1.5:35b        → 22–24 GB
# ornith-1.5:9b         → 6–7 GB
# gemma4:26b            → 17–19 GB
# gemma4:12b            → 7–8 GB
# gemma4:e4b            → 9–10 GB
# laguna-xs-2.1:q4_K_M  → 18–21 GB
# ornith:35b            → 20–22 GB
# ornith:9b             → 5–6 GB
# devstral-small-2      → 14–16 GB
# mistral-medium-3.5:128b → 78–82 GB
# qwen3.8:27b           → 17–19 GB

# --- Smoke tests ---

# Generalist (Qwen3.6-27B)
ollama run qwen3.6:27b "Write a Python function to merge two sorted lists and return the merged result."

# Thinking mode on (code debugger role)
ollama run qwen3.6:27b "/think Identify the bug in this Python snippet: def fib(n): return fib(n-1) + fib(n-2)"
# Expected: <think>...</think> block noting missing base cases

# Long context orchestration (Llama 4 Scout)
ollama run llama4:scout "List 5 subtasks for building a REST API with auth, rate-limiting, and tests."

# Reasoning chain (DeepSeek-R1)
ollama run deepseek-r1:32b "What is the derivative of x^3 * sin(x)? Show all steps."
# Expected: full chain-of-thought before answer

# Agentic coding, top pick (Ornith-1.5-35B-A3B)
ollama run ornith-1.5:35b "Refactor this function to handle None inputs gracefully: def get_len(s): return len(s)"

# Agentic coding, smaller pick (Ornith-1.5-9B)
ollama run ornith-1.5:9b "Implement a Python function that validates an email address using a regex, with unit tests."

# Vision (Gemma 4 E4B) — requires an image file
# ollama run gemma4:e4b "Describe what you see in this image." --image /path/to/screenshot.png

# Document understanding (Gemma 4 26B MoE)
ollama run gemma4:26b "Extract all column headers and row values from this table image." --image /path/to/table.png
```

---

## 7. Sources

### Platform & Release Notes
- [Ollama Releases — GitHub](https://github.com/ollama/ollama/releases) (primary — v0.32.6 through v0.32.14 confirmed)
- [Ollama Library](https://ollama.com/library)
- [Releasebot — Ollama Release Notes](https://releasebot.io/updates/ollama) (secondary — v0.32.14 changelog detail, repeat_penalty default change, NVFP4 MLX prefill speedup)

### Ollama Library Pages Fetched Directly (primary)
- [ornith / tags](https://ollama.com/library/ornith/tags) — confirmed `ornith:35b` 21 GB, `ornith:9b` 5.6 GB
- [ornith (model page)](https://ollama.com/library/ornith)
- [ornith-1.5 / tags](https://ollama.com/library/ornith-1.5/tags) — **fetched 2026-08-21**, confirmed `ornith-1.5:35b` 23 GB, `ornith-1.5:9b` 6.6 GB, `ornith-1.5:397b` 242 GB, all 256K ctx
- [qwen3.6 / tags](https://ollama.com/library/qwen3.6/tags) — confirmed `qwen3.6:27b` 17 GB
- [qwen3.8 / tags](https://ollama.com/library/qwen3.8/tags) — **fetched 2026-08-21**, confirmed `qwen3.8:27b`/`q4_K_M` 18 GB, `27b-mlx` 18 GB, `27b-bf16` 56 GB, 256K ctx, 11 total tag variants
- [laguna-xs-2.1 / tags](https://ollama.com/library/laguna-xs-2.1/tags) — confirmed `q4_K_M` 20 GB
- [llama4 / tags](https://ollama.com/library/llama4/tags) — confirmed `scout` 67 GB
- [gemma4 / tags](https://ollama.com/library/gemma4/tags) — confirmed 26b 18 GB, 12b 7.6 GB, e4b 9.6 GB
- [muse-glimmer / tags](https://ollama.com/library/muse-glimmer/tags) — confirmed `30b`/`q4_K_M` 18 GB
- [nemotron-3.5-lightning / tags](https://ollama.com/library/nemotron-3.5-lightning/tags) — confirmed `q4_K_M` 25 GB (arithmetic flag noted, §2)
- [kimi-k3](https://ollama.com/library/kimi-k3) — confirmed cloud-only tag

### Apple Silicon Benchmarks
- [oMLX Community M5 Max Benchmarks](https://omlx.ai/benchmarks) (primary/crowdsourced — individually-dated, directly-measured M5 Max figures for Ornith-1.0-9B/35B, Laguna XS 2.1, DeepSeek-R1-Distill-32B, Llama 3.1 8B, Llama 3.3 70B, Mistral Medium 3.5)
- [Apple Silicon LLM Benchmarks 2026 — LLMCheck](https://llmcheck.net/benchmarks)
- [PromptQuorum M5 Max Benchmarks](https://www.promptquorum.com/local-llms/m5-pro-max-llm-benchmarks-2026)
- [Presenc AI Local Benchmarks 2026](https://presenc.ai/research/local-llm-tokens-per-second-benchmarks-2026)
- [Contra Collective — KV Cache Quantization on Apple Silicon (M5 Max)](https://contracollective.com/blog/kv-cache-quantization-q8-vs-q4-m5-max-mlx-2026)
- [Apple ML Research — Exploring LLMs with MLX on M5](https://machinelearning.apple.com/research/exploring-llms-mlx-m5) (primary, base M5 only, not M5 Max)
- *Wale Akinfaderin, "Benchmarking Open-Weights LLMs on the MacBook Pro M5 Max" (Medium) — fetchable but paywalled beyond the intro; confirmed intro benchmarks the binned 32-core/460 GB/s M5 Max SKU, not this doc's 40-core/614 GB/s target — low value even if fully accessible.*

### Ornith-1.5 (current top/smaller code-implementer picks)
- [Ornith-1.5: From Self-Scaffolding to Self-Improvement — Ornith Blog](https://ornith.ai/ornith_1_5.html) (primary — methodology, self-improvement loop description)
- [ornith-ai/Ornith-1.5-35B-A3B — Hugging Face](https://huggingface.co/ornith-ai/Ornith-1.5-35B-A3B) (primary — **fetched directly 2026-08-21**: SWE-bench Verified 79.0%, 35B/~3B active MoE, MIT, 262K ctx extensible to ~1M via YaRN)
- [ornith-ai/Ornith-1.5-9B — Hugging Face](https://huggingface.co/ornith-ai/Ornith-1.5-9B) (primary — **fetched directly 2026-08-21**: SWE-bench Verified 70.6%, Terminal-Bench 2.1 46.2–47.0, 9B, MIT, 262K ctx)
- [Ornith-1.5 35B-A3B Benchmarks: How It Stacks Up Against Qwen3.6 — MindStudio](https://www.mindstudio.ai/blog/ornith-1-5-35b-a3b-benchmarks) (secondary)
- [Ornith-1.5 open models launch in 397B, 35B, and 9B sizes — TestingCatalog](https://www.testingcatalog.com/ornith-1-5-open-models-launch-in-397b-35b-and-9-b-sizes/) (secondary — confirms self-reported-only, no third-party reproduction yet)
- [DeepReinforce Releases Open-Source Ornith 1.5 Family — officechai](https://officechai.com/ai/deepreinforce-releases-open-source-orinth-1-5-family-of-models-with-solid-benchmarks-and-mit-license/) (secondary)

### Ornith-1.0 (beaten this cycle, still valid alternatives)
- [Ornith-1.0: Self-Scaffolding LLMs for Agentic Coding — DeepReinforce/Ornith Blog](https://ornith.ai/ornith_1_0.html) (primary — 75.6% SWE-bench Verified for 35B, methodology stated)
- [deepreinforce-ai/Ornith-1.0-35B — Hugging Face](https://huggingface.co/deepreinforce-ai/Ornith-1.0-35B) (primary — architecture and score confirmed)
- [deepreinforce-ai/Ornith-1.0-35B-GGUF — Hugging Face](https://huggingface.co/deepreinforce-ai/Ornith-1.0-35B-GGUF) (primary)
- [Ornith 1.0 35B MoE: Faster Than 9B, Better Than 31B — Ornith.site Blog](https://www.ornith.site/blog/ornith-1-0-35b-moe/) (secondary — independently confirms MoE architecture and 75.6% score)
- [Ornith-1.0-35B Benchmarks — BenchLM.ai](https://benchlm.ai/models/ornith-1-0-35b) (secondary — independently agrees on 75.6%)
- [deepreinforce-ai/Ornith-1.0-9B — Hugging Face](https://huggingface.co/deepreinforce-ai/Ornith-1.0-9B) (primary, smaller pick)

### Muse Glimmer
- [Introducing Muse Glimmer — Meta AI Research Blog](https://research.meta.ai/blog/introducing-muse-glimmer-open-agentic-model) (primary)
- [Muse Glimmer from Meta Superintelligence Labs is now available — Ollama Blog](https://ollama.com/blog/muse-glimmer) (primary)
- [meta-models/Muse-Glimmer-30B — Hugging Face](https://huggingface.co/meta-models/Muse-Glimmer-30B) (primary — 76.0% SWE-bench Verified and full benchmark table)
- [Muse Glimmer 30B: Benchmarks, Hardware & How to Run — kingy.ai](https://kingy.ai/blog/muse-glimmer-30b-benchmarks-hardware-run/) (secondary — comparison table vs Gemma4-31B/Qwen3.6-27B, methodology caveat noted)

### NVIDIA Nemotron 3.5 Lightning
- [nvidia/NVIDIA-Nemotron-3.5-Lightning-30B-A3B-NVFP4 — Hugging Face](https://huggingface.co/nvidia/NVIDIA-Nemotron-3.5-Lightning-30B-A3B-NVFP4) (primary — 52.80% SWE-bench Verified, full benchmark table)
- [Introducing NVIDIA Nemotron 3.5 Lightning — Baseten Blog](https://www.baseten.co/blog/introducing-nemotron-35-lightning/) (secondary/partner — throughput claims)
- [NVIDIA Developer Blog — Nemotron 3.5 Lightning](https://developer.nvidia.com/blog/nvidia-nemotron-3-5-lightning-delivers-fast-accurate-specialized-task-execution-for-long-running-agents/) (primary)
- [NVIDIA launches Nemotron 3.5 Lightning — Artificial Analysis](https://artificialanalysis.ai/articles/nemotron-3-5-lightning-launch) (secondary — Intelligence Index 24, Elo 824 on agentic benchmarks; new this cycle, doesn't change the SWE-bench-based verdict — see §2/§3.2)

### DeepSeek
- [DeepSeek V4-Flash coverage — Caixin Global](https://www.caixinglobal.com/2026-08-01/deepseek-releases-official-v4-flash-model-as-chinas-ai-race-intensifies-102470292.html) (secondary)
- [DeepSeek V4 Pro Benchmarks — BenchLM.ai](https://benchlm.ai/models/deepseek-v4-pro)
- [DeepSeek V4-Flash / tags — Ollama](https://ollama.com/library/deepseek-v4-flash/tags) (primary — re-checked 2026-08-15, confirmed `:cloud`-only tags, no local variant)
- [Ollama Ships DeepSeek V4 Pro — Collabnix](https://collabnix.com/ollama-ships-deepseek-v4-pro-a-1-6t-mixture-of-experts-model-with-1m-context-and-three-reasoning-modes/) (secondary — confirms `:cloud`-only Ollama listing, added 2026-08-15 check)
- [DeepSeek V4 Pro 0813: GA release — ofox.ai](https://ofox.ai/blog/deepseek-v4-pro-0813-price-weights-benchmarks-api-access-2026/) (secondary — GA date 2026-08-12)
- [Kimi K3 llama.cpp support pre-release analysis — ggml-org/llama.cpp Discussion #26041](https://github.com/ggml-org/llama.cpp/discussions/26041) (primary — PR #26185 status, re-checked 2026-08-15, still unmerged)

### Qwen Models
- [Qwen3.6-27B Tags — Ollama](https://ollama.com/library/qwen3.6/tags) (primary)
- [Qwen/Qwen3.8-27B — Hugging Face](https://huggingface.co/Qwen/Qwen3.8-27B) (primary — model card, SWE-bench Pro 61.7%, Terminal-Bench 2.1 73.0%, OSWorld-Verified 84.3%; **no SWE-bench Verified score published** — see §3.1)
- [Qwen3.8-27B Tags — Ollama](https://ollama.com/library/qwen3.8/tags) (primary — **fetched 2026-08-21**, confirmed `27b`/`q4_K_M` 18 GB)
- [Qwen 3.8 27B vs Qwen 3.6 27B: Same Architecture, 4 Months Apart — DEV Community](https://dev.to/jamilxt/qwen-38-27b-vs-qwen-36-27b-same-architecture-4-months-apart-and-a-different-kind-of-upgrade-3280) (secondary — source of the "overthinking"/`reasoning_effort=xhigh` field reports and side-by-side benchmark deltas cited in §3.1/§2)
- [Qwen3.8-27B: Specs, Benchmarks & Verdict — kingy.ai](https://kingy.ai/blog/qwen3-8-27b-specs-benchmarks-local-hardware/) (secondary)
- [Qwen3.8-27B Performance, benchmarks, GPU requirements — Northflank](https://northflank.com/blog/qwen3-8-27b-performance-benchmarks-gpu-requirements-and-how-to-run-it) (secondary — corroborates benchmark figures as Alibaba's own)
- [Qwen3.8-Max Open Weights Are Live — explainx.ai](https://www.explainx.ai/blog/qwen3-8-max-open-weights-live-hugging-face-august-2026) (secondary — confirms 2026-08-12 HF release of the 2.4T base model, architecturally irrelevant to this doc)

### Laguna XS 2.1
- [poolside/Laguna-XS-2.1 — Hugging Face](https://huggingface.co/poolside/Laguna-XS-2.1) (primary)
- [Laguna XS 2.1 on Ollama](https://ollama.com/library/laguna-xs-2.1) (primary)
- [poolside/Laguna-S-2.1-FP8 — Hugging Face](https://huggingface.co/poolside/Laguna-S-2.1-FP8) (primary — notes an August checkpoint refresh of the *different*, larger Laguna S 2.1 model, not XS 2.1)

### Llama 4 / Meta
- [The Llama 4 Herd — Meta AI Blog](https://ai.meta.com/blog/llama-4-multimodal-intelligence/) (primary)
- [Llama4 Tags — Ollama](https://ollama.com/library/llama4/tags) (primary)

### Gemma 4 / Google
- [Gemma4 Tags — Ollama](https://ollama.com/library/gemma4/tags) (primary)

### Kimi K3 / Moonshot
- [Kimi K3 — Ollama](https://ollama.com/library/kimi-k3) (primary — cloud-only tag confirmed)

### Benchmarks & Leaderboards
- [SWE-bench Verified — Vals.ai](https://www.vals.ai/benchmarks/swebench) (primary-adjacent leaderboard)
- [SWE-bench Verified Leaderboard — BenchLM.ai](https://benchlm.ai/benchmarks/sweVerified) (secondary)
- [RewardBench 2: ICLR 2026 Paper](https://arxiv.org/pdf/2506.01937) (primary — no fresher judge-benchmark data found)

### GLM-5.3 / Z.ai, MiniMax M3, Mistral, Devstral, Cohere
- [Z.ai Ships GLM-5.3 Without Retraining the Base Model — MarkTechPost](https://www.marktechpost.com/2026/08/14/z-ai-ships-glm-5-3-without-retraining-the-base-model-better-at-complex-coding-and-long-horizon-tasks/) (secondary — confirms unchanged 743B/40B active base, weights not yet public as of 2026-08-21)
- [Z.ai Launches GLM-5.3 With Frontier Coding — Unite.AI](https://www.unite.ai/z-ai-launches-glm-5-3-with-frontier-coding-and-a-cyber-capability-that-outgrew-its-training/) (secondary)
- [zai-org/GLM-5.2 architecture reference — vLLM recipes](https://recipes.vllm.ai/zai-org/GLM-5.2) (primary-adjacent — 743B total/39B active confirmed, basis for GLM-5.3's unchanged-base arithmetic)
- [Run GLM-5.2 Locally — GLM5.app](https://glm5.app/blog/how-to-run-glm-5-2-locally)
- [MiniMax M3 Open Weights — Nerova](https://nerova.ai/news/minimax-m3-open-weight-agent-builders-june-2026)
- [mistral-medium-3.5:128b — Ollama](https://ollama.com/library/mistral-medium-3.5) (primary)
- [Devstral Small 2 — Ollama](https://ollama.com/library/devstral-small-2) (primary)
- [Meet North Mini Code — MarkTechPost](https://www.marktechpost.com/2026/06/11/meet-north-mini-code-coheres-30b-open-weight-mixture-of-experts-model-with-3b-active-parameters-for-agentic-coding/)

### Llama 5 / Meta (checked, excluded — see §1)
- [Meta Releases Llama 5: 600B Parameters, 5M Token Context — RAGyfied](https://ragyfied.com/articles/meta-llama-5-released) (secondary — confirms ~600B MoE, 5M-token context, no smaller "Scout"-class sibling found)

### DeepSeek R2 (checked, not a real release)
- Multiple secondary sources (decodethefuture.org, felloai.com, layer3labs.io) describe DeepSeek R2 as **rumored/unreleased as of this window** — "no official R2 announcement, no R2 entry in DeepSeek's API, no official model card." Treated as speculation, not a candidate. What DeepSeek actually shipped in this window is V4-Flash/V4 Pro, already tracked above.

### Reference
- [Ollama VRAM Requirements 2026 — Local AI Master](https://localaimaster.com/blog/ollama-model-ram-vram-table)
- [GGUF Quantization Guide — Easton Blog](https://eastondev.com/blog/en/posts/ai/20260422-ollama-gguf-quantization/)
