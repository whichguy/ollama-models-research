# Ollama Models Research — 2026-08 Refresh

**Generated:** 2026-08-01
**Hardware target:** Apple M5 Max · 128 GB unified memory · 614 GB/s bandwidth
**Research window:** Last 60 days (2026-06-02 → 2026-08-01)

> **Diffs from previous refresh (`ollama-models-2026-07.md`, generated 2026-07-21):**
>
> **One role pick changed.** The code-implementer **smaller pick** moves from
> **Devstral Small 2** to **Ornith-1.0-9B** — a dense 9B model released
> 2026-06-25 that was in scope for the prior window but not surfaced until
> this cycle's sweep. It scores higher on SWE-bench Verified (69.4% vs
> 68.0%) at roughly a third of the resident memory (5.6 GB vs 15 GB), which
> also means materially faster generation in agentic loops. Devstral Small 2
> remains a fully valid alternative — its edge is a longer field track record
> in production agentic scaffolds (OpenHands, SWE-agent) that Ornith-1.0-9B,
> as a brand-new model, doesn't yet have. See §3.2.
>
> **No other role's Top or smaller pick changed.** Generalist, code-debugger,
> plan-orchestrator, judge, document-understanding, and vision all keep their
> prior picks.
>
> **Corrections to existing figures (no pick impact):**
> 1. **Mistral Medium 3.5 resident size corrected: ~64 GB → 80 GB** at
>    `q4_K_M` (confirmed directly on `ollama.com/library/mistral-medium-3.5`).
>    The prior ~64 GB figure used a naive 4.0-bits/weight assumption; the
>    model's actual q4_K_M average is closer to 5.0 bits/weight
>    (128B × 5/8 ≈ 80 GB, matching the listed size). Because the model is
>    dense and memory-bandwidth-bound, the corrected size also lowers its
>    tok/s ceiling: 614 GB/s ÷ 80 GB ≈ 7.7 tok/s theoretical max across
>    *any* runtime on this hardware, versus the ~9.6 tok/s ceiling the old
>    64 GB figure implied. The prior doc's "~9–12 / ~14–16 / ~18" tok/s
>    estimates for this model were arithmetically impossible at 80 GB and
>    have been revised down — see §1.
> 2. **Ollama stable version: v0.32.1 → v0.32.5** (2026-07-27). Three point
>    releases shipped in the window: v0.32.3 (Laguna 2.1 tool-calling +
>    Metal fix, CUDA on Windows ARM64), v0.32.4 (**Laguna gains native MLX
>    support on Apple GPUs**; Qwen3 MoE decode ~4–9% faster on M5 Max per
>    Ollama's own release notes — the best-sourced M5-Max-specific
>    performance datapoint found this cycle), v0.32.5 (fixed an MLX Metal
>    bug degrading output quality for NVFP4 models, "particularly Laguna").
>    Upgrade recommended, especially for Laguna XS 2.1 users who were
>    previously limited to the llama.cpp/Metal path.
> 3. **Laguna XS 2.1's 70.9% SWE-bench Verified score re-confirmed** directly
>    against Poolside's own Hugging Face model card (primary source). A
>    secondary aggregator (benchlm.ai) was found listing a *different*
>    entry, "Laguna XS.2" at 69.9% — that is the predecessor model, not
>    XS 2.1; the two are easy to conflate by name and the aggregator's
>    listing does not represent an error in the 70.9% figure already in this
>    doc.
>
> **Considered and rejected:**
> - **Ornith-1.0-35B** (deepreinforce-ai, MoE, ~3B active, `ornith:35b`,
>   21 GB Q4 — official Ollama tag, confirmed) — no SWE-bench Verified score
>   was found on any source checked this cycle (only Terminal-Bench 2.1:
>   64.4, vs Qwen3.5-397B's 53.5, secondary/MarkTechPost). **[unverified]**
>   for this document's purposes: fails the benchmark-gate requirement of a
>   primary source or two agreeing secondaries on the *headline* benchmark
>   used throughout this doc. Worth re-checking next cycle; not promoted to
>   any role pick.
> - **Kimi K3** — open weights actually published 2026-07-26 on Hugging Face
>   (a day ahead of the "Jul 27" promise noted last cycle). Still no local
>   GGUF/llama.cpp/MLX support as of 2026-08-01: Ollama offers only
>   `kimi-k3:cloud` (confirmed on `ollama.com/library/kimi-k3`), and
>   llama.cpp support is an open, unmerged PR per secondary sources. Remains
>   cloud-only — see §1.
> - **Qwen3.7 Flash** (2026-07-27) and **Qwen3.8-Max** (previewed 2026-07-19,
>   2.4T params) — both API/cloud-priced only, no open weights released,
>   no Ollama tag. Not locally-runnable candidates at all; noted for
>   completeness only.
> - **Mistral OCR 4** (2026-06-23) — self-hostable document-AI container
>   (bounding boxes, block classification, 170 languages), but ships as a
>   dedicated container/API product, not a GGUF — no evidence found of an
>   Ollama-loadable form. Mentioned in §3.6/§4 as a specialized option
>   outside the Ollama library, not promoted to a pick.
>
> **Nothing previously recommended was beaten or deprecated** outright —
> Devstral Small 2 is displaced from "smaller pick" but remains a fully
> supported, valid alternative (see §3.2).

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

Numbers are approximate; see measurement note below.

| Model | Architecture | Quant | tok/s Ollama | tok/s Ollama MLX | tok/s mlx_lm native |
|-------|-------------|-------|:------------:|:----------------:|:-------------------:|
| Llama 3.1 8B | Dense 8B | Q4_K_M | ~82 | ~138 | ~230 |
| Ornith-1.0-9B | Dense 9B | Q4_K_M | ~75 (extrapolated†) | ~120 (extrapolated†) | ~190 (extrapolated†) |
| Qwen3.6-27B | Dense 27B | Q4_K_M | ~45 | ~70 | ~85 |
| Qwen 3.5 30B-A3B | MoE 30B/3B active | Q4_K_M | ~45 | ~55 | ~68 |
| Laguna XS 2.1 | MoE 33B/3B active | Q4_K_M | ~43 | ~53‡ | ~65 |
| Ornith-1.0-35B | MoE 35B/~3B active | Q4_K_M | ~42 (extrapolated†) | ~52 (extrapolated†) | ~64 (extrapolated†) |
| DeepSeek-R1-Distill 32B | Dense 32B | Q4_K_M | ~27 | ~45 | ~60 |
| Llama 4 Scout | MoE 109B/17B active | Q4_K_M | ~22 | ~26 | ~50 |
| Llama 3.3 70B | Dense 70B | Q4_K_M | ~12–18 | ~15–22 | ~28 |
| Mistral Medium 3.5 | Dense 128B | Q4_K_M (80 GB)§ | ~6 | ~7 | ~7–8 |

† No direct M5 Max measurement found this cycle for Ornith-1.0-9B/35B (both
released too recently to appear in the Apple Silicon benchmark blogs this
doc otherwise relies on). Extrapolated by analogy to similarly-sized/shaped
rows already in this table (dense-9B ≈ Llama 3.1 8B class; MoE-35B/3B ≈
Laguna XS 2.1 class) — treat as directional, not measured.

‡ As of Ollama v0.32.4 (2026-07-25), Laguna gained a native MLX path on
Apple GPUs; prior to that it ran the "Ollama MLX" column via a Metal/
llama.cpp fallback. The ~53 tok/s figure predates this change and has not
been re-measured on the new MLX path — plausibly higher post-upgrade, not
yet confirmed.

§ **Bandwidth-ceiling-derived, not measured.** Mistral Medium 3.5's
resident size was corrected this cycle from ~64 GB to the confirmed 80 GB
(see diffs callout above). At 80 GB, 614 GB/s ÷ 80 GB ≈ **7.7 tok/s is the
hard physical ceiling** for any runtime on this hardware running this dense
model single-stream — the prior doc's 9–12/14–16/18 tok/s figures exceeded
that ceiling and have been replaced with arithmetic estimates capped just
under it. Runtime differences (MLX vs llama.cpp) matter far less here than
elsewhere in this table because the model is fully bandwidth-bound at this
size; there's no overhead headroom left for MLX to reclaim.

**Measurement note:** "Ollama" column = Ollama v0.32.x via llama.cpp
backend. "Ollama MLX" = Ollama's native MLX path (available since v0.30.0,
extended to Laguna in v0.32.4). "mlx_lm native" = via `mlx_lm.generate`
directly, bypassing Ollama overhead. LLMCheck (July 2026) is the primary
source for the first two columns on previously-covered models; mlx_lm
native figures from PromptQuorum and Presenc AI. No primary Apple source
for M5 Max tok/s was found this cycle either (a Medium deep-dive that might
have qualified, Wale Akinfaderin's "Benchmarking Open-Weights LLMs on the
MacBook Pro M5 Max," returned HTTP 503 on fetch and could not be reviewed).
Context length, KV-cache fill, and prompt length all materially affect
tok/s — treat these as interactive-session estimates at moderate context
(2K–8K tokens), not batch throughput.

Sources: [LLMCheck Apple Silicon Benchmarks](https://llmcheck.net/benchmarks) · [LLMCheck M5 Max Guide](https://llmcheck.net/blog/apple-silicon-m5-max-local-ai-guide/) · [PromptQuorum M5 Max Benchmarks](https://www.promptquorum.com/local-llms/m5-pro-max-llm-benchmarks-2026) · [Presenc AI Local Benchmarks 2026](https://presenc.ai/research/local-llm-tokens-per-second-benchmarks-2026) · [Ollama v0.32.4 release notes](https://github.com/ollama/ollama/releases) (primary, M5-Max-specific Qwen3 MoE speedup claim)

### Runtime Notes

- **MLX vs llama.cpp:** Ollama's bundled MLX engine (available since v0.30.0) gives 20–50% uplift over its llama.cpp path on Apple Silicon for models that aren't already bandwidth-saturated. Using `mlx_lm` directly gives another 30–50% on top of that. For speed-critical workflows, run `mlx_lm` directly; for convenience and tool/API compatibility, Ollama's MLX path is the right default. **Exception:** for large dense models near the bandwidth ceiling (e.g. Mistral Medium 3.5 at 80 GB), the uplift compresses toward zero — see §1 tok/s table footnote.
- **Ollama v0.32.3–v0.32.5 (new this refresh):** v0.32.4 (2026-07-25) is the notable release — it brings Laguna XS 2.1 onto Ollama's native MLX path on Apple GPUs (previously Metal/llama.cpp only) and ships a primary-sourced, M5-Max-specific ~4–9% Qwen3 MoE decode speedup. v0.32.5 (2026-07-27) is a correctness fix for an MLX Metal bug that degraded output quality specifically for Laguna on NVFP4. Net effect: Laguna XS 2.1 users should upgrade to at least v0.32.5 before benchmarking or relying on quality-sensitive output.
- **MoE-A3B behaviour:** MoE models activate only ~3B parameters per token despite loading a much larger full weight set. Laguna XS 2.1 (33B total, ~20 GB at Q4_K_M), Qwen 3.5 30B-A3B (30B total, ~17 GB), and the new Ornith-1.0-35B (35B total, ~21 GB) all deliver 30B-class quality while running at similar tok/s to a 7–8B dense model. This remains the highest quality-per-GB pattern in the current Ollama library.
- **KV-cache headroom:** At 128 GB, reserve ~10–18 GB for macOS + long-context KV-cache. Practical ceiling for a single loaded model is ~110 GB — comfortably above even Llama 4 Scout's ~67 GB footprint, and now tighter but still workable for Mistral Medium 3.5's corrected 80 GB footprint.
- **M5 Max vs M4 Max:** ~28% higher tok/s across all model sizes (614 GB/s vs 546 GB/s bandwidth, improved GPU, redesigned Neural Engine). No update to this figure found this cycle.

### What Exceeds 128 GB (Cloud-Only or Borderline)

| Model | Why it won't fit |
|-------|------------------|
| **Kimi K3** (Moonshot, 2.8T/50.4B active MoE) | Open weights published Jul 26, 2026, but Ollama offers only `kimi-k3:cloud`; llama.cpp support is an open PR, not merged. Native weights ~1.4 TB; even 2-bit quants ≈ 700+ GB. **Cloud-only.** |
| **Ornith-1.0-397B** (deepreinforce-ai, 397B/~3B active MoE) | Unsloth GGUF exists. Q4_K_M ≈ 200 GB (OOM); extreme Q2 ≈ 100 GB fits numerically but quality untested at that quant. **Effectively cloud-only.** Note: the same family's 9B and 35B members *do* fit comfortably — see §3.2 and diffs callout. |
| **GLM-5.2** (Z.ai, 744B/40B active MoE) | 62.1% SWE-bench Pro, 91.2% GPQA Diamond; smallest usable GGUF ~217 GB; Ollama only offers `:cloud`. |
| **MiniMax M3** (MiniMax, ~428B/23B active MoE) | 80.5% SWE-bench Verified; smallest local quant ~143 GB; Ollama only offers `:cloud`. |
| Llama 4 Maverick (400B total) | Q4 ≈ 200 GB — hard OOM |
| Kimi K2.7 Code (1T total) | Community GGUF builds exist but ~585 GB at Q4; cloud-only for this hardware |
| **DeepSeek-V4-Pro-Max** (1.6T/49B active MoE) | 80.6% SWE-bench Verified — still one of the highest open-weight scores found — but ~800 GB minimum at Q4; a circulating "~50 GB" claim is arithmetically impossible for 1.6T params and was rejected last cycle |
| DeepSeek-V3 full (671B MoE, 37B active) | Q4 ≈ 170 GB — borderline, likely OOM with KV-cache |
| Any 700B+ total-parameter model | Exceeds envelope at any practical quant |

---

## 2. Current Model Landscape (Last 60 Days)

### Models that moved SOTA (Jun 2 – Aug 1, 2026)

| Model | Family | Released | Why it matters |
|-------|--------|----------|-----------------|
| **Ornith-1.0 family** | deepreinforce-ai (9B dense, 35B MoE/~3B active, 397B MoE/~3B active) | Jun 25, 2026 | Self-scaffolding RL training (the model learns its own agentic-RL harnesses rather than using human-designed ones). The **9B member is new code-implementer smaller pick this cycle** (69.4% SWE-bench Verified at 5.6 GB Q4 — highest score-per-GB found this cycle). 35B MoE has strong Terminal-Bench 2.1 (64.4) but **[unverified]** SWE-bench Verified. 397B remains cloud-only. MIT license. |
| **Kimi K3** | Moonshot AI (MoE 2.8T/50.4B active) | Jul 16, 2026 (announced); open weights Jul 26, 2026 | Largest open-weight model with published weights; 1M context; native vision/video; 93.5% GPQA Diamond, 88.3% Terminal-Bench 2.1, 76.8% SWE-bench Verified (KimiCode harness). Still no local GGUF/Ollama/MLX support (`kimi-k3:cloud` only) as of 2026-08-01. **Cloud-only — see §1.** |
| **Laguna XS 2.1** | Poolside (MoE 33B/3B active) | Jul 2, 2026 | 70.9% SWE-bench Verified — re-confirmed directly against Poolside's own HF model card this cycle; highest-scoring *locally-runnable* Ollama model per GB in its size tier; 256K ctx; Apache-compatible license. Gained native MLX support on Apple GPUs via Ollama v0.32.4 (Jul 25). |
| **Mistral Medium 3.5** | Mistral AI (dense 128B) | Apr 29, 2026 | 77.6% SWE-bench Verified (higher than Laguna XS 2.1); Ollama tag exists (`mistral-medium-3.5:128b`). **Resident size corrected this cycle: 80 GB, not 64 GB** (still fits in 128 GB but leaves less headroom); bandwidth-ceiling tok/s ≈7.7 max — too slow for agentic coding loops. Viable for quality-critical one-shot tasks. See §3.2. |
| **Ornith-1.0-397B** | deepreinforce-ai (MoE 397B/~3B active) | Jun 25, 2026 | Open-weight SWE-bench Verified leader at 82.4% (not re-verified this cycle, carried forward). Q4 ≈ 200 GB. **Effectively cloud-only on 128 GB — see §1.** |
| **GLM-5.2** | Z.ai (MoE 744B/40B active) | Jun 13, 2026 | 62.1% SWE-bench Pro, 91.2% GPQA Diamond, MIT license — strongest all-round open-weight cited this window; doesn't fit 128 GB (~217 GB+); Ollama only offers `:cloud`. **Cloud-only — see §1.** |
| **MiniMax M3** | MiniMax (MoE ~428B/23B active) | Jun 1, 2026 | 80.5% SWE-bench Verified; 1M ctx; native multimodal — smallest local quant ~143 GB; Ollama only offers `:cloud`. **Cloud-only — see §1.** |
| **Kimi K2.7 Code** | Moonshot AI (MoE 1T/32B active) | Jun 2026 | Highest MCP-Mark score of any model (0.811) — strongest for MCP tool-calling; community GGUF exists but cloud-only at ~585 GB |
| **Cohere North Mini Code 1.0** | Cohere (MoE 30B/3B active) | Jun 11, 2026 | 67.6% SWE-bench Verified; Apache 2.0; real Ollama tag (`north-mini-code-1.0`) — same size class as Laguna XS 2.1 and now also Ornith-1.0-9B, scores lower than both |
| **DeepSeek-V4-Pro-Max** | DeepSeek (MoE 1.6T/49B active) | ~Apr 23, 2026 | 80.6% SWE-bench Verified; ~800 GB at Q4; **cloud-only** |
| **Mistral OCR 4** | Mistral AI (self-hostable document-AI container) | Jun 23, 2026 | Structure-aware OCR (bounding boxes, block classification, 170 languages); ships as a dedicated container/API product — no evidence of an Ollama-loadable GGUF form found. Relevant context for §3.6, not a pick. |
| **Ollama v0.32.5** | Platform release | Jul 27, 2026 | Current stable. See diffs callout and Runtime Notes for v0.32.3–v0.32.5 changes (Laguna MLX support, Qwen3 MoE M5-Max speedup, NVFP4 quality fix). |

### Models found this cycle but not locally relevant

| Model | Family | Released | Why noted |
|-------|--------|----------|-----------|
| **Qwen3.7 Flash** | Alibaba/Qwen | Jul 27, 2026 | API/cloud-priced only ($0.03/$0.13 per M tokens); no open weights, no Ollama tag. |
| **Qwen3.8-Max** | Alibaba/Qwen | Previewed Jul 19, 2026 | 2.4T params, API-only preview; weights not released. |

### Notable models evaluated just outside the 60-day window

| Model | Family | Released | Why noted |
|-------|--------|----------|-----------|
| **NVIDIA Nemotron 3 Super** | NVIDIA (hybrid Mamba-Transformer, 120B/12B active) | Mar 11, 2026 | ~60 GB at Q4 (fits comfortably), 1M context, 2.2× GPT-OSS-120B throughput — but 60.47% SWE-bench Verified, below all current role picks. Not selected. |

### SWE-bench Verified Snapshot (Jul 31, 2026)

For context on where role picks sit in the broader leaderboard. Frontier
(non-open-weight, API-only) scores are omitted from precise citation this
cycle — different trackers disagreed on the exact Claude figures (93.9–96%
range depending on source) and none of it changes a local pick, so per this
doc's disagreement-reporting rule we note the spread rather than pick a
number.

| Model | Score | Locally runnable? |
|-------|-------|--------------------|
| Ornith-1.0-397B | 82.4% | No (200 GB Q4) |
| DeepSeek V4 Pro Max | 80.6% | No (800 GB Q4) |
| MiniMax M3 | 80.5% | No (143 GB min) |
| Qwen3.7 Max | 80.4% | No (cloud/API only) |
| Kimi K2.6 | 80.2% | No |
| Qwen3.6 Plus | 78.8% | No (cloud/API only) |
| Qwen3.7 Plus | 77.7% | No (cloud/API only) |
| GLM-5 | 77.8% | No |
| Mistral Medium 3.5 | 77.6% | **Yes (80 GB Q4 — corrected this cycle; slow)** |
| Qwen3.6-27B | 77.2% | **Yes (~17 GB Q4)** |
| **Laguna XS 2.1** | **70.9%** | **Yes (~20 GB Q4) — top local coding pick** |
| **Ornith-1.0-9B** | **69.4%** | **Yes (~5.6 GB Q4) — new smaller coding pick** |
| Devstral Small 2 | 68.0% | **Yes (~15 GB Q4) — still a valid alternative** |
| Cohere North Mini Code 1.0 | 67.6% | Yes (~17 GB Q4) |

Source: [BenchLM.ai SWE-bench Verified Leaderboard](https://benchlm.ai/benchmarks/sweVerified) (secondary, snapshot dated 2026-07-31); Laguna XS 2.1 and Ornith-1.0-9B scores cross-checked against primary/independent-secondary sources per §3.2.

---

## 3. Recommendations by Role

### 3.1 Generalist Agentic Default
*One model that does most jobs adequately.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Qwen3.6-27B | ~17 GB | SWE-bench Verified 77.2%; Terminal-Bench 2.0 59.3%; 256K ctx | `qwen3.6:27b` |
| **Smaller pick** | Gemma 4 12B Unified | ~8 GB | MMLU Pro 77.2%; natively multimodal (text + image) | `gemma4:12b` |

**Rationale:** Qwen3.6-27B (Alibaba, Apr 22, 2026) is a 27B dense model with a built-in dual-mode thinking toggle: `/think` on for deep reasoning, off for fast chat. At 77.2% SWE-bench Verified it sits well within range of the frontier API models while running entirely locally. The 256K context window handles most agentic sessions without truncation. Gemma 4 12B Unified is the right answer when memory is at a premium or multimodal input is needed at low cost. No new challenger displaced this pick this cycle.

**Sizing note:** ~16.8 GB at Q4_K_M; ~19.5 GB at Q5_K_M; ~28.6 GB at Q8_0. Q5_K_M is recommended if the extra 3 GB is available — marginal quality uplift compounds in agentic loops.

### 3.2 Code Implementer
*Writes code, multi-file edits, agentic coding loops.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Laguna XS 2.1 | ~20 GB | SWE-bench Verified 70.9%; SWE-bench Multilingual 63.1%; 256K ctx | `laguna-xs-2.1:q4_K_M` |
| **Smaller pick** | **Ornith-1.0-9B** *(changed this cycle)* | ~5.6 GB | SWE-bench Verified 69.4%; Terminal-Bench 2.1 43.1; 256K ctx | `ornith:9b` |

**Rationale:** Laguna XS 2.1 (Poolside, Jul 2, 2026) is a 33B/3B-active MoE scoring 70.9% on SWE-bench Verified — the highest of any Ollama-available model in its memory tier, re-confirmed this cycle directly against Poolside's own HF model card. MoE sparsity keeps generation at ~43–53 tok/s (Ollama/MLX on M5 Max — now via a native MLX path as of Ollama v0.32.4), fast enough for interactive agentic loops. The 256K context window handles large multi-file codebases.

**Smaller pick changed this cycle:** Ornith-1.0-9B (deepreinforce-ai, Jun 25, 2026 — released before the prior 60-day window closed but not surfaced until this cycle's sweep) is a dense 9B model trained with a self-scaffolding RL method, meaning it learns its own agentic-loop harnesses rather than relying on hand-built ones — directly relevant to this role. It edges out the prior smaller pick, Devstral Small 2, on every axis checked this cycle: SWE-bench Verified (69.4% vs 68.0%), resident size (5.6 GB vs 15 GB, confirmed on `ollama.com/library/ornith/tags`), and — because it's roughly a third the size on the same dense architecture pattern — likely generation speed. **Tradeoff to weigh:** Devstral Small 2 has a materially longer production track record in agentic tool-calling scaffolds (OpenHands, SWE-agent); Ornith-1.0-9B is brand new and its tool-calling reliability in extended agentic loops has not been independently validated by any source checked this cycle. Teams that have already standardized on Devstral's tool-calling behavior should not feel obligated to switch; teams starting fresh or prioritizing footprint should default to Ornith-1.0-9B and fall back to Devstral if tool-calling issues surface.

**Also evaluated, not selected:**
- *Devstral Small 2* (Mistral, 24B dense; 68.0% SWE-bench; `devstral-small-2`, ~15 GB Q4, Apache 2.0): displaced from smaller pick this cycle — see above. Still fully valid, especially where tool-calling track record matters more than footprint.
- *Mistral Medium 3.5* (Apr 2026; 128B dense; 77.6% SWE-bench Verified — higher than Laguna; `mistral-medium-3.5:128b`, **80 GB Q4 — corrected this cycle**, Modified MIT): fits in 128 GB, but at the corrected size its bandwidth-bound tok/s ceiling is ≈7.7 tok/s on this hardware (down from the ~9–12 tok/s previously estimated against the wrong 64 GB figure). For a 1K-token agentic-loop completion, that's roughly **130–170 seconds** versus Laguna XS 2.1's ~19–23 seconds — the latency penalty dominates even more clearly than previously stated. **Prefer Mistral Medium 3.5 only for quality-critical, non-latency-sensitive one-shot tasks.**
- *Ornith-1.0-35B* (MoE, ~3B active, `ornith:35b`, 21 GB Q4): no SWE-bench Verified score found this cycle — **[unverified]**, not promoted. Worth re-checking next cycle.
- *Cohere North Mini Code 1.0* (Jun 11, 2026; 30B/3B-active MoE; 67.6% SWE-bench; `north-mini-code-1.0`): same size class as Laguna XS 2.1, scores lower than both current picks.
- *Ornith-1.0-397B* (82.4% SWE-bench Verified) and *DeepSeek-V4-Pro-Max* (80.6%), *MiniMax M3* (80.5%), *Kimi K3* (76.8%) top the global leaderboard but none fit 128 GB at practical quants — see §1. Laguna XS 2.1 remains the best *locally-runnable* top pick.

### 3.3 Code Debugger
*Reasoning / chain-of-thought, root-cause analysis, math-heavy debugging.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | DeepSeek-R1-Distill-Qwen-32B | ~20 GB | MMLU 83%; MATH 72% (highest of any sub-40 GB local model) | `deepseek-r1:32b` |
| **Smaller pick** | Qwen3.6-27B (thinking mode on) | ~17 GB | SWE-bench 77.2%; switchable CoT; same model as §3.1 | `qwen3.6:27b` |

**Rationale:** DeepSeek-R1-Distill-Qwen-32B provides always-on chain-of-thought reasoning and holds the highest MATH benchmark score of any locally-runnable model under 40 GB. Distilled from the 671B R1 teacher; generates full reasoning traces before answering. Use it for debugging sessions requiring step-by-step logical trace through a logic fault or complex algorithm. Qwen3.6-27B with `/think` is the pragmatic alternative if you want to avoid loading a second model. No challenger found this cycle.

### 3.4 Plan Orchestrator
*Long context + reliable tool calling + structured output for multi-agent coordination.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Llama 4 Scout | ~67 GB | 10M-token context; 17B active / 109B total MoE; natively multimodal; strong tool calling | `llama4:scout` |
| **Smaller pick** | Qwen3.6-27B | ~17 GB | 256K context; reliable structured output; strong tool fidelity | `qwen3.6:27b` |

**Rationale:** Llama 4 Scout (Meta, Apr 2026) offers the longest locally-runnable context window at 10M tokens — orders of magnitude beyond any competitor. Its 17B active-parameter MoE runs at ~22–26 tok/s via Ollama's MLX engine on M5 Max, and the native vision encoder lets the orchestrator ingest diagrams and screenshots without a separate VLM call. At ~67 GB Q4 it still fits alongside a 17 GB model (e.g., Qwen3.6-27B) with room to spare — and now, after Mistral Medium 3.5's size correction, notably more headroom than a Scout+Mistral pairing would leave. Qwen3.6-27B covers 256K context at a quarter the size and is adequate for most orchestration pipelines that don't require mega-context.

**MCP tool-calling context:** The MCP-Mark leaderboard (Jun 2026, 8 models) shows Kimi K2.7 Code leading at 0.811 for MCP tool use, followed by Qwen3.7 Max (0.608) and Qwen3.6-35B-A3B (0.370). Neither Scout nor Qwen3.6-27B appear in this evaluation; not re-checked this cycle (no update found in the window). Cloud-only models leading MCP benchmarks doesn't change local picks.

**Also evaluated, not selected:** NVIDIA Nemotron 3 Super (Mar 11, 2026; 120B/12B-active, 1M context, ~60 GB at Q4, 2.2× GPT-OSS-120B throughput) — fits comfortably alongside another model but scores 60.47% SWE-bench Verified, well below Qwen3.6-27B and Scout's orchestration-specific strengths.

### 3.5 LLM-as-Judge / Verifier
*Calibrated scoring against rubrics, pairwise preference evaluation, output verification.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Qwen3.6-27B | ~17 GB | Strong instruction following; consistent in non-thinking mode; 256K ctx | `qwen3.6:27b` |
| **Smaller pick** | Gemma 4 12B Unified | ~8 GB | Fast; MMLU Pro 77.2%; low latency for high-volume scoring | `gemma4:12b` |

**Rationale:** For judging, **disable thinking mode** — reasoning traces inflate token cost and can introduce self-consistency bias in scores. Qwen3.6-27B's instruction fidelity and calibration make it the recommended local judge. Inject a scored reference example at the top of the judge prompt to anchor the scoring scale. **Staleness flag, unchanged this cycle:** updated RewardBench 2 data for the current window was not found — last confirmed data remains Sep 2025 (RewardBench 2's ICLR 2026 paper reflects a Jun 2025 submission, not a fresh benchmark refresh). For high-stakes judging, frontier API models remain preferred; use local judges for cost-sensitive or privacy-constrained pipelines.

### 3.6 Document Understanding / Interpreter
*PDFs, OCR, table and equation extraction.*

| | Model | Q4 resident | Key capability | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Gemma 4 26B MoE (E26B-A4B) | ~18 GB | Natively multimodal; strong table/equation/diagram extraction; 89% AIME 2026 | `gemma4:26b` |
| **Smaller pick** | Mistral OCR 4 | *(container, not GGUF)* | Purpose-built OCR; structured Markdown/JSON output; 170 languages, bounding-box/block classification | *Not Ollama-loadable — see below* |

**Rationale:** Gemma 4's 26B MoE variant (Google, Apr 2026) understands images, text, tables, and LaTeX natively in a single forward pass — no separate OCR pre-processing needed. It handles complex PDF layouts, nested tables, and mixed-language documents reliably. **Note on the smaller pick, updated this cycle:** Mistral OCR 4 (released Jun 23, 2026, within this cycle's window) is a real upgrade in this space and purpose-tuned for exactly this role, but it ships as a self-hostable container/API product, not a GGUF — no evidence was found this cycle of an Ollama-loadable form. It remains listed here as the recommended specialist for dedicated high-throughput OCR pipelines run *outside* Ollama; within the Ollama-only constraint of this document, there is currently no smaller in-library alternative confirmed better than falling back to Gemma 4 12B Unified (§3.1) for lighter-weight document tasks.

### 3.7 Vision / Image Understanding
*VLMs for image comprehension only — not generation.*

| | Model | Q4 resident | Key capability | Ollama tag |
|-|-------|------------|-----------------|------------|
| **Top pick** | Llama 4 Scout | ~67 GB | Natively integrated vision; coherent multi-image + long-text reasoning | `llama4:scout` |
| **Smaller pick** | Gemma 4 E4B | ~10 GB | Natively multimodal (image, audio, video); best small VLM available | `gemma4:e4b` |

**Rationale:** Llama 4 Scout integrates vision natively (not via LLaVA-style adapter), resulting in more coherent multi-modal reasoning when combining images with long context. Supports up to 5 input images per prompt. Gemma 4 E4B is the go-to when you need a capable VLM in a small footprint — also accepts audio and video natively. Cloud-only alternatives (Qwen3-VL-235B, InternVL3-78B) lead the open-weight VLM benchmarks globally but don't fit locally. No update found in this window.

---

## 4. The "Sufficient but Smaller" Angle

Four patterns consistently pay off on M5 Max 128 GB:

### 4.1 MoE-A3B Models
Models like **Laguna XS 2.1** (33B/3B active, ~20 GB at Q4_K_M), **Qwen 3.5 30B-A3B** (30B/3B active, ~17 GB at Q4), and the newly-surfaced **Ornith-1.0-35B** (35B/~3B active, ~21 GB at Q4) activate only ~3B parameters per token despite loading a much larger weight set. The result: 30B-class quality at 40–55 tok/s on M5 Max — similar speed to a 7–8B dense model, quality well above its size class. This remains the highest quality-per-GB pattern in the current Ollama library.

### 4.2 Distilled Reasoning Models
**DeepSeek-R1-Distill-Qwen-32B** transfers chain-of-thought behaviour from a 671B teacher into a 32B student (20 GB at Q4). It achieves MATH scores that beat many naive 70B models, at twice the speed and half the memory. For debugging and reasoning tasks, the distill is sufficient — you don't need to reach for a 70B.

### 4.3 Tiny Purpose-Trained Coding Models (new pattern this cycle)
**Ornith-1.0-9B** demonstrates a third pattern distinct from MoE sparsity or distillation: a *dense* 9B model, trained specifically around self-generated agentic-RL scaffolds, that outscores a 15 GB competitor (Devstral Small 2) at roughly a third the resident memory. If this pattern holds up under further field testing (tool-calling reliability in particular — not yet independently confirmed), it suggests purpose-built small coding models may be catching up to, and in narrow cases beating, the MoE-sparsity pattern in §4.1 for raw quality-per-GB.

### 4.4 Purpose-Built Specialized Small Models
A handful-of-GB specialist routinely beats a 70B generalist on its target domain:
- **Mistral OCR 4**: purpose-tuned OCR with structured output — more reliable for batch document processing than prompting a generalist, though it runs outside Ollama (container/API, not GGUF — see §3.6)
- **Gemma 4 E4B** (~10 GB): best small VLM; handles image/audio/video natively
- **nomic-embed-text**: best-in-class embeddings for RAG pipelines; negligible resident memory

The 128 GB envelope lets you keep several specialists resident simultaneously.

---

## 5. Suggested Concrete Stack

Ollama loads and unloads models from unified memory on demand. The full inventory fits on disk; concurrent-pair examples show what stays live together without eviction.

### Full Inventory

| Model | Ollama tag | Q4 resident | Primary role |
|-------|-----------|------------|---------------|
| Llama 4 Scout | `llama4:scout` | ~67 GB | Orchestrator + vision |
| Qwen3.6-27B | `qwen3.6:27b` | ~17 GB | Generalist + judge |
| DeepSeek-R1-Distill-Qwen-32B | `deepseek-r1:32b` | ~20 GB | Code debugger / reasoning |
| Laguna XS 2.1 | `laguna-xs-2.1:q4_K_M` | ~20 GB | Code implementer (top pick, higher SWE-bench) |
| **Ornith-1.0-9B** *(new)* | `ornith:9b` | ~5.6 GB | Code implementer (smaller pick) |
| Gemma 4 26B MoE | `gemma4:26b` | ~18 GB | Document understanding |
| Gemma 4 12B Unified | `gemma4:12b` | ~8 GB | Fast general + doc assistant |
| Gemma 4 E4B | `gemma4:e4b` | ~10 GB | Small vision / audio / video |
| *Devstral Small 2 (optional)* | `devstral-small-2` | ~15 GB | *Code implementer alt — battle-tested tool-calling track record* |
| *Mistral Medium 3.5 (optional)* | `mistral-medium-3.5:128b` | ~80 GB *(corrected)* | *Quality-critical one-shot coding, latency-tolerant only* |

**Total core inventory size on disk (excluding optionals):** ~165.6 GB.
Devstral Small 2 adds ~15 GB, Mistral Medium 3.5 adds ~80 GB if both are
also pulled. Ollama evicts from RAM on demand.

### Recommended Concurrent Pairs

| Pair | Combined RAM | Use case |
|------|-------------|----------|
| Scout + Qwen3.6 | ~84 GB | Orchestrator + workhorse; long-context agentic sessions |
| **Ornith-1.0-9B + DeepSeek-R1 32B** *(new)* | ~25.6 GB | Coding session: implement → debug loop, now ~10 GB lighter than the prior Devstral-based pairing |
| Qwen3.6 + Gemma 4 12B | ~25 GB | Lightweight dual-model chat + vision |
| Scout + Ornith-1.0-9B | ~72.6 GB | Orchestrated multi-file coding, lighter footprint than a Scout+Devstral or Scout+Laguna pairing |
| Mistral Medium 3.5 + Qwen3.6 | ~97 GB *(corrected from ~81 GB)* | Quality-first coding + fast judge — still fits within 128 GB but with less KV-cache headroom than previously stated |

Devstral Small 2 remains a drop-in substitute for Ornith-1.0-9B in any pairing above, at the cost of ~9 GB more resident memory, if its longer agentic tool-calling track record is preferred.

### What to Exclude from Local Use

| Model | Reason |
|-------|--------|
| Kimi K3 (2.8T total) | ~700+ GB even at 2-bit; open weights exist but no local Ollama/GGUF runtime support as of 2026-08-01 |
| Ornith-1.0-397B (397B total) | Q4 ≈ 200 GB (OOM); Q2 ≈ 100 GB fits but is untested/quality-degraded |
| Llama 4 Maverick (400B total) | Q4 ≈ 200 GB — hard OOM |
| Kimi K2.7 Code (1T total) | Community GGUF exists but ~585 GB at Q4 |

---

## 6. Verification Checklist

```bash
# Ensure Ollama is up-to-date (v0.32.5 as of 2026-08-01)
ollama version

# --- Pull core models ---
ollama pull qwen3.6:27b              # ~17 GB  — generalist / judge / debugger fallback
ollama pull llama4:scout              # ~67 GB  — orchestrator + vision
ollama pull deepseek-r1:32b           # ~20 GB  — reasoning / debugging
ollama pull laguna-xs-2.1:q4_K_M      # ~20 GB  — code implementation (top pick)
ollama pull ornith:9b                 # ~5.6 GB — code implementation (smaller pick, NEW this cycle)
ollama pull gemma4:26b                # ~18 GB  — document understanding
ollama pull gemma4:12b                # ~8 GB   — fast general
ollama pull gemma4:e4b                # ~10 GB  — small vision/audio/video

# --- Optional alternatives ---
# ollama pull devstral-small-2          # ~15 GB  — code implementation alt (battle-tested tool-calling)
# ollama pull mistral-medium-3.5:128b   # ~80 GB  — 77.6% SWE-bench but ~6-8 tok/s (size/speed corrected this cycle)

# --- Verify resident memory after load ---
# (run `ollama ps` while the model is active)
# qwen3.6:27b           → 16–18 GB
# llama4:scout          → 65–70 GB
# deepseek-r1:32b       → 19–21 GB
# laguna-xs-2.1:q4_K_M  → 18–21 GB
# ornith:9b             → 5–6 GB
# gemma4:26b            → 17–19 GB
# gemma4:12b            → 7–9 GB
# gemma4:e4b            → 9–11 GB
# devstral-small-2      → 14–16 GB
# mistral-medium-3.5:128b → 78–82 GB

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

# Agentic coding, top pick (Laguna XS 2.1)
ollama run laguna-xs-2.1:q4_K_M "Refactor this function to handle None inputs gracefully: def get_len(s): return len(s)"

# Agentic coding, smaller pick (Ornith-1.0-9B) — NEW this cycle
ollama run ornith:9b "Implement a Python function that validates an email address using a regex, with unit tests."

# Vision (Gemma 4 E4B) — requires an image file
# ollama run gemma4:e4b "Describe what you see in this image." --image /path/to/screenshot.png

# Document understanding (Gemma 4 26B MoE)
ollama run gemma4:26b "Extract all column headers and row values from this table image." --image /path/to/table.png
```

---

## 7. Sources

### Platform & Release Notes
- [Ollama Releases — GitHub](https://github.com/ollama/ollama/releases) (primary — v0.32.3/.4/.5 confirmed this cycle)
- [Ollama July 2026: v0.32.0 Update + Best Models by Use Case — PromptQuorum](https://www.promptquorum.com/local-llms/top-open-source-models-ollama)
- [Ollama Latest Version + Version History — Local AI Master](https://localaimaster.com/blog/ollama-version-history)
- [Ollama Library](https://ollama.com/library)

### Ollama Library Pages Fetched Directly This Cycle (primary)
- [ornith / tags](https://ollama.com/library/ornith/tags) — confirmed `ornith:9b` 5.6 GB, `ornith:9b-q8_0` 9.5 GB, `ornith:9b-bf16` 18 GB
- [mistral-medium-3.5 / tags](https://ollama.com/library/mistral-medium-3.5/tags) — confirmed `:128b`/`:128b-q4_K_M` 80 GB (corrected from prior doc's 64 GB)
- [kimi-k3](https://ollama.com/library/kimi-k3) — confirmed cloud-only tag

### Apple Silicon Benchmarks
- [Apple Silicon LLM Benchmarks 2026 — LLMCheck](https://llmcheck.net/benchmarks)
- [M5 Max for Local AI: Complete Benchmark Guide 2026 — LLMCheck](https://llmcheck.net/blog/apple-silicon-m5-max-local-ai-guide/)
- [M5 Pro vs M5 Max 2026: Benchmark tok/s Comparison — PromptQuorum](https://www.promptquorum.com/local-llms/m5-pro-max-llm-benchmarks-2026)
- [Local LLM Tokens-per-Second Benchmarks 2026 — Presenc AI](https://presenc.ai/research/local-llm-tokens-per-second-benchmarks-2026)
- [Apple M5 Max Local LLM: 128GB Inference Guide 2026 — AI Productivity](https://aiproductivity.ai/blog/apple-m5-max-local-llm-guide/)
- [70B Models on M5 Max 128GB — PromptQuorum](https://www.promptquorum.com/local-llms/running-70b-models-apple-silicon-m5-max)
- *Wale Akinfaderin, "Benchmarking Open-Weights LLMs on the MacBook Pro M5 Max" (Medium) — attempted this cycle, returned HTTP 503, could not be reviewed; worth retrying next cycle.*

### Ornith-1.0 (new this refresh)
- [deepreinforce-ai/Ornith-1.0-9B — Hugging Face](https://huggingface.co/deepreinforce-ai/Ornith-1.0-9B) (primary)
- [deepreinforce-ai/Ornith-1.0-9B-GGUF — Hugging Face](https://huggingface.co/deepreinforce-ai/Ornith-1.0-9B-GGUF) (primary)
- [deepreinforce-ai/Ornith-1 — GitHub](https://github.com/deepreinforce-ai/Ornith-1) (primary)
- [Ornith-1.0: Self-Scaffolding LLMs for Agentic Coding — DeepReinforce Blog](https://deep-reinforce.com/ornith_1_0.html) (primary)
- [DeepReinforce Releases Ornith-1.0 — MarkTechPost](https://www.marktechpost.com/2026/06/25/deepreinforce-releases-ornith-1-0-an-open-source-coding-model-family-that-learns-its-own-rl-scaffolds/) (secondary — 69.4% SWE-bench Verified for 9B)
- [Ornith-1.0-9B Benchmarks — BenchLM.ai](https://benchlm.ai/models/ornith-1-0-9b) (secondary — independently agrees on 69.4%)
- [Ornith 1.0 Models comparison](https://ornith.site/models/) (secondary)

### Laguna XS 2.1
- [poolside/Laguna-XS-2.1 — Hugging Face](https://huggingface.co/poolside/Laguna-XS-2.1) (primary — 70.9% SWE-bench Verified re-confirmed this cycle)
- [Introducing Laguna XS 2.1 — Poolside](https://poolside.ai/blog/introducing-laguna-xs-2-1) (primary)
- [Laguna XS 2.1 on Ollama](https://ollama.com/library/laguna-xs-2.1) (primary)

### Mistral
- [mistral-medium-3.5:128b — Ollama](https://ollama.com/library/mistral-medium-3.5) (primary — size correction source)
- [Mistral Medium 3.5 — Mistral Docs](https://docs.mistral.ai/models/model-cards/mistral-medium-3-5-26-04) (primary)
- [Introducing Devstral 2 and Mistral Vibe CLI — Mistral AI](https://mistral.ai/news/devstral-2-vibe-cli/) (primary)
- [Devstral Small 2 — Ollama](https://ollama.com/library/devstral-small-2) (primary)
- Mistral OCR 4 — no primary Mistral changelog entry independently confirmed this cycle; secondary coverage only (explainx.ai, techtimes.com, digitalapplied.com)

### Qwen Models
- [Qwen3.6-27B — Qwen Blog](https://qwen.ai/blog?id=qwen3.6-27b) (primary)
- [Qwen3.6-27B Tags — Ollama](https://ollama.com/library/qwen3.6/tags) (primary)
- Qwen3.7 Flash / Qwen3.8-Max — secondary coverage only (yottalabs.ai, benchlm.ai, aimadetools.com); no open weights, not locally relevant

### Llama 4 / Meta
- [The Llama 4 Herd — Meta AI Blog](https://ai.meta.com/blog/llama-4-multimodal-intelligence/) (primary)
- [Llama4 Tags — Ollama](https://ollama.com/library/llama4/tags) (primary)

### Gemma 4 / Google
- [Gemma4 Tags — Ollama](https://ollama.com/library/gemma4/tags) (primary)
- [Gemma 4: Specs, Benchmarks, and Local Guide — Auriga IT](https://aurigait.com/blog/gemma-4-features-benchmarks-guide/) (secondary)

### Kimi K3 / Moonshot
- [Kimi K3 — Ollama](https://ollama.com/library/kimi-k3) (primary — cloud-only tag confirmed)
- [Kimi K3: 2.8T MoE — Local AI Master](https://localaimaster.com/models/kimi-k3) (secondary)

### Benchmarks & Leaderboards
- [SWE-bench Verified Leaderboard — BenchLM.ai](https://benchlm.ai/benchmarks/sweVerified) (secondary, snapshot 2026-07-31)
- [SWE-bench Verified — Vals.ai](https://www.vals.ai/benchmarks/swebench) (primary-adjacent leaderboard)
- [Terminal-Bench 2.0/2.1 — BenchLM.ai](https://benchlm.ai/benchmarks/terminalBench2)
- [MCP-Mark Leaderboard — LLM Stats](https://llm-stats.com/benchmarks/mcp-mark)
- [RewardBench 2: ICLR 2026 Paper](https://arxiv.org/pdf/2506.01937) (primary — no fresher judge-benchmark data found this cycle)

### GLM-5.2 / Z.ai
- [GLM-5.2 beats GPT-5.5 on long-horizon coding benchmarks — VentureBeat](https://venturebeat.com/technology/z-ais-open-weights-glm-5-2-beats-gpt-5-5-on-multiple-long-horizon-coding-benchmarks-for-1-6th-the-cost)
- [Run GLM-5.2 Locally: Ollama, VRAM & Hardware Guide — GLM5.app](https://glm5.app/blog/how-to-run-glm-5-2-locally)

### MiniMax M3
- [MiniMax M3 Open Weights Are Live — Nerova](https://nerova.ai/news/minimax-m3-open-weight-agent-builders-june-2026)
- [Best Open Source LLMs July 2026 Leaderboard — BenchLM.ai](https://benchlm.ai/best/open-source)

### DeepSeek
- [DeepSeek V4 Pro Benchmarks — BenchLM.ai](https://benchlm.ai/models/deepseek-v4-pro)
- [DeepSeek-V4-Pro — Ollama](https://ollama.com/library/deepseek-v4-pro)

### Kimi K2 / Kimi K2.7 Code
- [Kimi K2.7 Code on Ollama](https://ollama.com/library/kimi-k2.7-code)
- [unsloth/Kimi-K2.7-Code-GGUF — Hugging Face](https://huggingface.co/unsloth/Kimi-K2.7-Code-GGUF)

### NVIDIA Nemotron 3 Super
- [Nemotron 3 Super Technical Report — NVIDIA Research (arXiv)](https://arxiv.org/pdf/2604.12374)

### Cohere
- [Meet North Mini Code — MarkTechPost](https://www.marktechpost.com/2026/06/11/meet-north-mini-code-coheres-30b-open-weight-mixture-of-experts-model-with-3b-active-parameters-for-agentic-coding/)

### Reference
- [Ollama VRAM Requirements 2026 — Local AI Master](https://localaimaster.com/blog/ollama-model-ram-vram-table)
- [GGUF Quantization Guide — Easton Blog](https://eastondev.com/blog/en/posts/ai/20260422-ollama-gguf-quantization/)
