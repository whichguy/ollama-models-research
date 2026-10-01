# Ollama Models Research — M5 Max 128 GB

**Last updated:** 2026-10-01 (0 picks changed) · **Hardware target:** Apple M5 Max · 128 GB unified memory · 614 GB/s bandwidth
**Most recent research window:** Last 60 days (2026-08-02 → 2026-10-01)

> This is a **single living document** — it is updated in place every refresh
> (roughly every 10 days: the 1st, 11th, and 21st of each month). There is no
> per-month copy of this file. **Full history lives in git**, not in dated
> filenames — run `git log -- ollama-models.md` or see `COMMIT_FORMAT.md`'s
> Step 0 protocol to reconstruct what changed and when. If you're reading
> this in the GitHub UI, "History" on this file is the changelog.

## Current Picks at a Glance (as of 2026-10-01)

The single fastest way to answer "what's the latest recommendation right now" — no need to read further unless you want the rationale.

| Role | Top pick | Smaller pick |
|------|----------|---------------|
| 3.1 Generalist agentic default | **Qwen3.8-27B** (~18 GB) | Gemma 4 12B Unified (~7.6 GB) |
| 3.2 Code implementer | **Ornith-1.5-35B** (~23 GB) | Ornith-1.5-9B (~6.6 GB) |
| 3.3 Code debugger | **DeepSeek-R1-Distill-Qwen-32B** (~20 GB) | Qwen3.8-27B, thinking mode (~18 GB) |
| 3.4 Plan orchestrator | **Llama 4 Scout** (~67 GB) | Qwen3.8-27B (~18 GB) |
| 3.5 LLM-as-judge / verifier | **Qwen3.8-27B** (~18 GB) | Gemma 4 12B Unified (~7.6 GB) |
| 3.6 Document understanding | **Gemma 4 26B MoE** (~19 GB) | Mistral OCR 4.1 (container, not Ollama-loadable) |
| 3.7 Vision / image understanding | **Llama 4 Scout** (~67 GB) | Gemma 4 E4B (~9.6 GB) |

Every pick below is cross-referenced by section number (§3.1–§3.7). No pick changed this cycle — see "Recent Changes" immediately below for what was checked, and git log for the last substantive move (2026-09-21, code-implementer).

---

