# Ollama Models Research — 2026-07 Refresh

**Generated:** 2026-07-18  
**Hardware target:** Apple M5 Max · 128 GB unified memory · 614 GB/s bandwidth  
**Research window:** Last 60 days (2026-05-18 → 2026-07-18)

> **Diffs from previous refresh (see commit `c37448d`, 2026-07-18):** No role
> picks changed. This pass re-ran the "cover Z.ai/GLM, MiniMax, gpt-oss"
> requirement from the original research scope, which the first three
> passes had missed entirely, plus checked NVIDIA Nemotron and Cohere. Four
> real models were added to §2 that were absent from every prior version of
> this doc:
> 1. **GLM-5.2** (Z.ai, Jun 13, 2026) — 744B/40B-active MoE, beats GPT-5.5
>    on several coding benchmarks — but genuinely doesn't fit 128 GB even at
>    1–2 bit quantization (~217 GB minimum), and Ollama's only tag for it is
>    `:cloud` (routes to Z.ai's infra, not local inference). Added to §1's
>    "Cloud-Only" table and §2.
> 2. **MiniMax M3** (MiniMax, Jun 1, 2026) — currently #1 on the BenchLM
>    open-source leaderboard (Jul 17, 2026), beats GPT-5.5/Gemini 3.1 Pro on
>    SWE-bench Pro — but the smallest usable quant is ~143 GB (Q2), still
>    over budget, and Ollama's only tag is also `:cloud`. Added to §1 and §2.
> 3. **NVIDIA Nemotron 3 Super** (Mar 11, 2026) — 120B/12B-active hybrid
>    Mamba-Transformer, ~60 GB at Q4 (fits comfortably), 1M context, 2.2x
>    GPT-OSS-120B throughput — but scores 60.47% SWE-bench Verified, below
>    every current top/smaller pick in the code-implementer and
>    plan-orchestrator roles. Added to §2 and §3.4 as an evaluated
>    alternative.
> 4. **Cohere North Mini Code 1.0** (Cohere, Jun 11, 2026) — 30B/3B-active
>    MoE (same size class as Laguna XS 2.1), real Ollama tag
>    (`north-mini-code-1.0`), 67.6% SWE-bench Verified — below Laguna XS
>    2.1's 70.9%. Added to §2 and §3.2 as an evaluated alternative.
>
> All four were assessed against every relevant role's current top/smaller
> pick; none changed a recommendation — two don't fit the 128 GB envelope at
> any practical quant, two fit but score lower on directly comparable
> benchmarks. See `OPERATIONS.md` for the broader-sweep context; the prior
> commit's corrections (tag/size fixes) are unaffected by this pass.

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

| Model | Architecture | Quant | tok/s llama.cpp | tok/s MLX |
|-------|-------------|-------|-----------------|----------|
| Llama 3 8B | Dense 8B | Q4_K_M | ~82 | ~230 |
| Qwen3.6-27B | Dense 27B | Q4_K_M | ~45 | ~70 |
| Qwen 3.5 30B-A3B | MoE 30B/3B active | Q4_K_M | ~45 | ~58 |
| Laguna XS 2.1 | MoE 33B/3B active | Q4_K_M | ~43 | ~56 |
| DeepSeek-R1-Distill 32B | Dense 32B | Q4_K_M | ~35 | ~60 |
| Llama 4 Scout | MoE 109B/17B active | Q4_K_M | ~30 | ~50 |
| Llama 3 70B | Dense 70B | Q4_K_M | ~18 | ~28 |

Sources: [LLMCheck Apple Silicon Benchmarks](https://llmcheck.net/benchmarks) · [PromptQuorum M5 Max vs M4 Max](https://www.promptquorum.com/local-llms/m5-pro-max-llm-benchmarks-2026) · [Presenc AI Local Benchmarks 2026](https://presenc.ai/research/local-llm-tokens-per-second-benchmarks-2026)

### Runtime Notes

- **MLX vs llama.cpp:** Apple's native MLX framework delivers 40–80% higher throughput than llama.cpp on Apple Silicon. Ollama v0.32.0 (Jul 11, 2026) now bundles an MLX engine for Apple Silicon — use it for maximum speed. `mlx_lm` directly gives the most headroom for speed-critical workflows.
- **MoE-A3B behaviour:** MoE models activate only ~3B parameters per token despite loading much larger full weight sets, giving 30B-class quality at high tok/s. Qwen 3.5 30B-A3B loads ~17 GB at Q4; Laguna XS 2.1 loads ~20 GB at Q4_K_M. This remains the highest quality-per-GB pattern available in Ollama today.
- **Ollama v0.32.0 uplift:** Gemma 4 runs ~90% faster in this release due to multi-token prediction (MTP) and automatic hardware tuning. Pull the latest Ollama before benchmarking any Gemma 4 variant.
- **KV-cache headroom:** At 128 GB, plan for ~10–18 GB reserved for macOS + KV-cache at long context. Practical ceiling for a single loaded model is ~110 GB — comfortably above even Llama 4 Scout's corrected 67 GB footprint.
- **M5 Max vs M4 Max:** The M5 Max delivers ~28% higher tok/s across all model sizes vs M4 Max (600 GB/s → 614 GB/s bandwidth, improved GPU, redesigned Neural Engine).

### What Exceeds 128 GB (Cloud-Only)

| Model | Why it won't fit |
|-------|------------------|
| Llama 4 Maverick (400B total) | Q4 ≈ 200 GB — exceeds 128 GB envelope |
| Kimi K2.7 Code (1T parameters) | Community GGUF builds exist (Unsloth, mradermacher) but even Q4 (UD-Q4_K_XL) is ~585 GB — 4–5x the 128 GB envelope; no first-class Ollama library tag |
| **GLM-5.2** (Z.ai, 744B/40B active MoE) | Beats GPT-5.5 on several coding benchmarks, but even 1–2 bit GGUF quants are ~217–238 GB; Ollama's only tag is `:cloud` (Z.ai-hosted, not local) |
| **MiniMax M3** (MiniMax, ~230–428B/9.8B active MoE) | #1 on the BenchLM open-source leaderboard as of Jul 17, 2026, but the smallest usable local quant (UD-Q2_K_XL) is ~143 GB; Ollama's only tag is `:cloud` |
| DeepSeek-V3 full (671B MoE, 37B active) | Q4 ≈ 170 GB — borderline, likely OOM with KV-cache |
| Any 700B+ total-parameter model | Exceeds envelope at any practical quant |

---

## 2. Current Model Landscape (Last 60 Days)

### Models that moved SOTA (May 18 – Jul 18, 2026)

| Model | Family | Released | Why it matters |
|-------|--------|----------|----------------|
| **Laguna XS 2.1** | Poolside (MoE 33B/3B active) | Jul 2, 2026 | 70.9% SWE-bench Verified; highest-scoring Ollama-**locally-runnable** model per GB in its size tier; Apache 2.0 |
| **GLM-5.2** | Z.ai (MoE 744B/40B active) | Jun 13, 2026 | 62.1 SWE-bench Pro (vs GPT-5.5's 58.6), 91.2 GPQA-Diamond, MIT license — genuinely SOTA, but doesn't fit 128 GB at any practical quant (~217 GB+) and Ollama only offers `:cloud`. **Cloud-only for this hardware — see §1.** |
| **MiniMax M3** | MiniMax (MoE, 9.8B active) | Jun 1, 2026 | #1 on the BenchLM open-source leaderboard (Jul 17, 2026); beats GPT-5.5/Gemini 3.1 Pro on SWE-bench Pro and BrowseComp — but smallest usable local quant is ~143 GB and Ollama only offers `:cloud`. **Cloud-only for this hardware — see §1.** |
| **Kimi K2.7 Code** | Moonshot AI (1T MoE) | Jun 2026 | 30% lower thinking-token usage vs K2.6; community GGUF builds now exist but model is still cloud-only at ~585 GB even at Q4 |
| **Cohere North Mini Code 1.0** | Cohere (MoE 30B/3B active) | Jun 11, 2026 | 67.6% SWE-bench Verified, Apache 2.0, real Ollama tag (`north-mini-code-1.0`) — same size class as Laguna XS 2.1 but scores lower; genuinely fits and pulls locally, just not the top pick |
| **Gemma 4 12B Unified** | Google (dense 12B) | Jun 2026 | Fills mid-size gap in Gemma lineup; natively multimodal; 77.2% MMLU Pro |
| **Ollama v0.32.0** | Platform release | Jul 11, 2026 | Native MLX engine on Apple Silicon; agentic "code + delegate" mode; Gemma 4 ~90% faster |

### Notable model evaluated just outside the 60-day window

| Model | Family | Released | Why it matters |
|-------|--------|----------|----------------|
| **NVIDIA Nemotron 3 Super** | NVIDIA (hybrid Mamba-Transformer, 120B/12B active) | Mar 11, 2026 | ~60 GB at Q4 (fits comfortably), 1M context, 2.2x GPT-OSS-120B throughput — but 60.47% SWE-bench Verified is below every current top/smaller pick in the relevant roles. Evaluated for plan-orchestrator, not selected — see §3.4. |

OpenAI's **gpt-oss** (20B/120B, MoE, released Aug 2025) was also checked per the original research scope, which names it explicitly — it predates the 60-day window by nearly a year and none of its benchmark figures found here beat current picks in any role, so it isn't tabled above. Still cited in §7 for completeness since it was explicitly in scope.

### Context: models released just outside 60-day window but shaping current picks

| Model | Released | Relevance |
|-------|----------|-----------|
| Qwen3.6-27B | Apr 22, 2026 | 77.2% SWE-bench; replaces Qwen3-30B-A3B as the everyday dense recommendation |
| Gemma 4 (E2B/E4B/26B-MoE/31B) | Apr 2, 2026 | All variants natively multimodal; Gemma 4 26B-MoE is the document-understanding pick |
| Llama 4 Scout (109B MoE) | Apr 2026 | 10M-token context window; only locally-runnable model at that scale |
| Devstral Small 2 (24B) | 2025-12/2026-01 | Still the most-tested open agentic coding model; Apache 2.0 |

---

## 3. Recommendations by Role

### 3.1 Generalist Agentic Default
*One model that does most jobs adequately.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | Qwen3.6-27B | ~17 GB | SWE-bench Verified 77.2%; Terminal-Bench 2.0 59.3%; 256K ctx | `qwen3.6:27b` |
| **Smaller pick** | Gemma 4 12B Unified | ~8 GB | MMLU Pro 77.2%; natively multimodal (text + image) | `gemma4:12b` |

**Rationale:** Qwen3.6-27B (Alibaba, Apr 22, 2026) is a 27B *dense* model with a built-in dual-mode thinking toggle: `/think` on for deep reasoning, off for fast chat. At 77.2% SWE-bench Verified it sits within 3.7 points of Claude Opus 4.6 (80.8%) while running entirely locally. The 256K context window handles most agentic sessions without truncation. Gemma 4 12B Unified is the right answer when VRAM is at a premium or when multimodal input is needed at low cost.

**Note on Qwen3.6-27B sizing:** ~16.8 GB at Q4_K_M; ~19.5 GB at Q5_K_M; ~28.6 GB at Q8_0. Q5_K_M is recommended if the extra 3 GB is available — marginal quality uplift compounds in agentic loops.

### 3.2 Code Implementer
*Writes code, multi-file edits, agentic coding loops.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | Laguna XS 2.1 | ~20 GB | SWE-bench Verified 70.9%; SWE-bench Multilingual 63.1%; 256K ctx | `laguna-xs-2.1:q4_K_M` |
| **Smaller pick** | Devstral Small 2 | ~15 GB | SWE-bench Verified 68.0%; 256K ctx; Apache 2.0 | `devstral-small-2` |

**Rationale:** Laguna XS 2.1 (Poolside, Jul 2, 2026) is a 33B/3B-active MoE that scores 70.9% on SWE-bench Verified — the highest of any Ollama-available model in its memory tier as of July 2026. MoE sparsity keeps generation fast (~56 tok/s via MLX) while the 256K context window handles large multi-file codebases. Devstral Small 2 (Mistral, 24B dense) is nearly as capable at 68.0% and is more production-battle-tested in agentic scaffolds (OpenHands, SWE-agent). Devstral 2 full (123B, 72.2% SWE-bench) is the strongest open-weight coder but requires ~65 GB Q4 — it fits in 128 GB alone but leaves little room for concurrent models.

*(Corrected 2026-07-18: the Ollama tag is `devstral-small-2`, not `devstral-2:small` — the latter does not exist and would fail to pull. See the diffs callout above.)*

**Also evaluated, not selected:** Cohere North Mini Code 1.0 (Jun 11, 2026; 30B/3B-active MoE, 256K ctx, Apache 2.0, `ollama pull north-mini-code-1.0`) — same MoE-A3B size class as Laguna XS 2.1 but scores 67.6% SWE-bench Verified vs Laguna's 70.9%. GLM-5.2 and MiniMax M3 currently top coding leaderboards generally but don't fit 128 GB at any practical quant — see §1 and §2.

### 3.3 Code Debugger
*Reasoning / chain-of-thought, root-cause analysis, math-heavy debugging.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | DeepSeek-R1-Distill-Qwen-32B | ~20 GB | MMLU 83%; MATH 72% (highest of any sub-40 GB local model) | `deepseek-r1:32b` |
| **Smaller pick** | Qwen3.6-27B (thinking mode on) | ~17 GB | SWE-bench 77.2%; switchable CoT; same model as §3.1 | `qwen3.6:27b` |

**Rationale:** DeepSeek-R1-Distill-Qwen-32B provides always-on chain-of-thought reasoning (no toggle needed) and holds the highest MATH benchmark score of any locally-runnable model under 40 GB. It was distilled from the 671B R1 teacher, capturing systematic reasoning behaviour in a 20 GB package. Use it for debugging sessions that require step-by-step trace through a logic fault or complex algorithm. Qwen3.6-27B with `/think` enabled is the pragmatic alternative if you want to avoid loading a second model — at 77.2% SWE-bench it is strong, though the MATH ceiling is somewhat lower.

### 3.4 Plan Orchestrator
*Long context + reliable tool calling + structured output for multi-agent coordination.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | Llama 4 Scout | ~67 GB | 10M-token context; 17B active / 109B total MoE; natively multimodal; strong tool calling | `llama4:scout` |
| **Smaller pick** | Qwen3.6-27B | ~17 GB | 256K context; reliable structured output; strong tool fidelity | `qwen3.6:27b` |

**Rationale:** Llama 4 Scout (Meta, Apr 2026) offers the longest locally-runnable context window in existence at 10M tokens — orders of magnitude beyond any competitor. Its 17B active-parameter MoE architecture runs at ~50 tok/s via MLX on M5 Max, and the natively integrated vision encoder lets the orchestrator ingest diagrams and screenshots without a separate VLM call. At ~67 GB Q4 it still fits alongside a 17 GB model (e.g., Qwen3.6-27B) with room to spare. Qwen3.6-27B covers 256K context at roughly a quarter the size and is adequate for most orchestration pipelines that don't require mega-context.

*(Resident size corrected 2026-07-18: 55 GB → 67 GB, per the official Ollama tags page — see the diffs callout above.)*

**Also evaluated, not selected:** NVIDIA Nemotron 3 Super (Mar 11, 2026; 120B/12B-active hybrid Mamba-Transformer, 1M context, ~60 GB at Q4) — fits comfortably alongside another model and offers 2.2x GPT-OSS-120B throughput, but scores 60.47% SWE-bench Verified, well below Qwen3.6-27B and not directly comparable to Scout's orchestration-specific strengths (10M context, native vision). Worth revisiting if a future role specifically needs high-concurrency multi-agent throughput over raw context length.

### 3.5 LLM-as-Judge / Verifier
*Calibrated scoring against rubrics, pairwise preference evaluation, output verification.*

| | Model | Q4 resident | Key benchmarks | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | Qwen3.6-27B | ~17 GB | Strong instruction following; consistent in non-thinking mode; 256K ctx | `qwen3.6:27b` |
| **Smaller pick** | Gemma 4 12B Unified | ~8 GB | Fast; MMLU Pro 77.2%; low latency for high-volume scoring | `gemma4:12b` |

**Rationale:** For judging, **disable thinking mode** — reasoning traces inflate token cost and can introduce self-consistency bias in scores. Qwen3.6-27B's instruction fidelity and calibration make it the recommended local judge. RewardBench 2 (ICLR 2026) demonstrated that strong instruction-following generalizes to judge quality; Qwen3.6 performs well on this axis. Inject a scored reference example at the top of the judge prompt to anchor the scoring scale (calibration injection). **Important caveat:** For high-stakes judging pipelines (e.g., RLHF reward labeling), frontier models (Claude Opus 4.8, GPT-5.5) remain preferred. Reserve local judges for cost-sensitive or privacy-constrained pipelines.

### 3.6 Document Understanding / Interpreter
*PDFs, OCR, table and equation extraction.*

| | Model | Q4 resident | Key capability | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | Gemma 4 26B MoE (E26B-A4B) | ~18 GB | Natively multimodal; strong table/equation/diagram extraction; 89% AIME 2026 | `gemma4:26b` |
| **Smaller pick** | Mistral OCR 4 | ~6 GB | Purpose-built OCR; structured Markdown/JSON output; best for high-throughput privacy-preserving pipelines | *(check `ollama search mistral-ocr`)* |

**Rationale:** Gemma 4's 26B MoE variant (Google, Apr 2026) understands images, text, tables, and LaTeX natively in a single forward pass — no separate OCR pre-processing needed. It handles complex PDF layouts, nested tables, and mixed-language documents reliably. For dedicated high-throughput pipelines (batch invoice processing, form extraction), Mistral OCR 4 is purpose-tuned and outputs clean structured Markdown or JSON — it outperforms generalists on this narrow domain. Also consider IBM Granite 3.2 Vision for structured form/invoice extraction.

*(Resident size corrected 2026-07-18: 15 GB → 18 GB, per the official Ollama tags page — see the diffs callout above.)*

### 3.7 Vision / Image Understanding
*VLMs for image comprehension only — not generation.*

| | Model | Q4 resident | Key capability | Ollama tag |
|-|-------|------------|----------------|------------|
| **Top pick** | Llama 4 Scout | ~67 GB | Natively integrated vision (not bolt-on); coherent multi-image + long-text reasoning | `llama4:scout` |
| **Smaller pick** | Gemma 4 E4B | ~10 GB | Natively multimodal (image, audio, video); smallest of the resident models in this doc; best tiny VLM available | `gemma4:e4b` |

**Rationale:** Llama 4 Scout integrates vision natively into the base architecture (not via a separate LLaVA-style encoder adapter), resulting in more coherent multi-modal reasoning when combining images with long context. Supports up to 5 input images per prompt. Gemma 4 E4B is the go-to when you need a capable VLM in a small footprint — it also accepts audio and video natively, something no other model in this doc offers.

*(Corrected 2026-07-18: resident size 5 GB → ~10 GB, and the earlier "fits in 6 GB" claim removed as inaccurate — see the diffs callout above.)*

---

## 4. The "Sufficient but Smaller" Angle

Three patterns consistently pay off on M5 Max 128 GB:

### 4.1 MoE-A3B Models
Models like **Laguna XS 2.1** (33B/3B active, ~20 GB at Q4_K_M) and **Qwen 3.5 30B-A3B** (30B/3B active, ~17 GB at Q4) load a much larger full weight set but activate only ~3B parameters per token during inference. The result: 30B-class output quality at ~56–58 tok/s on M5 Max — faster than a dense 27B at the same quant, with quality competitive above its weight class. This is the highest quality-per-GB pattern in the current Ollama library.

### 4.2 Distilled Reasoning Models
**DeepSeek-R1-Distill-Qwen-32B** transfers chain-of-thought behaviour from a 671B teacher into a 32B student (20 GB at Q4). It achieves MATH scores that beat many naive 70B models, at twice the speed and half the memory. For debugging and reasoning tasks, the distill is sufficient and the correct choice — you don't need to reach for a 70B.

### 4.3 Purpose-Built Specialized Small Models
A 6–10 GB specialist routinely beats a 70B generalist on its target domain. Examples:
- **Mistral OCR 4** (~6 GB): purpose-tuned OCR with structured output — more reliable for batch document processing than prompting Qwen3.6-27B
- **Gemma 4 E4B** (~10 GB): best small VLM; handles image/audio/video natively
- **nomic-embed-text**: best-in-class embeddings for RAG pipelines; negligible resident memory

General principle: profile which tasks consume the most tokens in your workflow, then identify if a specialist exists. The 128 GB envelope lets you keep several specialists resident simultaneously.

---

## 5. Suggested Concrete Stack

Ollama loads and unloads models from unified memory on demand. The full inventory below fits in 128 GB; the "concurrent pair" examples show what stays live together without eviction.

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

**Total inventory size on disk:** ~157 GB (can all be pulled; Ollama evicts from RAM as needed). *(Corrected from ~139 GB — largely driven by the Llama 4 Scout and Gemma 4 E4B sizing fixes.)*

### Recommended Concurrent Pairs

| Pair | Combined RAM | Use case |
|------|-------------|----------|
| Scout + Qwen3.6 | ~84 GB | Orchestrator + workhorse; long-context agentic sessions |
| Devstral Small 2 + DeepSeek-R1 32B | ~35 GB | Coding session: implement → debug loop |
| Qwen3.6 + Gemma 4 12B | ~25 GB | Lightweight dual-model chat + vision |
| Scout + Devstral Small 2 | ~82 GB | Orchestrated multi-file coding |

All pairs still fit comfortably within the 128 GB envelope after the sizing corrections — no pair recommendation changed, only the totals.

### What to Exclude from Local Use

| Model | Reason |
|-------|--------|
| Llama 4 Maverick (400B total) | Q4 ≈ 200 GB — hard OOM |
| Kimi K2.7 Code (1T params) | Community GGUF exists but ~585 GB even at Q4 — cloud-only regardless |
| Devstral 2 full (123B) | Fits solo (~65 GB) but leaves < 10 GB for KV-cache at long context; borderline |

---

## 6. Verification Checklist

```bash
# --- Pull models ---
ollama pull qwen3.6:27b            # ~17 GB  — generalist / judge / debugger fallback
ollama pull llama4:scout            # ~67 GB  — orchestrator + vision
ollama pull deepseek-r1:32b         # ~20 GB  — reasoning / debugging
ollama pull devstral-small-2        # ~15 GB  — code implementation
ollama pull laguna-xs-2.1:q4_K_M    # ~20 GB  — code implementation (alt, higher SWE-bench)
ollama pull gemma4:12b              # ~8 GB   — fast general
ollama pull gemma4:e4b              # ~10 GB  — small vision/audio/video

# --- Verify resident memory after load ---
# (run `ollama ps` while the model is active to confirm)
# qwen3.6:27b          → 16–18 GB
# llama4:scout         → 65–70 GB
# deepseek-r1:32b      → 19–21 GB
# devstral-small-2     → 14–16 GB
# laguna-xs-2.1:q4_K_M → 18–21 GB
# gemma4:12b           → 7–9 GB
# gemma4:e4b           → 9–11 GB

# --- Smoke tests ---

# Generalist (Qwen3.6-27B)
ollama run qwen3.6:27b "Write a Python function to merge two sorted lists and return the merged result."

# Thinking mode on (code debugger role)
ollama run qwen3.6:27b "/think Identify the bug in this Python snippet: def fib(n): return fib(n-1) + fib(n-2)"
# Expected: <think>...</think> block then answer noting missing base cases

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
ollama pull gemma4:26b            # ~18 GB  — document understanding
ollama run gemma4:26b "Extract all column headers and row values from this table image." --image /path/to/table.png
```

---

## 7. Sources

- [Ollama July 2026: v0.32.0 Update + Best Models by Use Case — PromptQuorum](https://www.promptquorum.com/local-llms/top-open-source-models-ollama)
- [Ollama Release Notes July 2026 — Releasebot](https://releasebot.io/updates/ollama)
- [Ollama in 2026: From Local Runner to AI Platform — Angelo Lima](https://angelo-lima.fr/en/ollama-2026-state-of-the-art-en/)
- [Apple Silicon LLM Benchmarks 2026 — LLMCheck](https://llmcheck.net/benchmarks)
- [M5 Max for Local AI: Complete Benchmark Guide 2026 — LLMCheck](https://llmcheck.net/blog/apple-silicon-m5-max-local-ai-guide/)
- [M5 Pro vs M5 Max 2026: Benchmark tok/s Comparison — PromptQuorum](https://www.promptquorum.com/local-llms/m5-pro-max-llm-benchmarks-2026)
- [Local LLM Tokens-per-Second Benchmarks 2026 — Presenc AI](https://presenc.ai/research/local-llm-tokens-per-second-benchmarks-2026)
- [Apple M5 Max Local LLM: 128GB Inference Guide 2026 — AI Productivity](https://aiproductivity.ai/blog/apple-m5-max-local-llm-guide/)
- [70B Models on M5 Max 128GB — PromptQuorum](https://www.promptquorum.com/local-llms/running-70b-models-apple-silicon-m5-max)
- [Choosing an On-Device LLM Runtime on Apple Silicon — Medium](https://medium.com/@michael.hannecke/choosing-an-on-device-llm-runtime-on-apple-silicon-a-decision-framework-beyond-benchmarks-2449067b8b67)
- [Qwen3.6-27B: Flagship-Level Coding in a 27B Dense Model — Qwen Blog](https://qwen.ai/blog?id=qwen3.6-27b)
- [Qwen3.6-27B Review — Local AI Master](https://localaimaster.com/models/qwen-3-6-27b)
- [Qwen3.6-27B VRAM Requirements — Will It Run AI](https://willitrunai.com/blog/qwen-3-6-27b-vram-requirements)
- [Qwen3.6-27B Tags — Ollama](https://ollama.com/library/qwen3.6/tags)
- [Introducing Devstral 2 and Mistral Vibe CLI — Mistral AI](https://mistral.ai/news/devstral-2-vibe-cli/)
- [Devstral — Mistral AI](https://mistral.ai/news/devstral/)
- [Devstral Small 2 — Ollama](https://ollama.com/library/devstral-small-2)
- [Devstral Small 2 Tags — Ollama](https://ollama.com/library/devstral-small-2/tags)
- [Devstral Small 2 for Local AI in 2026 — RunAIHome](https://runaihome.com/blog/devstral-small-2-local-ai-hardware-guide-2026/)
- [Introducing Laguna XS 2.1 — Poolside](https://poolside.ai/blog/introducing-laguna-xs-2-1)
- [Laguna XS 2.1 on Ollama](https://ollama.com/library/laguna-xs-2.1)
- [Laguna XS 2.1 Tags — Ollama](https://ollama.com/library/laguna-xs-2.1/tags)
- [Laguna XS 2.1 on Hugging Face](https://huggingface.co/poolside/Laguna-XS-2.1)
- [Kimi K2.7 Code on Ollama](https://ollama.com/library/kimi-k2.7-code)
- [Kimi K2.7 Complete Guide — Codersera](https://codersera.com/blog/kimi-k2-7-complete-guide-2026/)
- [Kimi K2.7 Code GGUF — Unsloth Documentation](https://unsloth.ai/docs/models/kimi-k2.7-code)
- [unsloth/Kimi-K2.7-Code-GGUF — Hugging Face](https://huggingface.co/unsloth/Kimi-K2.7-Code-GGUF)
- [The Llama 4 Herd — Meta AI Blog](https://ai.meta.com/blog/llama-4-multimodal-intelligence/)
- [Llama 4 Guide: Running Scout and Maverick Locally — InsiderLLM](https://insiderllm.com/guides/llama-4-guide-scout-maverick/)
- [Llama4 Tags — Ollama](https://ollama.com/library/llama4/tags)
- [Llama4 Scout — Ollama](https://ollama.com/library/llama4:scout)
- [Gemma 4: Specs, Benchmarks, and Local Guide — Auriga IT](https://aurigait.com/blog/gemma-4-features-benchmarks-guide/)
- [Gemma4 Tags — Ollama](https://ollama.com/library/gemma4/tags)
- [Gemma4 26B — Ollama](https://ollama.com/library/gemma4:26b)
- [Gemma4 E4B — Ollama](https://ollama.com/library/gemma4:e4b)
- [Gemma4 12B — Ollama](https://ollama.com/library/gemma4:12b)
- [Best AI Coding Models Ranked: SWE-bench Leaderboard — Local AI Master](https://localaimaster.com/models/best-ai-coding-models)
- [AI Coding Benchmark Leaderboard 2026 — CodeSOTA](https://www.codesota.com/code-generation)
- [Best LLM for Coding July 2026 — BenchLM.ai](https://benchlm.ai/coding)
- [SWE-bench Verified — Vals.ai](https://www.vals.ai/benchmarks/swebench)
- [Best Local LLMs for Coding — Bodega One Code](https://www.bodegaone.ai/local-llms)
- [Best Open Source Reasoning Model 2026 — Ertas AI](https://www.ertas.ai/best/best-open-source-reasoning-model)
- [Best Local Reasoning LLMs 2026 — LLMRun](https://llmrun.dev/use/reasoning)
- [Qwen3 vs DeepSeek R1: Which Open-Source Reasoning Model? — QAInsights](https://qainsights.com/qwen3-vs-deepseek-r1-which-open-source-reasoning-model/)
- [DeepSeek-R1 Tags — Ollama](https://ollama.com/library/deepseek-r1/tags)
- [Best LLM Judge Models in 2026 — FutureAGI](https://futureagi.com/blog/best-llm-judge-models-2026/)
- [RewardBench 2: ICLR 2026 Paper](https://arxiv.org/pdf/2506.01937)
- [Best Local Vision Models 2026 — PromptQuorum](https://www.promptquorum.com/power-local-llm/local-vision-models-llava-ollama-2026)
- [Best Ollama Vision Models 2026 — Serverman](https://www.serverman.co.uk/ai/ollama/best-ollama-models-for-vision/)
- [Top 10 Vision Language Models in 2026 — DataCamp](https://www.datacamp.com/blog/top-vision-language-models)
- [Local AI Vision Tasks 2026: OCR, Invoices — Local AI Master](https://localaimaster.com/blog/local-ai-vision-tasks)
- [Introducing Mistral OCR 4 — Nexus AI Blog](https://nexus-ai-blog.com/en/article/introducing-mistral-ocr-4-a-new-era-for-mqyt85vx)
- [Mistral OCR — Mistral AI](https://mistral.ai/news/mistral-ocr/)
- [Ollama VRAM Requirements 2026 — Local AI Master](https://localaimaster.com/blog/ollama-model-ram-vram-table)
- [GGUF Quantization Guide — Easton Blog](https://eastondev.com/blog/en/posts/ai/20260422-ollama-gguf-quantization/)
- [Ollama Performance Tuning: KV Cache and OOM — Easton Blog](https://eastondev.com/blog/en/posts/ai/20260410-ollama-performance-optimization/)
- [Open Source LLM Comparison Table 2026 — ComputingForGeeks](https://computingforgeeks.com/open-source-llm-comparison/)
- [Best Ollama Models 2026 — Morph](https://www.morphllm.com/best-ollama-models)
- [The Local LLM Model Guide 2026 — InKeyBit](https://www.inkeybit.com/blog/local-llm-model-guide-2026)
- [GLM-5.2 beats GPT-5.5 on long-horizon coding benchmarks — VentureBeat](https://venturebeat.com/technology/z-ais-open-weights-glm-5-2-beats-gpt-5-5-on-multiple-long-horizon-coding-benchmarks-for-1-6th-the-cost)
- [GLM-5.2: #1 Open Source AI Model? Full Review — BuildFastWithAI](https://www.buildfastwithai.com/blogs/glm-5-1-open-source-review-2026)
- [Unsloth Quantizes GLM-5.2's 1.51TB to 217GB for Local Inference — AI Weekly](https://aiweekly.co/alerts/unsloth-quantizes-glm-52s-151tb-to-217gb-for-local-inference)
- [Run GLM-5.2 Locally: Ollama, VRAM & Hardware Guide — GLM5.app](https://glm5.app/blog/how-to-run-glm-5-2-locally)
- [MiniMax M3 Open Weights Are Live — Nerova](https://nerova.ai/news/minimax-m3-open-weight-agent-builders-june-2026)
- [MiniMax M3 Local AI Hardware Guide 2026 — RunAIHome](https://www.runaihome.com/blog/minimax-m3-local-ai-vram-hardware-guide-2026/)
- [MiniMax M3 — How to Run Locally — Unsloth Documentation](https://unsloth.ai/docs/models/minimax-m3)
- [Best Open Source LLMs July 2026 Leaderboard — BenchLM.ai](https://benchlm.ai/best/open-source)
- [NVIDIA Nemotron 3 Super: 60% SWE-bench, Best Open Model for Code — Abhishek Gautam](https://www.abhs.in/blog/nvidia-nemotron-3-super-swe-bench-60-percent-open-weight-coding-2026)
- [Nemotron 3 Super Technical Report — NVIDIA Research (arXiv)](https://arxiv.org/pdf/2604.12374)
- [NVIDIA Nemotron 3 Super — Ollama Blog](https://ollama.com/blog/nemotron-3-ultra)
- [Meet North Mini Code: Cohere's 30B Open-Weight MoE Model — MarkTechPost](https://www.marktechpost.com/2026/06/11/meet-north-mini-code-coheres-30b-open-weight-mixture-of-experts-model-with-3b-active-parameters-for-agentic-coding/)
- [Cohere North Mini Code 1.0: Open 30B Coding Model Guide — Codersera](https://codersera.com/blog/cohere-north-mini-code-guide-2026/)
- [Introducing gpt-oss — OpenAI](https://openai.com/index/introducing-gpt-oss/)
- [OpenAI gpt-oss — Ollama Blog](https://ollama.com/blog/gpt-oss)
- [gpt-oss Tags — Ollama](https://ollama.com/library/gpt-oss)
