# Commit Message Format — Ollama Models Refresh Routine

Every refresh commit to this repo must use the following structured format.
The short subject line is `refresh: YYYY-MM-DD`, followed by a blank line and
four clearly labeled sections.

---

```
refresh: YYYY-MM-DD

INTENT
<1–3 sentences: what this run set out to do and why. Include the hardware
target, the 60-day research window, and the scope (roles, model families).>

LEARNED
<Bullet list of concrete new findings: new model releases with dates,
benchmark numbers with sources, performance data, platform changes (Ollama
versions), anything that materially updates our understanding.>
- ModelName (Source, Date): key fact
- ...

VALIDATED / DISAVOWED
<What prior assumptions or recommendations held up, and what was proven
wrong or superseded. On the first run, note that no prior baseline exists.>
Validated:
- ...
Disavowed:
- ...

CONCLUSION
<2–5 sentences: the bottom-line output of this run. Which role picks changed
and why. What the reader should act on immediately (new pulls, retirements,
caveats).>
```

---

## Example (2026-07-18 initial run)

```
refresh: 2026-07-18

INTENT
Initial research run to establish baseline Ollama model recommendations for
Apple M5 Max 128 GB (614 GB/s). Covered 7 agentic roles, surveyed the
2026-05-18 – 2026-07-18 window, and sized every candidate against the 128 GB
unified-memory envelope.

LEARNED
- Laguna XS 2.1 (Poolside, Jul 2 2026): 33B/3B-active MoE; 70.9% SWE-bench
  Verified — highest coding score per GB in Ollama library at this date
- Kimi K2.7 Code (Moonshot, Jun 2026): 1T-param model; no GGUF exists yet;
  cloud-only until a quantized build ships
- Gemma 4 12B Unified (Google, Jun 2026): fills mid-size gap; natively
  multimodal; 77.2% MMLU Pro
- Ollama v0.32.0 (Jul 11 2026): ships native MLX engine on Apple Silicon;
  Gemma 4 generation ~90% faster
- Qwen3.6-27B (Alibaba, Apr 2026): 27B dense, 256K ctx, dual-mode thinking,
  77.2% SWE-bench Verified — within 3.7 pts of Claude Opus 4.6
- Llama 4 Scout (Meta, Apr 2026): only locally-runnable 10M-context model;
  17B active / 109B total MoE; ~55 GB Q4 on M5 Max
- DeepSeek-R1-Distill-Qwen-32B: highest MATH score of any sub-40 GB local
  model (72%); always-on CoT at ~20 GB Q4
- MLX delivers 40-80% higher tok/s than llama.cpp on M5 Max (LLMCheck Jul 2026)
- M5 Max ~28% faster than M4 Max across all model sizes (614 vs 546 GB/s bw)

VALIDATED / DISAVOWED
Validated:
- MoE-A3B pattern confirmed efficient: ~17 GB loaded, ~3B active per token,
  quality competitive with dense 30B models at faster inference
- MLX framework advantage confirmed: 40-80% throughput gain over llama.cpp
Disavowed:
- N/A — first run; no prior baseline recommendations to compare against

CONCLUSION
Established a 7-role stack for M5 Max 128 GB. Top picks: Llama 4 Scout
(orchestrator + vision), Qwen3.6-27B (generalist + judge), DeepSeek-R1-Distill
32B (debugger), Laguna XS 2.1 (code implementer, highest SWE-bench per GB),
Gemma 4 12B + E4B (fast/tiny multimodal). Kimi K2.7 Code is cloud-only —
watch for a GGUF build. Laguna XS 2.1 is the breakout new entrant this cycle.
Next refresh due 2026-07-28.
```