> **Recent Changes (as of the 2026-10-01 refresh; previous state was commit `a1efad2`, 2026-09-21):**
>
> **No role picks changed this cycle** — all 7 lines carry forward from the
> 2026-09-21 refresh. This was a genuinely researched cycle, not a rubber
> stamp: the Qwen 4 preview watch item flagged last cycle resolved (see below),
> three new cloud-only MoE releases were evaluated and excluded, every
> currently-pulled Ollama tag was re-verified against its live tags page with
> no drift found, and the Ollama platform advanced two stable releases plus an
> experimental preview track.
>
> **Qwen 4 previewed at Apsara, not shipped — first entry added to the Pending
> Releases watchlist.** At Alibaba's Apsara Conference (Sep 22, 2026), Qwen
> project lead Liu Dayiheng named four upcoming tiers — **Qwen 4 Max, Plus,
> Flash, and 27B** — confirming the family is in training on a new-generation
> architecture, with a release "very soon" but no committed date. This is a
> genuine step up from last cycle's unconfirmed-rumor status: it's now an
> on-the-record vendor statement from the Qwen team itself at their own event,
> not a leak. **It still fails the verification gates for a pick** — no model
> card, no weights, no benchmark scores, and no Ollama tag exist for any of the
> four tiers as of this writing. **Qwen 4 27B** is the one tier that matters for
> this doc (size-comparable to the current Qwen3.8-27B pick, which it would
> directly threaten in 4 of 7 roles if it ships with competitive benchmarks) —
> added to the §2 Pending Releases table, the first entry since the table was
> created 2026-08-15. **Not added:** Qwen 4 Max/Plus/Flash, which are
> API-tier/cloud-scale products, not locally-relevant regardless of size.
>
> **DeepSeek V5 and "Kimi K4" re-checked, still unconfirmed — not added.**
> Both were directly re-searched this cycle. DeepSeek has made no announcement,
> roadmap, or date for a V5; circulating "September" dates still trace to
> unverified social-media leaks, not DeepSeek itself. Moonshot's only public
> signal on a next-gen Kimi remains the same single third-party report (The
> Information, 2026-07-29) about GPU procurement carried forward from prior
> cycles — no new signal this cycle. Neither meets the Step 0.5 scope-discipline
> bar (an actual vendor promise), unlike Qwen 4 above.
>
> **Three new cloud-only MoE releases evaluated, none promoted (arithmetic +
> tag confirmed):** **DeepSeek-V4.1-Flash** (552B-parameter MoE backbone, 8B
> active at prefill / 16B at decode, per its own Ollama library page) ships as
> `deepseek-v4.1-flash:cloud` only — no local tag exists, consistent with
> arithmetic (552B × ~4.5 bpw ÷ 8 ≈ 310 GB at Q4, before accounting for MoE's
> all-experts-resident requirement). **MiniMax H3 Max** (Sep 8, 2026) and
> **MiniMax M3.1-Flash-Preview** (428B total/~23B active MoE, Sep 27, 2026,
> API-preview only via MiniMax's own Token Plan/MiniMax Code, no Ollama tag
> found) both fail on size alone — the latter's arithmetic: 428B × ~4.5 bpw ÷ 8
> ≈ 241 GB at Q4. See §1/§2.
>
> **IBM Granite 4.2 evaluated, not promoted — `[unverified]` on benchmarks.**
> New to the Ollama library this cycle (dense, Apache 2.0, up to 30B/~18 GB),
> enterprise-focused with a controllable thinking mode. No benchmark numbers
> against any of this doc's 7 roles were found on its own model card or
> independently — fails verification gate 3 (no primary or agreeing-secondary
> benchmark), so it cannot receive a pick regardless of its plausible size. See
> §2.
>
> **No Ornith 2.0 or successor to the 1.5 generation found.** Directly
> searched this cycle — DeepReinforce's most recent Ornith release remains the
> 1.5 family (Aug 19, 2026); the code-implementer picks are unchallenged within
> their own family this cycle.
>
> **Platform:** Ollama's stable release advanced two more versions since
> 2026-09-21's v0.34.2 — **v0.34.3** (2026-09-19, already noted last cycle as
> pre-release, now confirmed shipped) → **v0.34.4** (2026-09-23, single-pass
> structured output on thinking models, Qwen3.8 prompt-processing optimizations
> for Apple Silicon, fixes an intermittent "model not found" error on large
> libraries) → **v0.35.0** (2026-09-28, now the stable/"Latest" release — adds
> a `/v1/systemone` decision-model endpoint for classification/scoring tasks,
> with two new small models, Nimble and Tev1, neither relevant to this doc's 7
> roles). **v0.35.1** (2026-09-29) and **v0.40.0** (2026-09-25) are both
> pre-release in parallel tracks — v0.40.0 is the more consequential one to
> watch: it makes Ollama **run supported model architectures on the MLX
> runtime automatically by default on Apple Silicon**, rather than requiring
> an explicit MLX-tagged pull. This is a meaningful forward-looking runtime
> change for every pick in this doc's tok/s table, but it is pre-release and
> **no tok/s or resident-size re-measurement was done this cycle** — treat §1's
> existing Ollama-MLX column as still representing the opt-in path until
> v0.40.0 (or its successor) reaches stable. See §1 Runtime Notes.
>
> **Tag re-verification (no drift found):** `qwen3.8:27b` (18 GB, all variants
> including `-mlx`/`-mtp` confirmed), `ornith-1.5:35b` (23 GB), `ornith-1.5:9b`
> (6.6 GB), `llama4:scout` (67 GB), `deepseek-r1:32b` (20 GB), `gemma4:26b`
> (~18–19 GB across its quant variants), `gemma4:12b` (~7.7–8.0 GB), and
> `gemma4:e4b` (~9.5–9.6 GB) were all re-fetched directly against their live
> Ollama tags pages this cycle and confirmed unchanged from the prior refresh.
>
> **No new M5 Max-specific throughput data found this cycle.** The same
> previously-cited benchmark sources (LLMCheck, oMLX, PromptQuorum, Presenc AI)
> were re-checked; no fresh dated measurements for any current pick surfaced.
> §1's tok/s table is carried forward unchanged — not because it was skipped,
> but because nothing newer was found to supersede it.
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
| Ornith-1.5-9B | Dense 9B | ~5.9 bpw | ~72 (extrapolated†) | ~114 (extrapolated†) | *(new 2026-09-21)* No M5 Max figure found this cycle — too new for community benchmark coverage; extrapolated by analogy to Ornith-1.0-9B below, not measured |
| Ornith-1.0-9B *(superseded as smaller pick, still valid — see §3.2)* | Dense 9B | Q4_K_M / 4bit | ~75 (extrapolated†) | ~120 (extrapolated†) | **109.0 (oMLX, 4bit, 4K ctx, 2026-06-28)‡‡** — degrades to 44.4 at 195K ctx |
| Qwen3.6-27B | Dense 27B | Q4_K_M | ~45 | ~70 | ~63 (X/community post, runtime unstated, 2026-08-11)‡‡ |
| **Qwen3.8-27B** | Dense 27B (hybrid Gated-DeltaNet/Attention) | Q4_K_M / 4bit | ~46 (extrapolated†) | ~71 (extrapolated†) | **~58.7 short-context (oMLX, `oQ4e-mtp`, 40-core, 2026-09 dated)‡‡, corroborating the prior 63.3–65.2 single-source MTPLX figure within ~10%** — degrades to **~10.6 tok/s** at the top of its 262K context window (oMLX, same pattern as Ornith-1.0-9B's long-context degradation below) |
| Qwen 3.5 30B-A3B | MoE 30B/3B active | Q4_K_M | ~45 | ~55 | ~68 (mlx_lm) |
| Laguna XS 2.1 | MoE 33B/3B active | Q4_K_M / 4bit | ~43 | ~53§§ | **87–107 (oMLX, 4bit, 1K–4K ctx, 2026-08-10/11)‡‡** |
| **Ornith-1.5-35B** | MoE 35B/~3B active | ~5.3 bpw / 8bit | ~43 (extrapolated†) | ~53 (extrapolated†) | **96.7–100.7 avg (llm-bench.io, MLX 8bit — not 4bit, ~35.3 GB resident at 8bit, 2026-09 dated)‡‡** — not directly comparable to the Q4_K_M-class figures elsewhere in this table; no 4bit/oMLX M5 Max figure found this cycle |
| Ornith-1.0-35B *(superseded as top pick, still valid — see §3.2)* | MoE 35B/~3B active | Q4_K_M / 4bit | ~42 (extrapolated†) | ~52 (extrapolated†) | **77.0–123.4 (oMLX, 4bit, batch 1×–8×, 2026-07-02)‡‡** |
| DeepSeek-R1-Distill 32B | Dense 32B | Q4_K_M / 4bit | ~27 | ~45 | **27.6–28.9 (oMLX, 4bit, 1K ctx, 2026-07-15)‡‡** / ~60 (mlx_lm, older est.) |
| Llama 4 Scout | MoE 109B/17B active | Q4_K_M | ~22 | ~26 | ~50 (mlx_lm, no fresh M5 Max row found this cycle) |
| Llama 3.3 70B | Dense 70B | Q4_K_M / 4bit | ~12–18 | ~15–22 | **12.7–13.2 (oMLX, 4bit, 1K–4K ctx, 2026-07-20)‡‡** / ~7 (oMLX 8bit, older) |
| Mistral Medium 3.5 | Dense 128B | Q4_K_M (80 GB)§ | ~6 | ~7 | **7.1–7.2 (oMLX, 4bit, 4K ctx, 2026-07-16)‡‡** — matches prior bandwidth-ceiling estimate |

† No direct M5 Max measurement found this cycle for the "extrapolated" rows above
— treat as directional, not measured. Ornith-1.0-35B's own oMLX row (77–123 tok/s)
is real data but was not measured on the Ollama runtime specifically, hence its
Ollama/Ollama-MLX columns remain extrapolated by analogy.

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
- **Ollama v0.34.3–v0.35.0 stable (current "Latest" v0.35.0 as of 2026-10-01),
  plus v0.40.0 experimental preview:** since the 2026-09-21 refresh's v0.34.2,
  **v0.34.3** (2026-09-19) shipped as stable — exposes thinking controls/
  defaults via `GET /api/show`, adds Nemotron H vision support on Apple
  Silicon; **v0.34.4** (2026-09-23) optimizes Qwen3.8 prompt processing for
  Apple Silicon and fixes an intermittent "model not found" error on large
  libraries; **v0.35.0** (2026-09-28, now stable/"Latest") adds a
  `/v1/systemone` decision-model endpoint (new models Nimble, Tev1 — not
  relevant to this doc's 7 roles) and fixes stalled MLX downloads. Two
  pre-release tracks are running in parallel: **v0.35.1** (2026-09-29, patch
  pre-release) and, more significantly, **v0.40.0** (2026-09-25) — which makes
  Ollama **run MLX-supported architectures on the MLX runtime automatically by
  default on Apple Silicon**, removing the need for an explicit MLX-tagged
  pull. This would fold the "Ollama MLX" column below into the plain "Ollama"
  column for supported models once stable, but it is pre-release and **not yet
  re-measured** — none of these releases change a measured tok/s figure in the
  table above this cycle.
- **MoE-A3B behaviour:** Models like **Ornith-1.5-35B** (top code-implementer
  pick as of 2026-09-21, 35B/~3B active, ~23 GB), **Laguna XS 2.1** (33B/3B
  active, ~20 GB), and **Qwen 3.5 30B-A3B** (30B/3B active, ~17 GB) activate
  only ~3B parameters per token despite loading a much larger weight set —
  30B-class quality at 40–110+ tok/s on M5 Max depending on runtime. This
  remains the highest quality-per-GB pattern in the current Ollama library,
  and fresh oMLX/community data confirms it holds up under direct M5 Max
  measurement, not just theory.
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
| **Ornith-1.5-397B** (deepreinforce-ai/Ornith, 397B/~3B active MoE) *(updated 2026-09-21, supersedes the prior Ornith-1.0-397B row)* | Ollama's own tag `ornith-1.5:397b` resolves directly at **242 GB** — confirmed by direct fetch, not estimated. Arithmetic: 242 GB ÷ 397B ≈ 4.9 bits/weight, consistent with a Q4-class quant plus vision-encoder overhead. Roughly 2× the 128 GB envelope; an extreme ~Q2 requantization (~120 GB) would be numerically borderline but untested for quality. **Effectively cloud-only.** Note: the same family's 9B and 35B members fit comfortably and are this doc's current code-implementer picks — see §3.2. |
| **GLM-5.2** (Z.ai, 744B/40B active MoE) | 62.1% SWE-bench Pro, 91.2% GPQA Diamond; smallest usable GGUF ~217 GB; Ollama only offers `:cloud`. A "GLM-5.5" successor is rumored (analyst projection, not an official Z.ai announcement) — not treated as a real release. |
| **MiniMax M3** (MiniMax, ~428B/23B active MoE) | 80.5% SWE-bench Verified; smallest local quant ~143 GB; Ollama only offers `:cloud`. MiniMax's newer **H3** (~465B total/30B active MoE, omni-modal — video + stereo audio) has since shipped *(new 2026-09-11)*: 465B × ~4.5 bpw ÷ 8 ≈ **261 GB at Q4**, still cloud-only by arithmetic regardless of shipping status. |
| Llama 4 Maverick (400B total) | Q4 ≈ 200 GB — hard OOM |
| Kimi K2.7 Code (1T total) | Community GGUF builds exist but ~585 GB at Q4; cloud-only for this hardware |
| **DeepSeek V4 Pro** *(the doc's prior "DeepSeek-V4-Pro-Max" — 2026-08-15: left preview, went GA 2026-08-12 as "V4 Pro 0813")* (1.6T/49B active MoE) | 80.6% SWE-bench Verified — still one of the highest open-weight scores found — but ~800 GB minimum at Q4; a circulating "~50 GB" claim is arithmetically impossible for 1.6T params and was rejected early in this doc's history. Re-checked 2026-08-15: Ollama added it to its library as a **`:cloud` tag only** — confirms cloud-only independent of the arithmetic. Community (non-official) GGUF conversions exist but are unverified against DeepSeek's checkpoints. |
| DeepSeek-V3 full (671B MoE, 37B active) | Q4 ≈ 170 GB — borderline, likely OOM with KV-cache |
| **Qwen3.8-2.4T-A95B** (Alibaba, 2.4T total, ~95B active — the base model behind the Qwen3.8-Max API) | Open weights shipped 2026-08-12 (Hugging Face + ModelScope, FP8 quant available), but this changes nothing for local use. Arithmetic: 2.4T × ~4.5 bpw ÷ 8 ≈ **1.2+ TB at Q4**, ~750 GB even at an aggressive 2-bit quant — off by close to an order of magnitude versus 128 GB, not a borderline case. No Ollama tag exists. **Still not evaluable as a local candidate, weights or not.** Note: the size-*comparable* companion model, Qwen3.8-27B, shipped separately 2026-08-14 and **does** fit — see §3.1/§3.5, it is now this doc's generalist/judge top pick. |
| **Tencent Hy4 Preview** (770B total/49B active MoE) | *(new 2026-09-01)* Released 2026-08-28; reported by Cline as leading SWE-bench Pro among late-August releases. Arithmetic: 770B × ~4.5 bpw ÷ 8 ≈ **433 GB at Q4** — over 3× the envelope. **Cloud-only by arithmetic**, not independently tag-checked given the margin. |
| **GLM-5.3-Flash** (Z.ai, 320B total/18B active MoE) | *(new 2026-09-01)* Ollama's own tags page lists only `glm-5.3-flash:cloud` — no local/GGUF tag exists. Arithmetic independently confirms: 320B × ~4.5 bpw ÷ 8 ≈ **180 GB at Q4**. **Cloud-only, confirmed by Ollama's own tag listing.** |
| **Qwen3.8-Flash-Next** (Alibaba, 125B main + 51B N-gram-embedding component, ~6B active/token) | *(new 2026-09-01)* Released 2026-08-26 as an "experimental preview of the architecture that will underpin Qwen4." Smallest Ollama tags (`125b-a6b-nvfp4`, `125b-mlx`) both resolve at **105 GB** — arithmetic checks out (176B combined params × ~4.8 bpw ÷ 8 ≈ 105.6 GB, consistent with the listed size). Technically clears this doc's ~110 GB single-model ceiling but leaves only ~15–20 GB free, thin for a model whose selling point is 262K–1M context (large KV-cache needs). **Not cloud-only, but not promoted this cycle** — see the Recent Changes callout and §2. |
| **GLM-5.3** (Z.ai, full flagship — distinct from GLM-5.3-Flash below, 744B total/~40B active MoE) | *(new 2026-09-11)* Weights published to Hugging Face (`zai-org/GLM-5.3`) in BF16 (~1.5 TB) and FP8 (~750 GB) around 2026-08-28. Arithmetic: 744B × ~4.5 bpw ÷ 8 ≈ **418 GB at Q4** — over 3× the envelope, consistent with the primary-sourced ~750 GB FP8 figure. No Ollama tag (local or `:cloud`) found for the full flagship. **Cloud-only by arithmetic.** |
| **MiniMax H3** (MiniMax, ~465B total/30B active MoE, omni-modal) | *(new 2026-09-11)* 465B × ~4.5 bpw ÷ 8 ≈ **261 GB at Q4**. See the corrected MiniMax M3 row above. **Cloud-only by arithmetic.** MiniMax shipped a further **H3 Max** variant 2026-09-08 — not independently sized this cycle given the margin already established by H3 itself. |
| **DeepSeek-V4.1-Flash** (DeepSeek, 552B-parameter MoE backbone, 8B active at prefill / 16B at decode) | *(new 2026-10-01)* Ollama's own library page lists `deepseek-v4.1-flash:cloud` only — no local tag exists. Arithmetic independently backs this up: 552B × ~4.5 bpw ÷ 8 ≈ **310 GB at Q4** (before accounting for MoE's all-experts-resident requirement, which the low active-parameter count doesn't change). **Cloud-only, confirmed by Ollama's own tag listing.** |
| **MiniMax M3.1-Flash-Preview** (MiniMax, 428B total/~23B active MoE) | *(new 2026-10-01)* Released 2026-09-27 as an API-only preview via MiniMax's own Token Plan/MiniMax Code — no Ollama tag found. Arithmetic: 428B × ~4.5 bpw ÷ 8 ≈ **241 GB at Q4**. **Cloud-only by arithmetic.** |
| Any 700B+ total-parameter model | Exceeds envelope at any practical quant |

---

## 2. Current Model Landscape (Last 60 Days)

### Models that moved SOTA or shipped new (Aug 2 – Oct 1, 2026)

| Model | Family | Released | Why it matters |
|-------|--------|----------|-----------------|
| **Qwen 4 (Max/Plus/Flash/27B)** | Alibaba/Qwen | Previewed Sep 22, 2026 (Apsara Conference); not shipped | Vendor-confirmed in training, no date/weights/benchmarks yet. **Qwen 4 27B** would directly threaten Qwen3.8-27B (4 of 7 roles) if it ships competitively — added to the Pending Releases watchlist below, first entry since the table's creation. See Recent Changes callout. |
| **DeepSeek-V4.1-Flash** | DeepSeek (MoE, 552B backbone/8B-16B active) | ~Sep 2026 | New version of the previously-tracked V4-Flash with a larger backbone. `:cloud` tag only on Ollama — cloud-only confirmed at the tag level, also fails arithmetic (~310 GB at Q4). See §1. |
| **Ornith-1.5-35B / 1.5-9B** | DeepReinforce/Ornith (MoE 35B/~3B active and dense 9B) | Aug 19, 2026 (spotted this cycle) | **New code-implementer top + smaller picks**, displacing the 1.0-generation. Now multimodal (text+image). 79% / 70.6% SWE-bench Verified (vs. 1.0's 75.6% / 69.4%), 67.8–68.5% / 46.2–47% Terminal-Bench 2.1 (vs. 64.2% / 43.1%). MIT license, 256K context, separate `ornith-1.5` Ollama tag namespace confirmed. See §3.2. |
| **Qwen3.8-27B** | Alibaba/Qwen (dense 27B, hybrid Gated-DeltaNet/Attention) | Aug 14, 2026 | **New generalist + judge top pick; new code-debugger + plan-orchestrator smaller pick.** Direct successor to Qwen3.6-27B at the same footprint (18 GB vs 17 GB Q4, Apache 2.0, now natively multimodal). Beats it on every directly-comparable primary-sourced benchmark: GPQA Diamond 89.2 vs 87.8, LiveCodeBench v6 90.3 vs 83.9, SWE-bench Pro 61.7 vs 53.5. Does not report SWE-bench Verified (Qwen3.6-27B's prior headline number) — see Recent Changes callout for the verification-gate handling. See §3.1/§3.3/§3.4/§3.5. |
| **Tencent Hy4 Preview** | Tencent (MoE 770B/49B active) | Aug 28, 2026 | Reported by Cline as SWE-bench Pro leader among late-Aug releases, but 433 GB at Q4 — over 3× the envelope. **Cloud-only by arithmetic.** See §1. |
| **Z.ai GLM-5.3-Flash** | Z.ai (MoE 320B/18B active) | ~late Aug 2026 (exact date not independently confirmed) | MIT, natively multimodal, 1M context. Ollama offers only `glm-5.3-flash:cloud`. **Cloud-only, confirmed by Ollama's own tag listing** — also fails arithmetic (~180 GB at Q4). See §1. |
| **Qwen3.8-Flash-Next** | Alibaba/Qwen (ultra-sparse MoE, 125B main + 51B N-gram-embedding, ~6B active/token) | Aug 26, 2026 | Experimental preview of the architecture underpinning Qwen4. Smallest Ollama tags resolve at 105 GB — clears this doc's ~110 GB single-model ceiling but with thin KV-cache headroom for its own long-context selling point. **Evaluated for plan-orchestrator, not promoted** — no comparative agentic benchmark vs. Llama 4 Scout found this cycle; "experimental preview" status argues for a field track record first. See §1 and Recent Changes. |
| **Ornith-1.0-35B** | deepreinforce-ai (MoE 35B/~3B active) | Jun 25, 2026 (score confirmed 2026-08-11) | **Beaten as code-implementer top pick by Ornith-1.5-35B this cycle** — see the entry above. 75.6% SWE-bench Verified, confirmed by primary (DeepReinforce blog + HF card) and independent secondary (BenchLM.ai). Remains a fully valid alternative, not deprecated. MIT license, 256K context, text-only. |
| **Muse Glimmer** | Meta Superintelligence Labs (dense 30B + 1.8B vision encoder) | Aug 10, 2026 | Purpose-built for local agentic workloads; Meta's own comparison table predates Qwen3.8-27B's release (Aug 14), so the SWE-bench Verified comparison (76.0 vs Qwen3.6-27B's 77.2) has not been re-run against the new top pick this cycle — flagged, not re-benchmarked. Apache 2.0, ~18 GB Q4. **Still not promoted.** See §3.1. |
| **NVIDIA Nemotron 3.5 Lightning** | NVIDIA (hybrid Mamba+MoE, 30B total/3B active) | Aug 11, 2026 | NVIDIA's own card lists only 52.80% SWE-bench Verified — well below every code-implementer pick in this doc. **Not promoted to any role**, unchanged assessment. |
| **DeepSeek V4-Flash / V4 Pro** | DeepSeek (MoE 284B/13B active and 1.6T/49B active respectively) | Jul 31, 2026 (Flash); V4 Pro GA 2026-08-12 | Both confirmed cloud-only **by Ollama's own tag listing**, not just by size arithmetic — see §1. A `DeepSeek-V4-Flash-Vision-Exp` variant appeared 2026-08-21 (secondary source); presumed cloud-only by inheritance from V4-Flash, not independently tag-checked this cycle since it wouldn't change the verdict either way. |
| **Qwen3.8-2.4T-A95B** | Alibaba/Qwen (MoE 2.4T/~95B active) | Open weights Aug 12, 2026 | ~1.2+ TB at Q4, never a size-plausible local candidate. Its size-comparable companion, Qwen3.8-27B, is the entry above and fits comfortably. |
| **Laguna XS 2.1** | Poolside (MoE 33B/3B active) | Jul 2, 2026 | Beaten as code-implementer top pick by Ornith-1.0-35B (2026-08-11 cycle) — remains a fully valid alternative. Not deprecated. |
| **Kimi K3** | Moonshot AI (MoE 2.8T/50.4B active) | Jul 16, 2026 (announced); open weights Jul 26, 2026 | Still no local GGUF/Ollama/MLX support (`kimi-k3:cloud` only). **Cloud-only — see §1.** No K3.5/K4 successor found this cycle. |
| **Ollama v0.33.3–v0.35.0** | Platform release | Aug–Sep 2026 | Current stable/"Latest" is v0.35.0 (2026-09-28); v0.40.0 (2026-09-25) is an experimental pre-release track that makes MLX-supported architectures run on MLX by default on Apple Silicon. See Runtime Notes. |
| **GLM-5.3** (full flagship) | Z.ai (MoE 744B/~40B active) | ~Aug 28, 2026 (weights) | *(new 2026-09-11)* 744B × ~4.5 bpw ÷ 8 ≈ 418 GB at Q4 — matches the primary-sourced ~750 GB FP8 figure. No Ollama tag found. **Cloud-only by arithmetic.** See §1. |
| **MiniMax H3** | MiniMax (MoE ~465B/30B active, omni-modal) | ~Aug–Sep 2026 | *(new 2026-09-11)* Previously tracked as unshipped; now shipped. 465B × ~4.5 bpw ÷ 8 ≈ 261 GB at Q4. **Cloud-only by arithmetic.** See §1. |
| **OpenSWE-72B** | GAIR-NLP (dense 72B, AGPL-3.0) | ~Aug 2026 | *(new 2026-09-11)* 66.0% SWE-bench Verified (SWE-Agent scaffold), 68.0% combined with SWE-rebench — below both code-implementer picks at ~2× Ornith-1.0-35B's size (~40 GB Q4). No Ollama-loadable tag found this cycle. **`[unverified]` on Ollama availability; not promoted — scores lower regardless.** See §3.2. |

### Notable models evaluated and excluded (checked repeatedly, still excluded)

| Model | Family | Released | Why noted |
|-------|--------|----------|-----------|
| **GLM-5.2** | Z.ai (MoE 744B/40B active) | Jun 13, 2026 | Doesn't fit 128 GB; `:cloud` only. Superseded in the landscape table above by GLM-5.3-Flash, also cloud-only. |
| **MiniMax M3** | MiniMax (MoE ~428B/23B active) | Jun 1, 2026 *(one secondary source this cycle instead states Aug 24, 2026 — date discrepancy flagged, not resolved; doesn't affect the cloud-only conclusion either way)* | Doesn't fit; `:cloud` only. Newer "H3" (~465B/30B active, omni-modal) has since shipped *(2026-09-11)* — also doesn't fit, see §1. |
| **OpenSWE-72B** | GAIR-NLP (dense 72B, AGPL-3.0) | ~Aug 2026 | 66.0–68.0% SWE-bench Verified — below every current code-implementer pick, at roughly 1.7× the top pick's (Ornith-1.5-35B, ~23 GB) resident size. No Ollama tag found this or the prior cycle. See §3.2. |
| **NVIDIA Nemotron 3 Super** | NVIDIA (hybrid Mamba-Transformer, 120B/12B active) | Mar 11, 2026 | ~60 GB at Q4, but 60.47% SWE-bench Verified, below all current picks. Not selected. |
| **IBM Granite 4.2** | IBM (dense, up to 30B) | ~Sep 2026 (new to Ollama this cycle) | Apache 2.0, up to ~18 GB at 30B, controllable thinking mode. `[unverified]` — no benchmark numbers found on its own model card or independently against any of this doc's 7 roles. Fails verification gate 3 regardless of plausible size/footprint. |
| **MiniMax M3.1-Flash-Preview** | MiniMax (MoE 428B/~23B active) | Sep 27, 2026 | API-preview only (MiniMax Token Plan/MiniMax Code); no Ollama tag. ~241 GB at Q4 — fails on size alone even disregarding availability. |

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

| Model | Promised | Would threaten | Last checked |
|-------|----------|-----------------|---------------|
| **Qwen 4 27B** (Alibaba/Qwen, dense, successor to Qwen3.8-27B's size class) | Named at Alibaba's Apsara Conference (Sep 22, 2026) by Qwen project lead Liu Dayiheng; in training, release "very soon," no committed date | Qwen3.8-27B in all 4 roles it currently holds (generalist top, judge top, code-debugger smaller, plan-orchestrator smaller) — *if and only if* it ships with competitive benchmarks at a comparable footprint | 2026-10-01 (first added this cycle) |

**First entry added this cycle, 2026-10-01.** Qwen 4 27B is the first model to
enter this table since its creation 2026-08-15. This clears the bar the prior
"Qwen 4" rumor didn't: Alibaba's own team named it on the record at their own
conference (not a leak), and the 27B tier is explicitly size-comparable to a
current pick. It does **not** yet clear this doc's verification gates for an
actual pick — no model card, weights, Ollama tag, or benchmark score exists
for any of the four Qwen 4 tiers as of this writing; Qwen 4 Max/Plus/Flash are
excluded from this table entirely as API-tier/cloud-scale products that
wouldn't be locally relevant regardless of shipping status. **DeepSeek V5**
and a rumored **"Kimi K4"** were also re-searched this cycle and remain
unconfirmed — no vendor announcement, model card, or committed date for
either — so neither qualifies for this list. Re-check Qwen 4 27B's status
first at the next cycle (2026-10-11); if it ships, evaluate it as a normal
candidate against the four roles above per the usual gates.

### SWE-bench Verified Snapshot (Oct 1, 2026 — unchanged from Sep 21, 2026)

For context on where role picks sit in the broader leaderboard. Frontier
(non-open-weight, API-only) scores remain omitted from precise citation —
trackers still disagree on exact figures for the current frontier and none of it
changes a local pick. **Qwen3.8-27B is intentionally absent from this table** —
its own model card does not report a SWE-bench Verified score (see Recent
Changes callout); its SWE-bench Pro score (61.7%, vs Qwen3.6-27B's 53.5%) isn't
directly comparable to the Verified-benchmark numbers below, so it isn't force-fit
into this specific snapshot.

| Model | Score | Locally runnable? |
|-------|-------|--------------------|
| Kimi K3 | 93.4% (vals.ai) | No (`:cloud` only, 2.8T params) |
| **Ornith-1.5-397B** | **86%** | No — 242 GB (`ornith-1.5:397b`, confirmed tag), *(new 2026-09-21)* effectively cloud-only, see §1 |
| Ornith-1.0-397B | 82.4% | No (200 GB Q4) — superseded within-family by Ornith-1.5-397B above |
| DeepSeek V4 Pro | 80.6% | No (`:cloud` tag only, 800 GB Q4) |
| MiniMax M3 | 80.5% | No (143 GB min) |
| **Ornith-1.5-35B** | **79%** | **Yes (~23 GB) — current top local coding pick** *(new 2026-09-21, see §3.2)* |
| Mistral Medium 3.5 | 77.6% | **Yes (80 GB Q4 — slow, ~7 tok/s)** |
| Qwen3.6-27B | 77.2% | Yes (~17 GB Q4) — **beaten as top pick by Qwen3.8-27B, see §3.1** |
| **Muse Glimmer** | 76.0% | Yes (~18 GB Q4) — **evaluated, not promoted, see §3.1** |
| Ornith-1.0-35B | 75.6% | Yes (~21 GB Q4) — **beaten as top pick by Ornith-1.5-35B, still a valid alternative, see §3.2** |
| **Laguna XS 2.1** | **70.9%** | **Yes (~20 GB Q4) — beaten as top pick, still a valid alternative** |
| **Ornith-1.5-9B** | **70.6%** | **Yes (~6.6 GB) — current smaller coding pick** *(new 2026-09-21, see §3.2)* |
| Ornith-1.0-9B | 69.4% | Yes (~5.6 GB Q4) — **beaten as smaller pick by Ornith-1.5-9B, still a valid alternative, see §3.2** |
| Devstral Small 2 | 68.0% | Yes (~15 GB Q4) — still a valid alternative |
| OpenSWE-72B | 66.0–68.0% | `[unverified]` — no Ollama tag found; would be ~40 GB Q4 if pullable, still below every current pick |
| Cohere North Mini Code 1.0 | 67.6% | Yes (~17 GB Q4) |
| NVIDIA Nemotron 3.5 Lightning | 52.80% | Yes (~25 GB) — too low-scoring for any current pick |

Source: [vals.ai SWE-bench leaderboard](https://www.vals.ai/benchmarks/swebench) and [BenchLM.ai](https://benchlm.ai/benchmarks/sweVerified) (both primary-adjacent/secondary, cross-checked); Ornith-1.5 and Muse Glimmer scores verified directly against primary vendor sources per §3.2/§3.1.

---

## 3. Recommendations by Role

### 3.1 Generalist Agentic Default
*One model that does most jobs adequately.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | **Qwen3.8-27B** | ~18 GB | GPQA Diamond 89.2%; LiveCodeBench v6 90.3%; SWE-bench Pro 61.7%; 262K ctx (1M extensible) | `qwen3.8:27b` |
| **Smaller pick** | Gemma 4 12B Unified | ~7.6 GB | MMLU Pro 77.2%; natively multimodal (text + image) | `gemma4:12b` |

**Rationale, as of 2026-09-01:** Qwen3.8-27B (Alibaba, shipped Aug 14, 2026, Apache 2.0) is the direct successor to the prior top pick, Qwen3.6-27B, at effectively the same footprint (18 GB vs 17 GB Q4_K_M — arithmetic checks out: 27.78B params × ~5.2 bpw effective ÷ 8 ≈ 18 GB, within tolerance for a Q4_K_M-class quant carrying a vision encoder). It beats Qwen3.6-27B on every benchmark both primary model cards report: GPQA Diamond 89.2 vs 87.8, LiveCodeBench v6 90.3 vs 83.9, and SWE-bench Pro 61.7 vs 53.5 (an 8.2-point gain, independently matched by a secondary dev.to comparison). **Gate note:** Qwen3.8-27B's own card does not report SWE-bench Verified — Qwen3.6-27B's prior headline metric — so this is a genuine gap in like-for-like comparison, not a worse score; SWE-bench Pro is used as the headline benchmark instead, satisfying the verification gate via a primary-sourced number. Reasoning is on by default (`reasoning_effort: xhigh`) and controlled via `enable_thinking`/`reasoning_effort` API parameters rather than a `/think` prefix — several HF discussion threads report the default effort level over-thinking short prompts; `reasoning_effort: low` is recommended for latency-sensitive generalist use. **Throughput:** a single-source community M5 Max measurement (MTPLX-optimized 4-bit, single-stream) shows 63.3–65.2 tok/s, comparable to Qwen3.6-27B's own ~63–70 tok/s class — no material latency penalty for the benchmark gains (see §1). Gemma 4 12B Unified remains the right answer when memory is at a premium or multimodal input is needed at low cost.

**Evaluated, not selected:** *Muse Glimmer* (Meta Superintelligence Labs, dense 30B, released Aug 10, 2026, Apache 2.0, `muse-glimmer:30b`, ~18 GB Q4) is a genuine contender purpose-built for agentic workloads, winning big on agentic/tool-use benchmarks (MCP-Atlas 75.5, DeepSearch QA 74.6, GAIA2 43.3, WildClawBench 47.6) in Meta's own comparison table. That table predates Qwen3.8-27B's Aug 14 release and only compares against Qwen3.6-27B (SWE-bench Verified 76.0 vs 77.2, TerminalBench 2.1 51.7 vs 60.7, OSWorld-Verified 65.9 vs 75.6). **Closed out 2026-09-11:** an independent comparison (LLM Stats, secondary, aggregating both models' own published numbers) now directly compares Muse Glimmer against Qwen3.8-27B — Qwen3.8-27B wins all 8 directly-compared benchmarks (CharXiv-R, GPQA, Humanity's Last Exam, IFBench, OmniDocBench 1.5, OSWorld-Verified, SWE-Bench Pro, Terminal-Bench 2.1), Muse Glimmer wins 0. Muse Glimmer's real advantage — roughly 4× lower KV-cache memory at long context — doesn't change the verdict here, since Qwen3.8-27B already fits comfortably at this footprint. Combined with the methodology caveat already on record (kingy.ai flagged Meta "selected the more favorable of a competitor's self-reported score or its own reproduction"), this is now a **confirmed non-promotion, not an open "watch next cycle" item**.

**Sizing note:** ~16.8 GB at Q4_K_M; ~19.5 GB at Q5_K_M; ~28.6 GB at Q8_0 for Qwen3.6-27B (retained for reference — still Ollama-servable as `qwen3.6:27b`, just no longer the recommended pick in any role). Qwen3.8-27B's own Ollama tags page lists `q4_K_M` at 18 GB and `q8_0` at 30 GB; the MLX-native tag matches the q4 size at 18 GB.

### 3.2 Code Implementer
*Writes code, multi-file edits, agentic coding loops.*

| | Model | Resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | **Ornith-1.5-35B** | ~23 GB | SWE-bench Verified 79%; Terminal-Bench 2.1 67.8–68.5%; 256K ctx; text+image | `ornith-1.5:35b` |
| **Smaller pick** | Ornith-1.5-9B | ~6.6 GB | SWE-bench Verified 70.6%; Terminal-Bench 2.1 46.2–47%; 256K ctx; text+image | `ornith-1.5:9b` |

**Rationale — top and smaller pick, as of 2026-09-21:** Ornith-1.5-35B and Ornith-1.5-9B (DeepReinforce/Ornith, MIT license, shipped Aug 19, 2026) displace the 1.0-generation picks that held this role since 2026-08-11/2026-08-01. The 1.5 generation extends 1.0's self-scaffolding agentic-RL approach into a full self-improvement loop — the model generates its own tasks, scaffolds, and rollouts for RL rather than training against a fixed human-curated task set — and adds native image input on top of 1.0's text-only design. Per Ornith's own blog (primary, same methodology family as 1.0's prior verified numbers): Ornith-1.5-35B scores **79% SWE-bench Verified** (+3.4pts over 1.0-35B's 75.6%) and **67.8–68.5% Terminal-Bench 2.1** (+3.6–4.3pts over 64.2%, two scaffolds reported — Terminus-2 and Claude Code); Ornith-1.5-9B scores **70.6% SWE-bench Verified** (+1.2pts over 1.0-9B's 69.4%) and **46.2–47% Terminal-Bench 2.1** (+3.1–3.9pts over 43.1%). **All three verification gates pass:** existence and parameter counts (397B/35B/9B) are confirmed by Ornith's own blog and Hugging Face repos (primary); the Ollama tags were fetched directly this cycle — `ornith-1.5:9b` (6.6 GB), `ornith-1.5:35b` (23 GB), and `ornith-1.5:397b` (242 GB) all resolve, under a **separate model slug** from `ornith` (which still serves only the 1.0 generation — no in-place overwrite, so both generations remain independently pullable); headline benchmarks are primary-sourced. **Arithmetic:** 23 GB ÷ 35B ≈ 5.26 bits/weight and 6.6 GB ÷ 9B ≈ 5.87 bits/weight — both modestly above a plain-Q4 estimate (~4.5 bpw) but within the ~25% rejection threshold and consistent with the new vision-encoder component; since the sizes come directly from Ollama's own tags page rather than a secondary blog restatement, this is a passed sanity check, not a correction. **On the speed tradeoff** (this doc's selection criterion that throughput is part of quality): no directly-comparable Q4/Ollama-path M5 Max figure was found for 1.5 this cycle — the only data point located (llm-bench.io, secondary, MLX **8-bit**, not 4-bit) shows Ornith-1.5-35B-A3B averaging 96.7–100.7 tok/s at ~35.3 GB resident (the 8-bit build, not the 23 GB Q4-class tag above) — in the same broad range as 1.0-35B's measured 77–123 tok/s (oMLX, 4-bit), so no red flag on speed, but flagged as a different quant/tool rather than blended into a single number. Resident size grows modestly for both picks (21→23 GB top, 5.6→6.6 GB smaller) with no material impact on any §5 stack combination.

**Also evaluated, not selected:**
- *Ornith-1.0-35B / Ornith-1.0-9B* (deepreinforce-ai, MIT, Jun 25, 2026; 75.6% / 69.4% SWE-bench Verified; `ornith:35b` ~21 GB, `ornith:9b` ~5.6 GB): **beaten as top and smaller pick by Ornith-1.5**, not deprecated. Both remain fully valid, Ollama-servable alternatives — a longer field track record than 1.5, and text-only if multimodal input isn't needed. Teams already standardized on 1.0's behavior have no urgent reason to switch.
- *Laguna XS 2.1* (Poolside, Jul 2, 2026; MoE 33B/3B active; 70.9% SWE-bench Verified; `laguna-xs-2.1:q4_K_M`, ~20 GB, Apache-compatible): beaten as top pick by both Ornith generations, not deprecated. Remains a fully valid alternative with a longer field track record and its own very competitive measured M5 Max throughput.
- *Muse Glimmer* (Meta, dense 30B, Aug 10, 2026; SWE-bench Verified 76.0%; SWE-bench Pro 51.2 vs Qwen3.6-27B's 50.2 — narrowly ahead there but behind on Verified, TerminalBench, and OSWorld): evaluated for this role too given its coding-adjacent benchmark profile, but doesn't clearly beat either current pick on the headline metric. See §3.1 for the full comparison.
- *NVIDIA Nemotron 3.5 Lightning* (30B-A3B MoE, released Aug 11, 2026): 52.80% SWE-bench Verified — well below every pick and alternative in this section. Fast (~25 GB, claimed 4× throughput vs. similar models, unverified independently) but not competitive on quality for this role.
- *Devstral Small 2* (Mistral, 24B dense; 68.0% SWE-bench; `devstral-small-2`, ~15 GB Q4, Apache 2.0): still a valid alternative, especially where Devstral's longer production track record in agentic tool-calling scaffolds (OpenHands, SWE-agent) matters more than footprint.
- *Mistral Medium 3.5* (128B dense; 77.6% SWE-bench Verified; `mistral-medium-3.5:128b`, 80 GB Q4): fits, but at ≈7.1–7.2 tok/s (confirmed by direct oMLX measurement, not just bandwidth arithmetic) a 1K-token completion takes roughly 140–150 seconds versus Ornith-1.5-35B's low tens-of-seconds range — the latency penalty rules it out for agentic loops. Quality-critical one-shot tasks only.
- *OpenSWE-72B* (GAIR-NLP, dense 72B, AGPL-3.0, ~Aug 2026; 66.0% SWE-bench Verified with SWE-Agent, 68.0% combined with SWE-rebench training — both from GAIR's own HF card/GitHub repo, primary): below every current pick on the headline metric while costing roughly 2× Ornith-1.5-35B's resident size (72B × ~4.5 bpw ÷ 8 ≈ 40 GB Q4). Also fails verification gate 2 independently of the score gap — no Ollama-loadable tag (official or community GGUF) was found this or the prior cycle. **Not promoted on either count.**
- *Ornith-1.5-397B* (86%), *DeepSeek V4 Pro* (80.6%), *MiniMax M3* (80.5%), *Kimi K3* (93.4% per vals.ai) top the global leaderboard but none fit 128 GB at practical quants — see §1.

### 3.3 Code Debugger
*Reasoning / chain-of-thought, root-cause analysis, math-heavy debugging.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | DeepSeek-R1-Distill-Qwen-32B | ~20 GB | MMLU 83%; MATH 72% (highest of any sub-40 GB local model) | `deepseek-r1:32b` |
| **Smaller pick** | **Qwen3.8-27B** (`reasoning_effort: xhigh`) | ~18 GB | GPQA Diamond 89.2%; SWE-bench Pro 61.7%; reasoning on by default; same model as §3.1 | `qwen3.8:27b` |

**Rationale:** DeepSeek-R1-Distill-Qwen-32B provides always-on chain-of-thought reasoning and holds the highest MATH benchmark score of any locally-runnable model under 40 GB. Distilled from the 671B R1 teacher; generates full reasoning traces before answering. oMLX M5 Max data (2026-07-15, 4bit) directly measures 27.6–28.9 tok/s at short context — consistent with this doc's Ollama-MLX estimate. No challenger has beaten this pick. The smaller pick moves from Qwen3.6-27B to its successor, **Qwen3.8-27B** (see §3.1 for the full case) — reasoning is on by default at `reasoning_effort: xhigh` (set via API param, not a `/think` prefix), the pragmatic alternative if you want to avoid loading a second model. Same footprint and price as before (~18 GB vs ~17 GB).

### 3.4 Plan Orchestrator
*Long context + reliable tool calling + structured output for multi-agent coordination.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Llama 4 Scout | ~67 GB | 10M-token context; 17B active / 109B total MoE; natively multimodal; strong tool calling | `llama4:scout` |
| **Smaller pick** | **Qwen3.8-27B** | ~18 GB | 262K context (1M extensible); reliable structured output; strong tool fidelity | `qwen3.8:27b` |

**Rationale:** Llama 4 Scout (Meta, Apr 2026) still offers the longest locally-runnable context window at 10M tokens by a wide margin. Its 17B active-parameter MoE runs at ~22–26 tok/s via Ollama's MLX engine on M5 Max — no fresh M5 Max data point exists for Scout as of the most recent cycle (a gap in the community benchmark sources checked, not a regression). At ~67 GB Q4 it still fits alongside an 18 GB model with room to spare. The smaller pick moves from Qwen3.6-27B to its successor **Qwen3.8-27B** (see §3.1) — 262K native context extensible to 1M via YaRN, at the same ~18 GB footprint.

**New candidate considered, not selected:** **Qwen3.8-Flash-Next** (Alibaba, 125B main + 51B N-gram-embedding, ~6B active/token, released Aug 26, 2026, "experimental preview of the architecture that will underpin Qwen4") is architecturally the most interesting new orchestrator candidate this cycle — 262K–1M context at only ~6B active params per token — but its smallest Ollama tag resolves at 105 GB, leaving only ~15–20 GB of headroom in the 128 GB envelope versus Scout's ~67 GB (room for another 40+ GB model alongside it). No comparative agentic/tool-calling benchmark against Scout was found this cycle, and its own "experimental preview" framing argues for a field track record before promotion. **Not promoted — watch next cycle.** See §1 and §2.

**Also evaluated, not selected:** NVIDIA Nemotron 3.5 Lightning (30B-A3B MoE, released 2026-08-11, 1M-token context on its standard variant, claimed high throughput for "always-on agent execution layers") — no tool-calling-specific benchmark exists for it yet, and its SWE-bench Verified score (52.80%) suggests weaker code-comprehension than Scout or Qwen3.8-27B for orchestration tasks that involve reasoning about code. NVIDIA Nemotron 3 Super (Mar 11, 2026; 120B/12B-active, 1M context, ~60 GB at Q4) scores 60.47% SWE-bench Verified, below both current picks' orchestration-specific strengths.

**MCP tool-calling context:** No update to the MCP-Mark leaderboard (last dated Jun 2026: Kimi K2.7 Code leads at 0.811) has been found. Neither Scout nor Qwen3.8-27B appear in that evaluation.

### 3.5 LLM-as-Judge / Verifier
*Calibrated scoring against rubrics, pairwise preference evaluation, output verification.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | **Qwen3.8-27B** (thinking disabled) | ~18 GB | Strong instruction following; GPQA Diamond 89.2%; 262K ctx | `qwen3.8:27b` |
| **Smaller pick** | Gemma 4 12B Unified | ~7.6 GB | Fast; MMLU Pro 77.2%; low latency for high-volume scoring | `gemma4:12b` |

**Rationale:** For judging, **disable thinking mode** — reasoning traces inflate token cost and can introduce self-consistency bias in scores. The top pick moves from Qwen3.6-27B to its successor **Qwen3.8-27B** (see §3.1 for the full benchmark case) at the same ~18 GB footprint. Thinking is on by default (`reasoning_effort: xhigh`) and **must be explicitly disabled** via `enable_thinking: false` (there is no `/think` toggle on this model, unlike Qwen3.6-27B) — several HF discussion threads report the default effort level over-thinking even short prompts, which is a direct liability for the "fast, low-cost scoring" use case this role targets; test this explicitly before deploying. Inject a scored reference example at the top of the judge prompt to anchor the scoring scale. **Staleness flag, unchanged:** no fresher RewardBench 2 data has been found — last confirmed data remains the Jun 2025 submission behind RewardBench 2's ICLR 2026 paper. For high-stakes judging, frontier API models remain preferred; use local judges for cost-sensitive or privacy-constrained pipelines.

### 3.6 Document Understanding / Interpreter
*PDFs, OCR, table and equation extraction.*

| | Model | Q4 resident | Key capability | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Gemma 4 26B MoE (E26B-A4B) | ~19 GB | Natively multimodal; strong table/equation/diagram extraction; 89% AIME 2026 | `gemma4:26b` |
| **Smaller pick** | Mistral OCR 4.1 | *(container, not GGUF)* | Purpose-built OCR; structured Markdown/JSON output; 170 languages, bounding-box/block classification | *Not Ollama-loadable — see below* |

**Rationale:** Gemma 4's 26B MoE variant (Google, Apr 2026) understands images, text, tables, and LaTeX natively in a single forward pass. It handles complex PDF layouts, nested tables, and mixed-language documents reliably. No update has moved this pick. Mistral's OCR line reached **4.1** (GA per Mistral's own changelog, primary) this cycle — still an API/container product with no GGUF or Ollama tag found for either version; within the Ollama-only constraint of this document, Gemma 4 12B Unified (§3.1) remains the fallback for lighter-weight document tasks.

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
**Ornith-1.5-35B** (top code-implementer pick, 35B/~3B active, ~23 GB), **Laguna XS 2.1** (33B/3B active, ~20 GB), and **Qwen 3.5 30B-A3B** (30B/3B active, ~17 GB) all activate only ~3B parameters per token despite loading a much larger weight set. Community M5 Max data confirms the pattern holds up under direct measurement: 96.7–100.7 tok/s for Ornith-1.5-35B (MLX 8bit, llm-bench.io), 77–123 tok/s for its 1.0-generation predecessor (oMLX, 4bit), 87–107 tok/s for Laguna XS 2.1 — all firmly in "interactive agentic loop" territory despite 30B+ class quality. This remains the highest quality-per-GB pattern in the current Ollama library.

### 4.2 Distilled Reasoning Models
**DeepSeek-R1-Distill-Qwen-32B** transfers chain-of-thought behaviour from a 671B teacher into a 32B student (20 GB at Q4). It achieves MATH scores that beat many naive 70B models, at twice the speed and half the memory. For debugging and reasoning tasks, the distill is sufficient — you don't need to reach for a 70B.

### 4.3 Tiny Purpose-Trained Coding Models
**Ornith-1.5-9B** is a *dense* 9B model, trained around a self-improvement loop that generates its own agentic-RL tasks and scaffolds, that beats larger competitors at a fraction of their resident memory. Its 35B sibling's continued position as top pick across two generations (1.0 → 1.5) suggests the self-scaffolding/self-improvement training approach scales cleanly across sizes within the same family, not just as a one-off small-model trick.

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
| **Qwen3.8-27B** | `qwen3.8:27b` | ~18 GB | Generalist + judge + debugger/orchestrator smaller pick |
| DeepSeek-R1-Distill-Qwen-32B | `deepseek-r1:32b` | ~20 GB | Code debugger / reasoning |
| **Ornith-1.5-35B** | `ornith-1.5:35b` | ~23 GB | Code implementer (top pick, higher SWE-bench, multimodal) |
| Ornith-1.5-9B | `ornith-1.5:9b` | ~6.6 GB | Code implementer (smaller pick, multimodal) |
| Gemma 4 26B MoE | `gemma4:26b` | ~19 GB | Document understanding |
| Gemma 4 12B Unified | `gemma4:12b` | ~7.6 GB | Fast general + doc assistant |
| Gemma 4 E4B | `gemma4:e4b` | ~9.6 GB | Small vision / audio / video |
| *Ornith-1.0-35B (optional, alternative)* | `ornith:35b` | ~21 GB | *Code implementer alt — beaten by Ornith-1.5-35B, text-only, longer field track record* |
| *Ornith-1.0-9B (optional, alternative)* | `ornith:9b` | ~5.6 GB | *Code implementer alt — beaten by Ornith-1.5-9B, text-only, smallest footprint* |
| *Laguna XS 2.1 (optional, alternative)* | `laguna-xs-2.1:q4_K_M` | ~20 GB | *Code implementer alt — longer field track record, very competitive speed* |
| *Devstral Small 2 (optional)* | `devstral-small-2` | ~15 GB | *Code implementer alt — battle-tested tool-calling track record* |
| *Mistral Medium 3.5 (optional)* | `mistral-medium-3.5:128b` | ~80 GB | *Quality-critical one-shot coding, latency-tolerant only* |
| *Qwen3.6-27B (optional, superseded)* | `qwen3.6:27b` | ~17 GB | *No longer the recommended pick in any role as of 2026-09-01 — still Ollama-servable if a workflow depends on its `/think` prefix UX specifically* |

**Total core inventory size on disk (excluding optionals):** ~170.6 GB (Ornith
swapped from 1.0 to 1.5, +2 GB top pick / +1 GB smaller pick vs. the prior
cycle's figure). Ornith-1.0 pair adds ~26.6 GB, Laguna XS 2.1 adds ~20 GB,
Devstral Small 2 adds ~15 GB, Mistral Medium 3.5 adds ~80 GB if all optionals
are also pulled. Ollama evicts from RAM on demand.

### Recommended Concurrent Working Sets (each under ~100 GB)

| Set | Combined RAM | Use case |
|------|-------------|----------|
| Scout + Qwen3.8-27B | ~85 GB | Orchestrator + workhorse; long-context agentic sessions |
| Ornith-1.5-35B + Ornith-1.5-9B | ~29.6 GB | Same-family coding pair: escalate from smaller to top pick within one session without a family-behavior mismatch |
| Ornith-1.5-9B + DeepSeek-R1 32B | ~26.6 GB | Coding session: implement → debug loop |
| Qwen3.8-27B + Gemma 4 12B | ~25.6 GB | Lightweight dual-model chat + vision |
| Scout + Ornith-1.5-35B | ~90 GB | Orchestrated multi-file coding with the top code pick |
| Mistral Medium 3.5 + Qwen3.8-27B | ~98 GB | Quality-first coding + fast judge — fits but with limited KV-cache headroom |

Laguna XS 2.1 or Ornith-1.0-35B remain drop-in substitutes for Ornith-1.5-35B in any pairing above (same ~20-23 GB class) if a longer field track record is preferred over the higher benchmark score.

### What to Exclude from Local Use

| Model | Reason |
|-------|--------|
| Kimi K3 (2.8T total) | `:cloud` only; ~700+ GB even at 2-bit |
| DeepSeek V4-Flash / V4 Pro (284B / 1.6T total) | `:cloud` tags only on Ollama — no local variant exists |
| Ornith-1.5-397B (397B total) | `ornith-1.5:397b` confirmed at 242 GB (direct tag fetch) — roughly 2× the envelope |
| Llama 4 Maverick (400B total) | Q4 ≈ 200 GB — hard OOM |
| Kimi K2.7 Code (1T total) | Community GGUF exists but ~585 GB at Q4 |

---

## 6. Verification Checklist

```bash
# Ensure Ollama is up-to-date (v0.35.0 stable as of 2026-10-01; v0.40.0 experimental
# preview adds MLX-by-default on Apple Silicon, not yet stable)
ollama version

# --- Pull core models ---
ollama pull qwen3.8:27b               # ~18 GB  — generalist / judge / debugger + orchestrator fallback
ollama pull llama4:scout              # ~67 GB  — orchestrator + vision
ollama pull deepseek-r1:32b           # ~20 GB  — reasoning / debugging
ollama pull ornith-1.5:35b            # ~23 GB  — code implementation (top pick, multimodal)
ollama pull ornith-1.5:9b             # ~6.6 GB — code implementation (smaller pick, multimodal)
ollama pull gemma4:26b                # ~19 GB  — document understanding
ollama pull gemma4:12b                # ~7.6 GB — fast general
ollama pull gemma4:e4b                # ~9.6 GB — small vision/audio/video

# --- Optional alternatives ---
# ollama pull ornith:35b                # ~21 GB  — code implementation alt (beaten by Ornith-1.5-35B, text-only, longer track record)
# ollama pull ornith:9b                 # ~5.6 GB — code implementation alt (beaten by Ornith-1.5-9B, text-only)
# ollama pull laguna-xs-2.1:q4_K_M      # ~20 GB  — code implementation alt (beaten as top pick, still valid — longer track record)
# ollama pull devstral-small-2          # ~15 GB  — code implementation alt (battle-tested tool-calling)
# ollama pull mistral-medium-3.5:128b   # ~80 GB  — 77.6% SWE-bench but ~7 tok/s (confirmed via direct M5 Max measurement)
# ollama pull qwen3.6:27b               # ~17 GB  — superseded by qwen3.8:27b in every role as of 2026-09-01; keep only if a workflow depends on its `/think` prefix UX

# --- Verify resident memory after load ---
# (run `ollama ps` while the model is active)
# qwen3.8:27b           → 17–19 GB
# llama4:scout          → 65–70 GB
# deepseek-r1:32b       → 19–21 GB
# ornith-1.5:35b        → 22–24 GB
# ornith-1.5:9b         → 6–7 GB
# gemma4:26b            → 18–20 GB
# gemma4:12b            → 7–8 GB
# gemma4:e4b            → 9–10 GB
# ornith:35b (optional) → 20–22 GB
# ornith:9b (optional)  → 5–6 GB
# laguna-xs-2.1:q4_K_M  → 18–21 GB
# devstral-small-2      → 14–16 GB
# mistral-medium-3.5:128b → 78–82 GB
# qwen3.6:27b (optional) → 16–18 GB

# --- Smoke tests ---

# Generalist (Qwen3.8-27B) — thinking is on by default (reasoning_effort: xhigh)
ollama run qwen3.8:27b "Write a Python function to merge two sorted lists and return the merged result."

# Reasoning/debugger use (thinking on by default — no /think prefix on this model)
ollama run qwen3.8:27b "Identify the bug in this Python snippet: def fib(n): return fib(n-1) + fib(n-2)"
# Expected: a reasoning block before the final answer, noting missing base cases

# Judge use — thinking must be explicitly disabled via API param (enable_thinking: false),
# not available as a CLI flag on `ollama run`; verify via the API/SDK before deploying as judge.

# Long context orchestration (Llama 4 Scout)
ollama run llama4:scout "List 5 subtasks for building a REST API with auth, rate-limiting, and tests."

# Reasoning chain (DeepSeek-R1)
ollama run deepseek-r1:32b "What is the derivative of x^3 * sin(x)? Show all steps."
# Expected: full chain-of-thought before answer

# Agentic coding, top pick (Ornith-1.5-35B)
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
- [Ollama Releases — GitHub](https://github.com/ollama/ollama/releases) (primary — v0.32.6–v0.35.1 confirmed, re-fetched 2026-10-01: v0.34.3 Sep 19, v0.34.4 Sep 23, v0.35.0 Sep 28 stable/"Latest"; v0.35.1 Sep 29 and v0.40.0 Sep 25 both pre-release)
- [Ollama Library](https://ollama.com/library)
- [Release v0.33.0 — ollama/ollama GitHub](https://github.com/ollama/ollama/releases/tag/v0.33.0) (primary)
- [Ollama 0.34.0-rc1 — AI/TLDR](https://ai-tldr.dev/releases/ollama-0-34-0-rc1/) (secondary — ChatGPT Desktop integration, structured-output/tool-search changes, cross-checked against release notes)

### Ollama Library Pages Fetched Directly (primary)
- [ornith / tags](https://ollama.com/library/ornith/tags) — re-confirmed 2026-09-21: `ornith:35b` 21 GB, `ornith:9b` 5.6 GB, unchanged (1.0 generation — no in-place overwrite by 1.5, see below)
- [ornith (model page)](https://ollama.com/library/ornith)
- [ornith-1.5 / tags](https://ollama.com/library/ornith-1.5/tags) — *(new 2026-09-21)* fetched directly: `ornith-1.5:9b` 6.6 GB, `ornith-1.5:35b` 23 GB, `ornith-1.5:397b` 242 GB, all with text+image input, 256K context
- [ornith-1.5 (model page)](https://ollama.com/library/ornith-1.5) — *(new 2026-09-21)* confirms 333.3K downloads as of this cycle, updated ~1 month prior to fetch
- [qwen3.6 / tags](https://ollama.com/library/qwen3.6/tags) — confirmed `qwen3.6:27b` 17 GB
- [qwen3.8 / tags](https://ollama.com/library/qwen3.8/tags) — re-confirmed 2026-09-21: `qwen3.8:27b` 18 GB (`q4_K_M`), 30 GB (`q8_0`), unchanged, full quant/MLX tag list
- [qwen3.8-flash-next (model page)](https://ollama.com/library/qwen3.8-flash-next) and [/tags](https://ollama.com/library/qwen3.8-flash-next/tags) — confirmed smallest tags (`125b-a6b-nvfp4`, `125b-mlx`) resolve at 105 GB
- [glm-5.3-flash — Ollama](https://ollama.com/library/glm-5.3-flash) — confirmed `:cloud`-only tag, no local variant
- [laguna-xs-2.1 / tags](https://ollama.com/library/laguna-xs-2.1/tags) — confirmed `q4_K_M` 20 GB
- [llama4 / tags](https://ollama.com/library/llama4/tags) — re-confirmed 2026-09-21: `scout` 67 GB, unchanged
- [deepseek-r1 / tags](https://ollama.com/library/deepseek-r1/tags) — re-confirmed 2026-09-21: `32b` 20 GB, unchanged
- [gemma4 / tags](https://ollama.com/library/gemma4/tags) — re-confirmed 2026-09-21: `26b` 19 GB, `12b` 7.6 GB, `e4b` 9.6 GB, all unchanged
- [muse-glimmer / tags](https://ollama.com/library/muse-glimmer/tags) — confirmed `30b`/`q4_K_M` 18 GB
- [nemotron-3.5-lightning / tags](https://ollama.com/library/nemotron-3.5-lightning/tags) — confirmed `q4_K_M` 25 GB (arithmetic flag noted, §2)
- [kimi-k3](https://ollama.com/library/kimi-k3) — confirmed cloud-only tag

### Apple Silicon Benchmarks
- [oMLX Community M5 Max Benchmarks](https://omlx.ai/benchmarks) (primary/crowdsourced — individually-dated, directly-measured M5 Max figures for Ornith-1.0-9B/35B, Laguna XS 2.1, DeepSeek-R1-Distill-32B, Llama 3.1 8B, Llama 3.3 70B, Mistral Medium 3.5)
- [Qwen3.8-27B-MLX-oQ4e-mtp on M5 Max (40c) — oMLX Benchmark](https://omlx.ai/benchmarks/performance/lclma2k9) (primary/crowdsourced — long-context throughput, ~10.6 tok/s near the top of the 262K window; direct fetch returned HTTP 403/503, figures corroborated via cached search-result excerpts of the page's own content, not a secondary restatement)
- [Ornith-1.5-35B-A3B-MLX-8bit — llm-bench.io](https://llm-bench.io/models/ornith-1-5-35b-a3b-mlx-8bit) (secondary, crowdsourced — *(new 2026-09-21)* 96.7–100.7 tok/s avg on M5 Max, MLX 8bit, ~35.3 GB resident; not the Q4-class tag used for the resident-size figure elsewhere in this doc, flagged as such)
- [Apple Silicon LLM Benchmarks 2026 — LLMCheck](https://llmcheck.net/benchmarks)
- [PromptQuorum M5 Max Benchmarks](https://www.promptquorum.com/local-llms/m5-pro-max-llm-benchmarks-2026)
- [Presenc AI Local Benchmarks 2026](https://presenc.ai/research/local-llm-tokens-per-second-benchmarks-2026)
- [Contra Collective — KV Cache Quantization on Apple Silicon (M5 Max)](https://contracollective.com/blog/kv-cache-quantization-q8-vs-q4-m5-max-mlx-2026)
- [Apple ML Research — Exploring LLMs with MLX on M5](https://machinelearning.apple.com/research/exploring-llms-mlx-m5) (primary, base M5 only, not M5 Max)
- *Wale Akinfaderin, "Benchmarking Open-Weights LLMs on the MacBook Pro M5 Max" (Medium) — fetchable but paywalled beyond the intro; confirmed intro benchmarks the binned 32-core/460 GB/s M5 Max SKU, not this doc's 40-core/614 GB/s target — low value even if fully accessible.*

### Ornith-1.0 / Ornith-1.5
- [Ornith-1.0: Self-Scaffolding LLMs for Agentic Coding — DeepReinforce/Ornith Blog](https://ornith.ai/ornith_1_0.html) (primary — 75.6% SWE-bench Verified for 35B, methodology stated)
- [deepreinforce-ai/Ornith-1.0-35B — Hugging Face](https://huggingface.co/deepreinforce-ai/Ornith-1.0-35B) (primary — architecture and score confirmed)
- [deepreinforce-ai/Ornith-1.0-35B-GGUF — Hugging Face](https://huggingface.co/deepreinforce-ai/Ornith-1.0-35B-GGUF) (primary)
- [Ornith 1.0 35B MoE: Faster Than 9B, Better Than 31B — Ornith.site Blog](https://www.ornith.site/blog/ornith-1-0-35b-moe/) (secondary — independently confirms MoE architecture and 75.6% score)
- [Ornith-1.0-35B Benchmarks — BenchLM.ai](https://benchlm.ai/models/ornith-1-0-35b) (secondary — independently agrees on 75.6%)
- [deepreinforce-ai/Ornith-1.0-9B — Hugging Face](https://huggingface.co/deepreinforce-ai/Ornith-1.0-9B) (primary, 1.0-generation smaller pick)
- [Ornith-1.5: from self-scaffolding to self-improvement — Ornith Blog](https://ornith.ai/ornith_1_5.html) (primary — *(new 2026-09-21)* 397B/35B/9B benchmark table: SWE-bench Verified 86/79/70.6, Terminal-Bench 2.1 86.1–85.2/67.8–68.5/46.2–47, GPQA Diamond 92.8/89.2/86.4; MIT license, text+image input, 256K context, self-improvement RL loop)
- [ornith-ai/Ornith-1.5-9B-GGUF — Hugging Face](https://huggingface.co/ornith-ai/Ornith-1.5-9B-GGUF) (primary — *(new 2026-09-21)*)
- [ornith-ai/Ornith-1.5-35B-A3B-GGUF — Hugging Face](https://huggingface.co/ornith-ai/Ornith-1.5-35B-A3B-GGUF) (primary — *(new 2026-09-21)*)
- [ornith-ai/Ornith-1.5-397B-GGUF — Hugging Face](https://huggingface.co/ornith-ai/Ornith-1.5-397B-GGUF) (primary — *(new 2026-09-21)*)
- [Ornith releases Ornith-1.5 open weights under MIT — DataNorth.ai](https://datanorth.ai/news/ornith-releases-ornith-1-5) (secondary — *(new 2026-09-21)* independently confirms MIT license and Aug 19, 2026 release date)

### Muse Glimmer
- [Introducing Muse Glimmer — Meta AI Research Blog](https://research.meta.ai/blog/introducing-muse-glimmer-open-agentic-model) (primary)
- [Muse Glimmer from Meta Superintelligence Labs is now available — Ollama Blog](https://ollama.com/blog/muse-glimmer) (primary)
- [meta-models/Muse-Glimmer-30B — Hugging Face](https://huggingface.co/meta-models/Muse-Glimmer-30B) (primary — 76.0% SWE-bench Verified and full benchmark table)
- [Muse Glimmer 30B: Benchmarks, Hardware & How to Run — kingy.ai](https://kingy.ai/blog/muse-glimmer-30b-benchmarks-hardware-run/) (secondary — comparison table vs Gemma4-31B/Qwen3.6-27B, methodology caveat noted)

### NVIDIA Nemotron 3.5 Lightning
- [nvidia/NVIDIA-Nemotron-3.5-Lightning-30B-A3B-NVFP4 — Hugging Face](https://huggingface.co/nvidia/NVIDIA-Nemotron-3.5-Lightning-30B-A3B-NVFP4) (primary — 52.80% SWE-bench Verified, full benchmark table)
- [Introducing NVIDIA Nemotron 3.5 Lightning — Baseten Blog](https://www.baseten.co/blog/introducing-nemotron-35-lightning/) (secondary/partner — throughput claims)
- [NVIDIA Developer Blog — Nemotron 3.5 Lightning](https://developer.nvidia.com/blog/nvidia-nemotron-3-5-lightning-delivers-fast-accurate-specialized-task-execution-for-long-running-agents/) (primary)

### DeepSeek
- [DeepSeek V4-Flash coverage — Caixin Global](https://www.caixinglobal.com/2026-08-01/deepseek-releases-official-v4-flash-model-as-chinas-ai-race-intensifies-102470292.html) (secondary)
- [DeepSeek V4 Pro Benchmarks — BenchLM.ai](https://benchlm.ai/models/deepseek-v4-pro)
- [DeepSeek V4-Flash / tags — Ollama](https://ollama.com/library/deepseek-v4-flash/tags) (primary — re-checked 2026-08-15, confirmed `:cloud`-only tags, no local variant)
- [Ollama Ships DeepSeek V4 Pro — Collabnix](https://collabnix.com/ollama-ships-deepseek-v4-pro-a-1-6t-mixture-of-experts-model-with-1m-context-and-three-reasoning-modes/) (secondary — confirms `:cloud`-only Ollama listing, added 2026-08-15 check)
- [DeepSeek V4 Pro 0813: GA release — ofox.ai](https://ofox.ai/blog/deepseek-v4-pro-0813-price-weights-benchmarks-api-access-2026/) (secondary — GA date 2026-08-12)
- [Kimi K3 llama.cpp support pre-release analysis — ggml-org/llama.cpp Discussion #26041](https://github.com/ggml-org/llama.cpp/discussions/26041) (primary — PR #26185 status, re-checked 2026-08-15, still unmerged)

### Qwen Models
- [Qwen3.6-27B Tags — Ollama](https://ollama.com/library/qwen3.6/tags) (primary)
- [Qwen/Qwen3.6-27B — Hugging Face](https://huggingface.co/Qwen/Qwen3.6-27B) (primary — *(new 2026-09-01)* full benchmark table used for the Qwen3.8-27B comparison: SWE-bench Verified 77.2, SWE-bench Pro 53.5, Terminal-Bench 2.0 59.3, GPQA Diamond 87.8, LiveCodeBench v6 83.9; Apache 2.0; natively multimodal)
- [Qwen/Qwen3.8-27B — Hugging Face](https://huggingface.co/Qwen/Qwen3.8-27B) (primary — *(new 2026-09-01)* full benchmark table: Terminal Bench 2.1 73.0, SWE-bench Pro 61.7, GPQA Diamond 89.2, LiveCodeBench v6 90.3; 27B dense, hybrid Gated-DeltaNet/Attention; Apache 2.0; 262K ctx extensible to 1M; no SWE-bench Verified reported)
- [Qwen 3.8 27B vs Qwen 3.6 27B — DEV Community](https://dev.to/jamilxt/qwen-38-27b-vs-qwen-36-27b-same-architecture-4-months-apart-and-a-different-kind-of-upgrade-3280) (secondary — *(new 2026-09-01)* independently matches the SWE-bench Pro 61.7 vs 53.5 comparison)
- [Youssofal/Qwen3.8-27B-MTPLX-Optimized-Speed — Hugging Face](https://huggingface.co/Youssofal/Qwen3.8-27B-MTPLX-Optimized-Speed) (secondary/community — *(new 2026-09-01)* single-source M5 Max throughput, 63.3–65.2 tok/s, 4-bit single-stream, fans-max)
- [Qwen3.8-27B thinking/reasoning-effort behavior — HF Discussions #113, #97](https://huggingface.co/Qwen/Qwen3.8-27B/discussions/113) (primary — *(new 2026-09-01)* confirms `enable_thinking`/`reasoning_effort` params, default `xhigh`, over-thinking reports)
- [QwenLM/Qwen3.8-Flash-Next — GitHub](https://github.com/QwenLM/Qwen3.8-Flash-Next) and [Hugging Face](https://huggingface.co/Qwen/Qwen3.8-Flash-Next) (primary — *(new 2026-09-01)* 125B main + 51B N-gram-embedding, ~6B active, "experimental preview of the architecture that will underpin Qwen4," released 2026-08-26)
- [Qwen3.8-Max Open Weights Are Live — explainx.ai](https://www.explainx.ai/blog/qwen3-8-max-open-weights-live-hugging-face-august-2026) (secondary — confirms 2026-08-12 HF release of the 2.4T base model)
- [ollama.com/library/qwen3.8-max](https://ollama.com/library/qwen3.8-max) — re-confirmed 404, no Ollama tag exists (primary)

### Tencent Hy4 / MiniMax M3 (this cycle's checks)
- ["Five open weight releases in nine days" — Requesty](https://www.requesty.ai/blog/open-weight-frontier-august-2026-glm-qwen-hy4) (secondary — Tencent Hy4 Preview 770B/49B active, GLM-5.3-Flash 320B/18B active, Qwen3.8-Flash-Next release-date cluster, MiniMax M3 Aug 24 date claim — flagged as conflicting with this doc's prior Jun 1 date, see §2)

### Qwen 4 preview / Apsara Conference (new 2026-10-01)
- [Alibaba unveils Qwen 4 and a 10 trillion parameter roadmap — pasqualepillitteri.it](https://pasqualepillitteri.it/en/news/17552/qwen-4-alibaba-apsara-en) (secondary — Apsara Conference reporting, cites Qwen project lead Liu Dayiheng)
- [Qwen 4 Max announced at Apsara 2026: the four tiers — orcarouter.ai](https://www.orcarouter.ai/blog/qwen-4-max-lineup-announced-apsara-2026) (secondary — independently corroborates tier names and "no spec sheet/no benchmarks" status)

### DeepSeek-V4.1-Flash / MiniMax H3 Max / M3.1-Flash-Preview (new 2026-10-01)
- [DeepSeek-V4.1-Flash — Ollama](https://ollama.com/library/deepseek-v4.1-flash) (primary — `:cloud` tag confirmed, 552B backbone/8B-16B active architecture)
- [M3.1-Flash-Preview: Coding & Research — MiniMax](https://agent.minimax.io/tools/m3-1-flash-preview) (primary — MiniMax's own docs, 428B/~23B active, Token Plan/MiniMax Code availability only)
- [MiniMax released M3.1-Flash-Preview — DataNorth AI](https://datanorth.ai/news/minimax-releases-m3-1-flash-preview) (secondary)

### IBM Granite 4.2 (new to library, 2026-10-01)
- [Granite4.2 — Ollama](https://ollama.com/library/granite4.2) (primary — sizes, architecture, no benchmark data)

### Ollama v0.34.3–v0.35.1 / v0.40.0 preview (new 2026-10-01)
- [Ollama Releases — GitHub](https://github.com/ollama/ollama/releases) (primary)

### GLM-5.3 (full flagship) / MiniMax H3 / OpenSWE-72B (new 2026-09-11)
- [GLM-5.3 Weights Are Out—But Running Them Takes Eight GPUs — kingy.ai](https://kingy.ai/blog/glm-5-3-specs-benchmarks-api-how-to-use/) (secondary — parameter count, BF16/FP8 size figures)
- [GLM-5.3 Open Weights: Z.ai's 744B Model Released — techjacksolutions.com](https://techjacksolutions.com/ai-brief/zai-glm-53-open-weights-744b-safety-hold/) (secondary)
- [zai-org/GLM-5 — Hugging Face](https://huggingface.co/zai-org/GLM-5) (primary — weights repo)
- ["Is it a coincidence that both MiniMax and Z.ai are releasing frontier open weights..." — Hacker News](https://news.ycombinator.com/item?id=48518889) (secondary — MiniMax H3 shipping context)
- [GAIR/OpenSWE-72B — Hugging Face](https://huggingface.co/GAIR/OpenSWE-32B) and [GAIR-NLP/OpenSWE — GitHub](https://github.com/GAIR-NLP/OpenSWE) (primary — 66.0%/68.0% SWE-bench Verified, AGPL-3.0, training methodology)

### Muse Glimmer vs. Qwen3.8-27B re-run (new 2026-09-11)
- [Muse Glimmer-30B vs Qwen3.8-27B: Benchmarks, Pricing & Which Is Better in 2026 — LLM Stats](https://llm-stats.com/models/compare/muse-glimmer-30b-vs-qwen3.8-27b) (secondary — aggregates both models' own published benchmark numbers; 8/8 head-to-head comparisons won by Qwen3.8-27B)

### Ollama v0.33.3 / v0.34.0-rc1 (new 2026-09-11)
- [Ollama Release Notes & Changelog — releases.sh](https://releases.sh/ollama) (secondary, cross-checked against GitHub releases)

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

### GLM-5.2 / Z.ai, MiniMax M3, Mistral, Devstral, Cohere
- [Run GLM-5.2 Locally — GLM5.app](https://glm5.app/blog/how-to-run-glm-5-2-locally)
- [MiniMax M3 Open Weights — Nerova](https://nerova.ai/news/minimax-m3-open-weight-agent-builders-june-2026)
- [mistral-medium-3.5:128b — Ollama](https://ollama.com/library/mistral-medium-3.5) (primary)
- [Mistral Docs Changelog](https://docs.mistral.ai/resources/changelogs) (primary — *(new 2026-09-21)* Mistral OCR 4.1 reached GA; still no GGUF/Ollama tag)
- [Devstral Small 2 — Ollama](https://ollama.com/library/devstral-small-2) (primary)
- [Meet North Mini Code — MarkTechPost](https://www.marktechpost.com/2026/06/11/meet-north-mini-code-coheres-30b-open-weight-mixture-of-experts-model-with-3b-active-parameters-for-agentic-coding/)

### Reference
- [Ollama VRAM Requirements 2026 — Local AI Master](https://localaimaster.com/blog/ollama-model-ram-vram-table)
- [GGUF Quantization Guide — Easton Blog](https://eastondev.com/blog/en/posts/ai/20260422-ollama-gguf-quantization/)
