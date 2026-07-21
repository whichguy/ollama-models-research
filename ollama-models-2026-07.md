# Ollama Models Research — 2026-07 Refresh

**Generated:** 2026-07-21  
**Hardware target:** Apple M5 Max · 128 GB unified memory · 614 GB/s bandwidth  
**Research window:** Last 60 days (2026-05-21 → 2026-07-21)

> **Diffs from previous refresh (`ollama-models-2026-07.md`, generated 2026-07-18):**
>
> **No role picks changed.** Four items are new or updated since July 18:
>
> 1. **Kimi K3** (Moonshot AI, July 16, 2026) — The largest open-weight model ever announced: 2.8T total / 50.4B active MoE, 1M context, native vision. Strong benchmarks: 93.5% GPQA Diamond, 88.3% Terminal-Bench 2.1, 76.8% SWE-bench Verified (KimiCode harness). But no GGUF/Ollama/llama.cpp/MLX support as of 2026-07-21 (weights promised July 27), and even at 2-bit the model needs ~700+ GB — approximately 5–6× this hardware's envelope. Added to §1 cloud-only table and §2. **Does not affect any role pick.**
>
> 2. **Ornith-1.0-397B** (deepreinforce-ai, June 25, 2026) — New open-weight SWE-bench Verified leader at 82.4%, topping DeepSeek V4 Pro Max's 80.6%. Architecture: Qwen3_5 MoE, ~3B active parameters per token, 262K context, MIT license. Unsloth GGUF exists. However 397B total × Q4_K_M ≈ 200 GB (OOM); extreme Q2 ≈ 100 GB would technically fit but with significant quality degradation at an untested quant level for this model. Classified as cloud-only for practical purposes on 128 GB hardware. Added to §1 and §2. **Does not affect any role pick.**
>
> 3. **Mistral Medium 3.5** (Mistral AI, Apr 29, 2026) — 128B dense, 77.6% SWE-bench Verified (higher than Laguna XS 2.1's 70.9%), Q4_K_M ≈ 64 GB, real Ollama tag (`mistral-medium-3.5:128b`). **This model fits in 128 GB.** However, at 128B dense it runs at approximately 9–12 tok/s on M5 Max — roughly 4× slower than Laguna XS 2.1's 43–53 tok/s. For agentic coding loops (many 500–2000 token completions per session), the speed penalty dominates: a 1K-token output takes ~90–110 s vs ~20–25 s. **Laguna XS 2.1 remains the top pick for §3.2** (higher throughput wins in loops). Mistral Medium 3.5 is worth considering for one-shot quality-critical tasks where latency is unconstrained. Added to §2 and §3.2 notes.
>
> 4. **Ollama v0.32.1** (July 16, 2026) — Minor patch: Gemma 4 multi-turn tool-call fixes, MLX model cache leak patched, adds `OLLAMA_LOAD_TIMEOUT` for MLX text models. Upgrade recommended.

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
| Qwen3.6-27B | Dense 27B | Q4_K_M | ~45 | ~70 | ~85 |
| Qwen 3.5 30B-A3B | MoE 30B/3B active | Q4_K_M | ~45 | ~55 | ~68 |
| Laguna XS 2.1 | MoE 33B/3B active | Q4_K_M | ~43 | ~53 | ~65 |
| DeepSeek-R1-Distill 32B | Dense 32B | Q4_K_M | ~27 | ~45 | ~60 |
| Mistral Medium 3.5 | Dense 128B | Q4_K_M | ~9–12 | ~14–16 | ~18 |
| Llama 4 Scout | MoE 109B/17B active | Q4_K_M | ~22 | ~26 | ~50 |
| Llama 3.3 70B | Dense 70B | Q4_K_M | ~12–18 | ~15–22 | ~28 |

**Measurement note:** "Ollama" column = Ollama v0.32.x via llama.cpp backend. "Ollama MLX" = Ollama's native MLX path (available since v0.30.0, significantly improved in v0.32.0+). "mlx_lm native" = via `mlx_lm.generate` directly, bypassing Ollama overhead. LLMCheck (July 2026) is the primary source for the first two columns; mlx_lm native figures from PromptQuorum and Presenc AI. Context length, KV-cache fill, and prompt length all materially affect tok/s — treat these as interactive-session estimates at moderate context (2K–8K tokens), not batch throughput.

Sources: [LLMCheck Apple Silicon Benchmarks](https://llmcheck.net/benchmarks) · [LLMCheck M5 Max Guide](https://llmcheck.net/blog/apple-silicon-m5-max-local-ai-guide/) · [PromptQuorum M5 Max Benchmarks](https://www.promptquorum.com/local-llms/m5-pro-max-llm-benchmarks-2026) · [Presenc AI Local Benchmarks 2026](https://presenc.ai/research/local-llm-tokens-per-second-benchmarks-2026)

### Runtime Notes

- **MLX vs llama.cpp:** Ollama's bundled MLX engine (available since v0.30.0, substantially improved in v0.32.0) gives 20–50% uplift over its llama.cpp path on Apple Silicon. Using `mlx_lm` directly gives another 30–50% on top of that. For speed-critical workflows, run `mlx_lm` directly; for convenience and tool/API compatibility, Ollama's MLX path is the right default.
- **MoE-A3B behaviour:** MoE models activate only ~3B parameters per token despite loading a much larger full weight set. Laguna XS 2.1 (33B total, ~20 GB at Q4_K_M) and Qwen 3.5 30B-A3B (30B total, ~17 GB) deliver 30B-class quality while running at similar tok/s to a 7–8B dense model. This remains the highest quality-per-GB pattern in the current Ollama library.
- **Ollama v0.32.1 uplift (Gemma 4):** Gemma 4 runs ~90% faster in v0.32.0+ due to multi-token prediction (MTP) and automatic hardware tuning. Upgrade to v0.32.1 before benchmarking any Gemma 4 variant; the patch also fixes MLX cache leaks that caused memory growth across requests.
- **KV-cache headroom:** At 128 GB, reserve ~10–18 GB for macOS + long-context KV-cache. Practical ceiling for a single loaded model is ~110 GB — comfortably above even Llama 4 Scout's ~67 GB footprint.
- **M5 Max vs M4 Max:** ~28% higher tok/s across all model sizes (614 GB/s vs 546 GB/s bandwidth, improved GPU, redesigned Neural Engine).

### What Exceeds 128 GB (Cloud-Only or Borderline)

| Model | Why it won't fit |
|-------|------------------|
| **Kimi K3** (Moonshot, 2.8T/50.4B active MoE) | NEW (Jul 16, 2026). No GGUF/Ollama yet (weights promised Jul 27). Native weights ~1.4 TB; even 2-bit quants ≈ 700+ GB. **Cloud-only.** |
| **Ornith-1.0-397B** (deepreinforce-ai, 397B/~3B active MoE) | NEW (Jun 25, 2026). Unsloth GGUF exists. Q4_K_M ≈ 200 GB (OOM). Extreme Q2 ≈ 100 GB fits numerically but quality untested; not a practical local choice on 128 GB. **Effectively cloud-only.** |
| **GLM-5.2** (Z.ai, 744B/40B active MoE) | 62.1% SWE-bench Pro, 91.2% GPQA Diamond; smallest usable GGUF ~217 GB; Ollama only offers `:cloud`. |
| **MiniMax M3** (MiniMax, ~428B/23B active MoE) | #1 on BenchLM open-source leaderboard (Jul 17, 2026); 80.5% SWE-bench Verified; smallest local quant ~143 GB; Ollama only offers `:cloud`. |
| Llama 4 Maverick (400B total) | Q4 ≈ 200 GB — hard OOM |
| Kimi K2.7 Code (1T total) | Community GGUF builds exist but ~585 GB at Q4; cloud-only for this hardware |
| **DeepSeek-V4-Pro-Max** (1.6T/49B active MoE) | 80.6% SWE-bench Verified — second-highest open-weight score confirmed this session — but ~800 GB minimum at Q4; a circulating "~50 GB" claim is arithmetically impossible for 1.6T params and was rejected |
| DeepSeek-V3 full (671B MoE, 37B active) | Q4 ≈ 170 GB — borderline, likely OOM with KV-cache |
| Any 700B+ total-parameter model | Exceeds envelope at any practical quant |

---

## 2. Current Model Landscape (Last 60 Days)

### Models that moved SOTA (May 21 – Jul 21, 2026)

| Model | Family | Released | Why it matters |
|-------|--------|----------|----------------|
| **Kimi K3** | Moonshot AI (MoE 2.8T/50.4B active) | Jul 16, 2026 | Largest open-weight model ever announced; 1M context; native vision/video; 93.5% GPQA Diamond, 88.3% Terminal-Bench 2.1, 76.8% SWE-bench Verified (KimiCode harness), #1 Program Bench 77.8 — but ~1.4 TB native, no GGUF/Ollama/MLX as of 2026-07-21. **Cloud-only — see §1.** |
| **Ornith-1.0-397B** | deepreinforce-ai (MoE 397B/~3B active) | Jun 25, 2026 | NEW open-weight SWE-bench Verified leader at 82.4%, topping DeepSeek V4 Pro Max. Qwen3_5 MoE architecture, 262K ctx, MIT license. Unsloth GGUF exists but Q4 ≈ 200 GB, Q2 ≈ 100 GB (borderline/quality-degraded). **Effectively cloud-only on 128 GB — see §1.** |
| **Laguna XS 2.1** | Poolside (MoE 33B/3B active) | Jul 2, 2026 | 70.9% SWE-bench Verified; highest-scoring *locally-runnable* Ollama model per GB in its size tier; 256K ctx; Apache-compatible license |
| **Mistral Medium 3.5** | Mistral AI (dense 128B) | Apr 29, 2026 | 77.6% SWE-bench Verified (higher than Laguna XS 2.1); Q4_K_M ≈ 64 GB, Ollama tag exists (`mistral-medium-3.5:128b`); Modified MIT. **Fits in 128 GB but runs at ~9–12 tok/s — too slow for agentic coding loops.** Viable for quality-critical one-shot tasks. See §3.2. |
| **GLM-5.2** | Z.ai (MoE 744B/40B active) | Jun 13, 2026 | 62.1% SWE-bench Pro, 91.2% GPQA Diamond, MIT license — strongest all-round open-weight as of July 2026; doesn't fit 128 GB (~217 GB+); Ollama only offers `:cloud`. **Cloud-only — see §1.** |
| **MiniMax M3** | MiniMax (MoE ~428B/23B active) | Jun 1, 2026 | #1 on BenchLM open-source leaderboard (Jul 17, 2026); 80.5% SWE-bench Verified; 1M ctx; native multimodal — but smallest local quant ~143 GB; Ollama only offers `:cloud`. **Cloud-only — see §1.** |
| **Kimi K2.7 Code** | Moonshot AI (MoE 1T/32B active) | Jun 2026 | Highest MCP-Mark score of any model (0.811 vs next-best Qwen3.7 Max at 0.608) — strongest for MCP tool-calling; community GGUF exists but cloud-only at ~585 GB |
| **Cohere North Mini Code 1.0** | Cohere (MoE 30B/3B active) | Jun 11, 2026 | 67.6% SWE-bench Verified; Apache 2.0; real Ollama tag (`north-mini-code-1.0`) — same size class as Laguna XS 2.1 but scores lower; genuinely fits and pulls locally |
| **DeepSeek-V4-Pro-Max** | DeepSeek (MoE 1.6T/49B active) | ~Apr 23, 2026 | 80.6% SWE-bench Verified (now second-highest open-weight, below Ornith); ~800 GB at Q4; **cloud-only** |
| **Gemma 4 12B Unified** | Google (dense 12B) | Jun 2026 | Fills mid-size gap in Gemma lineup; natively multimodal; 77.2% MMLU Pro |
| **Ollama v0.32.1** | Platform release | Jul 16, 2026 | Gemma 4 tool-call fixes, MLX cache leak patch, OLLAMA_LOAD_TIMEOUT support. Minor patch; upgrade recommended. |
| **Ollama v0.32.0** | Platform release | Jul 11, 2026 | Native MLX engine integration on Apple Silicon; agentic "code + delegate" mode; Gemma 4 ~90% faster |

### Notable models evaluated just outside the 60-day window

| Model | Family | Released | Why noted |
|-------|--------|----------|-----------|
| **NVIDIA Nemotron 3 Super** | NVIDIA (hybrid Mamba-Transformer, 120B/12B active) | Mar 11, 2026 | ~60 GB at Q4 (fits comfortably), 1M context, 2.2× GPT-OSS-120B throughput — but 60.47% SWE-bench Verified below all current role picks. Evaluated for plan-orchestrator; not selected. |

### Context: models released just outside 60-day window but shaping current picks

| Model | Released | Relevance |
|-------|----------|-----------|
| Qwen3.6-27B | Apr 22, 2026 | 77.2% SWE-bench Verified; replaces Qwen3-30B-A3B as the everyday dense recommendation |
| Gemma 4 (E2B/E4B/26B-MoE/31B) | Apr 2, 2026 | All variants natively multimodal; Gemma 4 26B-MoE is the document-understanding pick |
| Llama 4 Scout (109B MoE) | Apr 2026 | 10M-token context window; only locally-runnable model at that scale |
| Devstral Small 2 (24B) | 2025-12/2026-01 | Most-tested open agentic coding model in production scaffolds; Apache 2.0 |

### SWE-bench Verified Snapshot (July 20, 2026)

For context on where role picks sit in the broader leaderboard:

| Rank | Model | Score | Locally runnable? |
|------|-------|-------|-------------------|
| 1 | Claude Mythos 5 | 95.5% | No (API-only) |
| 2 | Claude Fable 5 | 95.0% | No |
| 3 | Claude Opus 4.8 | 88.6% | No |
| **7** | **Ornith-1.0-397B** | **82.4%** | **No (200 GB Q4)** |
| **10** | **DeepSeek V4 Pro Max** | **80.6%** | **No (800 GB Q4)** |
| **11** | **MiniMax M3** | **80.5%** | **No (143 GB min)** |
| — | Mistral Medium 3.5 | 77.6% | **Yes (~64 GB Q4; slow)** |
| — | Qwen3.6-27B | 77.2% | **Yes (~17 GB Q4)** |
| — | **Laguna XS 2.1** | **70.9%** | **Yes (~20 GB Q4) — top local pick** |
| — | Devstral Small 2 | 68.0% | **Yes (~15 GB Q4)** |
| — | Cohere North Mini Code 1.0 | 67.6% | Yes (~17 GB Q4) |

Source: [BenchLM.ai SWE-bench Verified Leaderboard](https://benchlm.ai/benchmarks/sweVerified)

---

## 3. Recommendations by Role

### 3.1 Generalist Agentic Default
*One model that does most jobs adequately.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | Qwen3.6-27B | ~17 GB | SWE-bench Verified 77.2%; Terminal-Bench 2.0 59.3%; 256K ctx | `qwen3.6:27b` |
| **Smaller pick** | Gemma 4 12B Unified | ~8 GB | MMLU Pro 77.2%; natively multimodal (text + image) | `gemma4:12b` |

**Rationale:** Qwen3.6-27B (Alibaba, Apr 22, 2026) is a 27B dense model with a built-in dual-mode thinking toggle: `/think` on for deep reasoning, off for fast chat. At 77.2% SWE-bench Verified it sits within 3.7 points of Claude Opus 4.8 (88.6% as of Jul 20) while running entirely locally. The 256K context window handles most agentic sessions without truncation. Gemma 4 12B Unified is the right answer when memory is at a premium or multimodal input is needed at low cost.

**Sizing note:** ~16.8 GB at Q4_K_M; ~19.5 GB at Q5_K_M; ~28.6 GB at Q8_0. Q5_K_M is recommended if the extra 3 GB is available — marginal quality uplift compounds in agentic loops.

### 3.2 Code Implementer
*Writes code, multi-file edits, agentic coding loops.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | Laguna XS 2.1 | ~20 GB | SWE-bench Verified 70.9%; SWE-bench Multilingual 63.1%; 256K ctx | `laguna-xs-2.1:q4_K_M` |
| **Smaller pick** | Devstral Small 2 | ~15 GB | SWE-bench Verified 68.0%; 256K ctx; Apache 2.0 | `devstral-small-2` |

**Rationale:** Laguna XS 2.1 (Poolside, Jul 2, 2026) is a 33B/3B-active MoE scoring 70.9% on SWE-bench Verified — the highest of any Ollama-available model in its memory tier as of July 2026. MoE sparsity keeps generation at ~43–53 tok/s (Ollama/MLX on M5 Max), making it fast enough for interactive agentic loops. The 256K context window handles large multi-file codebases. Devstral Small 2 (Mistral, 24B dense) is nearly as capable at 68.0% and is more battle-tested in production agentic scaffolds (OpenHands, SWE-agent). Devstral 2 full (123B, 72.2% SWE-bench) is the strongest open-weight coder with an Ollama tag but requires ~65 GB Q4 — fits solo but leaves little room for concurrent models.

**Also evaluated, not selected:**
- *Mistral Medium 3.5* (Apr 2026; 128B dense; 77.6% SWE-bench Verified — higher than Laguna; `mistral-medium-3.5:128b`, ~64 GB Q4, Modified MIT): **fits in 128 GB**, but dense 128B generates at only ~9–12 tok/s on M5 Max. For a typical agentic coding session producing 500–2000 tokens per turn, that's 45–220 seconds per completion vs 10–50 seconds for Laguna XS 2.1. The quality advantage (6.7 pp SWE-bench) is real, but the latency penalty dominates for iterative loops. **Prefer Mistral Medium 3.5 only for quality-critical, non-latency-sensitive one-shot tasks.** Pull with `ollama pull mistral-medium-3.5:128b`.
- *Cohere North Mini Code 1.0* (Jun 11, 2026; 30B/3B-active MoE; 67.6% SWE-bench; `north-mini-code-1.0`): same size class as Laguna XS 2.1, scores lower.
- *Ornith-1.0-397B* (82.4% SWE-bench Verified, new open-weight leader) and *DeepSeek-V4-Pro-Max* (80.6%), *MiniMax M3* (80.5%), *Kimi K3* (76.8%) top the global leaderboard but none fit 128 GB at practical quants — see §1. Laguna XS 2.1 remains the best *locally-runnable* pick.

### 3.3 Code Debugger
*Reasoning / chain-of-thought, root-cause analysis, math-heavy debugging.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | DeepSeek-R1-Distill-Qwen-32B | ~20 GB | MMLU 83%; MATH 72% (highest of any sub-40 GB local model) | `deepseek-r1:32b` |
| **Smaller pick** | Qwen3.6-27B (thinking mode on) | ~17 GB | SWE-bench 77.2%; switchable CoT; same model as §3.1 | `qwen3.6:27b` |

**Rationale:** DeepSeek-R1-Distill-Qwen-32B provides always-on chain-of-thought reasoning and holds the highest MATH benchmark score of any locally-runnable model under 40 GB. Distilled from the 671B R1 teacher; generates full reasoning traces before answering. Use it for debugging sessions requiring step-by-step logical trace through a logic fault or complex algorithm. Qwen3.6-27B with `/think` is the pragmatic alternative if you want to avoid loading a second model.

### 3.4 Plan Orchestrator
*Long context + reliable tool calling + structured output for multi-agent coordination.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | Llama 4 Scout | ~67 GB | 10M-token context; 17B active / 109B total MoE; natively multimodal; strong tool calling | `llama4:scout` |
| **Smaller pick** | Qwen3.6-27B | ~17 GB | 256K context; reliable structured output; strong tool fidelity | `qwen3.6:27b` |

**Rationale:** Llama 4 Scout (Meta, Apr 2026) offers the longest locally-runnable context window at 10M tokens — orders of magnitude beyond any competitor. Its 17B active-parameter MoE runs at ~22–26 tok/s via Ollama's MLX engine on M5 Max, and the native vision encoder lets the orchestrator ingest diagrams and screenshots without a separate VLM call. At ~67 GB Q4 it still fits alongside a 17 GB model (e.g., Qwen3.6-27B) with room to spare. Qwen3.6-27B covers 256K context at a quarter the size and is adequate for most orchestration pipelines that don't require mega-context.

**MCP tool-calling context:** The MCP-Mark leaderboard (Jun 2026, 8 models) shows Kimi K2.7 Code leading at 0.811 for MCP tool use, followed by Qwen3.7 Max (0.608) and Qwen3.6-35B-A3B (0.370). Neither Scout nor Qwen3.6-27B appear in this evaluation, but Qwen3.6's instruction fidelity is consistently cited for reliable tool calling. Cloud-only models (Kimi K2.7 Code, GLM-5.2) leading MCP benchmarks doesn't change local picks.

**Also evaluated, not selected:** NVIDIA Nemotron 3 Super (Mar 11, 2026; 120B/12B-active, 1M context, ~60 GB at Q4, 2.2× GPT-OSS-120B throughput) — fits comfortably alongside another model but scores 60.47% SWE-bench Verified, well below Qwen3.6-27B and Scout's orchestration-specific strengths (10M context, native vision).

### 3.5 LLM-as-Judge / Verifier
*Calibrated scoring against rubrics, pairwise preference evaluation, output verification.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | Qwen3.6-27B | ~17 GB | Strong instruction following; consistent in non-thinking mode; 256K ctx | `qwen3.6:27b` |
| **Smaller pick** | Gemma 4 12B Unified | ~8 GB | Fast; MMLU Pro 77.2%; low latency for high-volume scoring | `gemma4:12b` |

**Rationale:** For judging, **disable thinking mode** — reasoning traces inflate token cost and can introduce self-consistency bias in scores. Qwen3.6-27B's instruction fidelity and calibration make it the recommended local judge. Inject a scored reference example at the top of the judge prompt to anchor the scoring scale. **Important caveat:** Updated RewardBench 2 data for May–July 2026 was not found during this research session (last confirmed data: Sep 2025). For high-stakes judging (RLHF reward labeling), frontier models remain preferred — use local judges for cost-sensitive or privacy-constrained pipelines.

### 3.6 Document Understanding / Interpreter
*PDFs, OCR, table and equation extraction.*

| | Model | Q4 resident | Key capability | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | Gemma 4 26B MoE (E26B-A4B) | ~18 GB | Natively multimodal; strong table/equation/diagram extraction; 89% AIME 2026 | `gemma4:26b` |
| **Smaller pick** | Mistral OCR 4 | ~6 GB | Purpose-built OCR; structured Markdown/JSON output; best for high-throughput privacy-preserving pipelines | *(check `ollama search mistral-ocr`)* |

**Rationale:** Gemma 4's 26B MoE variant (Google, Apr 2026) understands images, text, tables, and LaTeX natively in a single forward pass — no separate OCR pre-processing needed. It handles complex PDF layouts, nested tables, and mixed-language documents reliably. For dedicated high-throughput pipelines (batch invoice processing, form extraction), Mistral OCR 4 is purpose-tuned and outputs clean structured Markdown or JSON — outperforms generalists on this narrow domain.

### 3.7 Vision / Image Understanding
*VLMs for image comprehension only — not generation.*

| | Model | Q4 resident | Key capability | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | Llama 4 Scout | ~67 GB | Natively integrated vision; coherent multi-image + long-text reasoning | `llama4:scout` |
| **Smaller pick** | Gemma 4 E4B | ~10 GB | Natively multimodal (image, audio, video); best small VLM available | `gemma4:e4b` |

**Rationale:** Llama 4 Scout integrates vision natively (not via LLaVA-style adapter), resulting in more coherent multi-modal reasoning when combining images with long context. Supports up to 5 input images per prompt. Gemma 4 E4B is the go-to when you need a capable VLM in a small footprint — also accepts audio and video natively. Cloud-only alternatives (Qwen3-VL-235B, InternVL3-78B) lead the open-weight VLM benchmarks globally but don't fit locally.

---

## 4. The "Sufficient but Smaller" Angle

Three patterns consistently pay off on M5 Max 128 GB:

### 4.1 MoE-A3B Models
Models like **Laguna XS 2.1** (33B/3B active, ~20 GB at Q4_K_M) and **Qwen 3.5 30B-A3B** (30B/3B active, ~17 GB at Q4) activate only ~3B parameters per token despite loading a much larger weight set. The result: 30B-class quality at 43–55 tok/s on M5 Max (Ollama MLX) — similar speed to a 7–8B dense model, quality well above its size class. This is the highest quality-per-GB pattern in the current Ollama library.

### 4.2 Distilled Reasoning Models
**DeepSeek-R1-Distill-Qwen-32B** transfers chain-of-thought behaviour from a 671B teacher into a 32B student (20 GB at Q4). It achieves MATH scores that beat many naive 70B models, at twice the speed and half the memory. For debugging and reasoning tasks, the distill is sufficient — you don't need to reach for a 70B.

### 4.3 Purpose-Built Specialized Small Models
A 6–10 GB specialist routinely beats a 70B generalist on its target domain:
- **Mistral OCR 4** (~6 GB): purpose-tuned OCR with structured output — more reliable for batch document processing than prompting a generalist
- **Gemma 4 E4B** (~10 GB): best small VLM; handles image/audio/video natively
- **nomic-embed-text**: best-in-class embeddings for RAG pipelines; negligible resident memory

The 128 GB envelope lets you keep several specialists resident simultaneously.

---

## 5. Suggested Concrete Stack

Ollama loads and unloads models from unified memory on demand. The full inventory fits on disk; concurrent-pair examples show what stays live together without eviction.

### Full Inventory

| Model | Ollama tag | Q4 resident | Primary role |
|-------|-----------|------------|-------------|
| Llama 4 Scout | `llama4:scout` | ~67 GB | Orchestrator + vision |
| Qwen3.6-27B | `qwen3.6:27b` | ~17 GB | Generalist + judge |
| DeepSeek-R1-Distill-Qwen-32B | `deepseek-r1:32b` | ~20 GB | Code debugger / reasoning |
| Devstral Small 2 | `devstral-small-2` | ~15 GB | Code implementer (battle-tested) |
| Laguna XS 2.1 | `laguna-xs-2.1:q4_K_M` | ~20 GB | Code implementer (higher SWE-bench) |
| Gemma 4 12B Unified | `gemma4:12b` | ~8 GB | Fast general + doc assistant |
| Gemma 4 E4B | `gemma4:e4b` | ~10 GB | Small vision / audio / video |
| *Mistral Medium 3.5* | `mistral-medium-3.5:128b` | ~64 GB | *Optional: quality-critical one-shot coding* |

**Total core inventory size on disk:** ~157 GB. Mistral Medium 3.5 adds ~64 GB if pulled. Ollama evicts from RAM on demand.

### Recommended Concurrent Pairs

| Pair | Combined RAM | Use case |
|------|-------------|----------|
| Scout + Qwen3.6 | ~84 GB | Orchestrator + workhorse; long-context agentic sessions |
| Devstral Small 2 + DeepSeek-R1 32B | ~35 GB | Coding session: implement → debug loop |
| Qwen3.6 + Gemma 4 12B | ~25 GB | Lightweight dual-model chat + vision |
| Scout + Devstral Small 2 | ~82 GB | Orchestrated multi-file coding |
| Mistral Medium 3.5 + Qwen3.6 | ~81 GB | Quality-first coding + fast judge (loads within 128 GB) |

All pairs fit within the 128 GB envelope, leaving headroom for KV-cache.

### What to Exclude from Local Use

| Model | Reason |
|-------|--------|
| Kimi K3 (2.8T total) | ~700+ GB even at 2-bit; no GGUF/Ollama as of 2026-07-21 |
| Ornith-1.0-397B (397B total) | Q4 ≈ 200 GB (OOM); Q2 ≈ 100 GB fits but is untested/quality-degraded |
| Llama 4 Maverick (400B total) | Q4 ≈ 200 GB — hard OOM |
| Kimi K2.7 Code (1T total) | Community GGUF exists but ~585 GB at Q4 |
| Devstral 2 full (123B) | Fits solo (~65 GB) but leaves < 10 GB for KV-cache at long context; borderline |

---

## 6. Verification Checklist

```bash
# Ensure Ollama is up-to-date (v0.32.1 as of 2026-07-21)
ollama version

# --- Pull core models ---
ollama pull qwen3.6:27b              # ~17 GB  — generalist / judge / debugger fallback
ollama pull llama4:scout              # ~67 GB  — orchestrator + vision
ollama pull deepseek-r1:32b           # ~20 GB  — reasoning / debugging
ollama pull devstral-small-2          # ~15 GB  — code implementation (battle-tested)
ollama pull laguna-xs-2.1:q4_K_M      # ~20 GB  — code implementation (higher SWE-bench)
ollama pull gemma4:12b                # ~8 GB   — fast general
ollama pull gemma4:e4b                # ~10 GB  — small vision/audio/video

# --- Optional: quality-first coding (slow but higher SWE-bench) ---
# ollama pull mistral-medium-3.5:128b  # ~64 GB  — 77.6% SWE-bench but ~9–12 tok/s

# --- Verify resident memory after load ---
# (run `ollama ps` while the model is active)
# qwen3.6:27b           → 16–18 GB
# llama4:scout          → 65–70 GB
# deepseek-r1:32b       → 19–21 GB
# devstral-small-2      → 14–16 GB
# laguna-xs-2.1:q4_K_M  → 18–21 GB
# gemma4:12b            → 7–9 GB
# gemma4:e4b            → 9–11 GB
# mistral-medium-3.5:128b → 62–68 GB

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

# Agentic coding (Devstral Small 2)
ollama run devstral-small-2 "Implement a Python function that validates an email address using a regex."

# Agentic coding alt (Laguna XS 2.1)
ollama run laguna-xs-2.1:q4_K_M "Refactor this function to handle None inputs gracefully: def get_len(s): return len(s)"

# Vision (Gemma 4 E4B) — requires an image file
# ollama run gemma4:e4b "Describe what you see in this image." --image /path/to/screenshot.png

# Document understanding (Gemma 4 26B MoE)
ollama pull gemma4:26b                # ~18 GB  — document understanding
ollama run gemma4:26b "Extract all column headers and row values from this table image." --image /path/to/table.png
```

---

## 7. Sources

### Platform & Release Notes
- [Ollama July 2026: v0.32.0 Update + Best Models by Use Case — PromptQuorum](https://www.promptquorum.com/local-llms/top-open-source-models-ollama)
- [Ollama Release Notes July 2026 — Releasebot](https://releasebot.io/updates/ollama)
- [Ollama Latest Version (v0.32.1) + Version History — Local AI Master](https://localaimaster.com/blog/ollama-version-history)
- [Ollama in 2026: From Local Runner to AI Platform — Angelo Lima](https://angelo-lima.fr/en/ollama-2026-state-of-the-art-en/)
- [Ollama Library](https://ollama.com/library)

### Apple Silicon Benchmarks
- [Apple Silicon LLM Benchmarks 2026 — LLMCheck](https://llmcheck.net/benchmarks)
- [M5 Max for Local AI: Complete Benchmark Guide 2026 — LLMCheck](https://llmcheck.net/blog/apple-silicon-m5-max-local-ai-guide/)
- [M5 Pro vs M5 Max 2026: Benchmark tok/s Comparison — PromptQuorum](https://www.promptquorum.com/local-llms/m5-pro-max-llm-benchmarks-2026)
- [Local LLM Tokens-per-Second Benchmarks 2026 — Presenc AI](https://presenc.ai/research/local-llm-tokens-per-second-benchmarks-2026)
- [Apple M5 Max Local LLM: 128GB Inference Guide 2026 — AI Productivity](https://aiproductivity.ai/blog/apple-m5-max-local-llm-guide/)
- [Apple M5 Max for Local LLMs: First Benchmarks vs RTX Pro 6000 and RTX 5090 — Hardware Corner](https://www.hardware-corner.net/m5-max-local-llm-benchmarks-20261233/)
- [Exploring LLMs with MLX and Neural Accelerators in the M5 GPU — Apple Machine Learning Research](https://machinelearning.apple.com/research/exploring-llms-mlx-m5)
- [70B Models on M5 Max 128GB — PromptQuorum](https://www.promptquorum.com/local-llms/running-70b-models-apple-silicon-m5-max)
- [Choosing an On-Device LLM Runtime on Apple Silicon — Medium](https://medium.com/@michael.hannecke/choosing-an-on-device-llm-runtime-on-apple-silicon-a-decision-framework-beyond-benchmarks-2449067b8b67)

### Kimi K3 (new this refresh)
- [Kimi K3: 2.8T MoE — Specs, Release Date, Local Reality Check — Local AI Master](https://localaimaster.com/models/kimi-k3)
- [Kimi K3: World's First Open 2.8T Parameter AI Model — Labellerr](https://www.labellerr.com/blog/kimi-k3-world-first-open-2-8t-ai-model/amp/)
- [Kimi K3 Benchmarks, Pricing & Speed (July 2026) — BenchLM.ai](https://benchlm.ai/models/kimi-3)
- [Kimi K3, and what we can still learn from the pelican benchmark — Simon Willison](https://simonwillison.net/2026/Jul/16/kimi-k3/)
- [Kimi K3 Open Weights: Guide for Regulated Business — Layer3Labs](https://www.layer3labs.io/open-weights/kimi-k3-open-weights-guide)
- [What Is Kimi K3? Moonshot's 2.8T, 1M-Context Flagship — Kie.ai](https://kie.ai/blog/what-is-kimi-k3)
- [Kimi K3 Launch and Coding Benchmarks — DevCove](https://devcove.dev/en/articles/kimi-k3-release-and-coding-benchmarks/)

### Ornith-1.0-397B (new this refresh)
- [Ornith 1.0 Models — 9B vs 35B vs 397B Comparison](https://ornith.site/models/)
- [Ornith 1.0-397B: landing page, specs, benchmarks, deployment guide](https://ornith.online/ornith-1-0-model-397b)
- [unsloth/Ornith-1.0-397B-GGUF — Hugging Face](https://huggingface.co/unsloth/Ornith-1.0-397B-GGUF)
- [deepreinforce-ai/Ornith-1.0-397B — Hugging Face](https://huggingface.co/deepreinforce-ai/Ornith-1.0-397B)

### Mistral Medium 3.5 (new this refresh)
- [Mistral Medium 3.5: 128B Dense, 4-GPU Setup (2026) — Local AI Master](https://localaimaster.com/models/mistral-medium-3-5)
- [How to Run Mistral Medium 3.5 Locally — AIMadeTools](https://www.aimadetools.com/blog/how-to-run-mistral-medium-3-5-locally/)
- [mistral-medium-3.5:128b — Ollama](https://ollama.com/library/mistral-medium-3.5:128b)
- [Mistral Medium 3.5 — Mistral Docs](https://docs.mistral.ai/models/model-cards/mistral-medium-3-5-26-04)
- [Let's Data Science: Mistral Medium 3.5 — Let's Data Science](https://letsdatascience.com/blog/mistral-medium-3-5-128b-open-weight-merged-model)

### Qwen Models
- [Qwen3.6-27B: Flagship-Level Coding in a 27B Dense Model — Qwen Blog](https://qwen.ai/blog?id=qwen3.6-27b)
- [Qwen3.6-27B Review — Local AI Master](https://localaimaster.com/models/qwen-3-6-27b)
- [Qwen3.6-27B VRAM Requirements — Will It Run AI](https://willitrunai.com/blog/qwen-3-6-27b-vram-requirements)
- [Qwen3.6-27B Tags — Ollama](https://ollama.com/library/qwen3.6/tags)

### Devstral / Mistral
- [Introducing Devstral 2 and Mistral Vibe CLI — Mistral AI](https://mistral.ai/news/devstral-2-vibe-cli/)
- [Devstral Small 2 — Ollama](https://ollama.com/library/devstral-small-2)
- [Introducing Mistral OCR 4 — Nexus AI Blog](https://nexus-ai-blog.com/en/article/introducing-mistral-ocr-4-a-new-era-for-mqyt85vx)

### Laguna XS 2.1
- [Introducing Laguna XS 2.1 — Poolside](https://poolside.ai/blog/introducing-laguna-xs-2-1)
- [Laguna XS 2.1 on Ollama](https://ollama.com/library/laguna-xs-2.1)
- [Laguna XS 2.1 on Hugging Face](https://huggingface.co/poolside/Laguna-XS-2.1)

### Llama 4 / Meta
- [The Llama 4 Herd — Meta AI Blog](https://ai.meta.com/blog/llama-4-multimodal-intelligence/)
- [Llama 4 Guide: Running Scout and Maverick Locally — InsiderLLM](https://insiderllm.com/guides/llama-4-guide-scout-maverick/)
- [Llama4 Tags — Ollama](https://ollama.com/library/llama4/tags)

### Gemma 4 / Google
- [Gemma 4: Specs, Benchmarks, and Local Guide — Auriga IT](https://aurigait.com/blog/gemma-4-features-benchmarks-guide/)
- [Gemma4 Tags — Ollama](https://ollama.com/library/gemma4/tags)

### Benchmarks & Leaderboards
- [SWE-bench Verified Leaderboard — BenchLM.ai](https://benchlm.ai/benchmarks/sweVerified)
- [SWE-bench Verified — Vals.ai](https://www.vals.ai/benchmarks/swebench)
- [SWE-bench Pro Leaderboard 2026 — Morph](https://www.morphllm.com/swe-bench-pro)
- [Best LLM for Coding (2026): 12 Models Ranked — Morph](https://www.morphllm.com/best-ai-model-for-coding)
- [AI Coding Benchmark Leaderboard 2026 — CodeSOTA](https://www.codesota.com/code-generation)
- [Best Ollama Models (July 2026): Ranked + Pull Commands — BenchLM.ai](https://benchlm.ai/best/ollama-models)
- [Best LLM for Coding July 2026 — BenchLM.ai](https://benchlm.ai/coding)
- [Terminal-Bench Leaderboard — LLM Stats](https://llm-stats.com/benchmarks/terminal-bench)
- [Terminal-Bench 2.0 — BenchLM.ai](https://benchlm.ai/benchmarks/terminalBench2)
- [MCP-Mark Leaderboard — LLM Stats](https://llm-stats.com/benchmarks/mcp-mark)
- [RewardBench 2: ICLR 2026 Paper](https://arxiv.org/pdf/2506.01937)
- [Best LLM Judge Models in 2026 — FutureAGI](https://futureagi.com/blog/best-llm-judge-models-2026/)

### GLM-5.2 / Z.ai
- [GLM-5.2 beats GPT-5.5 on long-horizon coding benchmarks — VentureBeat](https://venturebeat.com/technology/z-ais-open-weights-glm-5-2-beats-gpt-5-5-on-multiple-long-horizon-coding-benchmarks-for-1-6th-the-cost)
- [Unsloth Quantizes GLM-5.2's 1.51TB to 217GB for Local Inference — AI Weekly](https://aiweekly.co/alerts/unsloth-quantizes-glm-52s-151tb-to-217gb-for-local-inference)
- [Run GLM-5.2 Locally: Ollama, VRAM & Hardware Guide — GLM5.app](https://glm5.app/blog/how-to-run-glm-5-2-locally)

### MiniMax M3
- [MiniMax M3 Open Weights Are Live — Nerova](https://nerova.ai/news/minimax-m3-open-weight-agent-builders-june-2026)
- [MiniMax M3 Local AI Hardware Guide 2026 — RunAIHome](https://www.runaihome.com/blog/minimax-m3-local-ai-vram-hardware-guide-2026/)
- [MiniMax M3 — Morph](https://www.morphllm.com/minimax-m3)
- [Best Open Source LLMs July 2026 Leaderboard — BenchLM.ai](https://benchlm.ai/best/open-source)

### DeepSeek
- [DeepSeek V4 Pro Benchmarks, Pricing & Speed — BenchLM.ai](https://benchlm.ai/models/deepseek-v4-pro)
- [DeepSeek-V4-Pro — Ollama](https://ollama.com/library/deepseek-v4-pro)
- [DeepSeek V4 Released — TechCrunch](https://techcrunch.com/2026/04/24/deepseek-previews-new-ai-model-that-closes-the-gap-with-frontier-models/)

### Kimi K2 / Kimi K2.7 Code
- [Kimi K2.7 Code on Ollama](https://ollama.com/library/kimi-k2.7-code)
- [unsloth/Kimi-K2.7-Code-GGUF — Hugging Face](https://huggingface.co/unsloth/Kimi-K2.7-Code-GGUF)
- [Every Kimi AI Model Explained (Jul 2026) — Second Talent](https://www.secondtalent.com/resources/every-kimi-ai-model-explained-compared/)

### NVIDIA Nemotron 3 Super
- [Nemotron 3 Super Technical Report — NVIDIA Research (arXiv)](https://arxiv.org/pdf/2604.12374)
- [NVIDIA Nemotron 3 Super — Ollama Blog](https://ollama.com/blog/nemotron-3-ultra)

### Cohere
- [Meet North Mini Code: Cohere's 30B Open-Weight MoE Model — MarkTechPost](https://www.marktechpost.com/2026/06/11/meet-north-mini-code-coheres-30b-open-weight-mixture-of-experts-model-with-3b-active-parameters-for-agentic-coding/)
- [Cohere North Mini Code 1.0: Open 30B Coding Model Guide — Codersera](https://codersera.com/blog/cohere-north-mini-code-guide-2026/)

### OpenAI gpt-oss (noted, predates window)
- [Introducing gpt-oss — OpenAI](https://openai.com/index/introducing-gpt-oss/)
- [OpenAI gpt-oss — Ollama Blog](https://ollama.com/blog/gpt-oss)

### Reference
- [Open Source LLM Comparison Table 2026 — ComputingForGeeks](https://computingforgeeks.com/open-source-llm-comparison/)
- [Ollama VRAM Requirements 2026 — Local AI Master](https://localaimaster.com/blog/ollama-model-ram-vram-table)
- [GGUF Quantization Guide — Easton Blog](https://eastondev.com/blog/en/posts/ai/20260422-ollama-gguf-quantization/)
- [Best Open-Source LLMs July 2026 Leaderboard — Techsy.io](https://techsy.io/en/blog/best-open-source-llms-2026)
- [Best Open-Source AI Models 2026 — TechJack Solutions](https://techjacksolutions.com/ai-tools/open-source/best-open-source-ai-models/)
- [Best Open Source LLMs for Agentic Coding 2026 — MindStudio](https://www.mindstudio.ai/blog/best-open-source-llms-agentic-coding-2026)
