# Monthly Budget: Self-Hosting Qwen3-32B on a GPU VPS

**Date:** 2026-10-11
**Question:** "...and for Qwen3 32 billion."

> [!NOTE] Naming
> The current series is **Qwen3** (there is no "Qwen 3.8"). **Qwen3-32B** is the
> largest *dense* model in the family and its practical flagship for self-hosting.
> ([Hugging Face Transformers — Qwen3](https://huggingface.co/docs/transformers/en/model_doc/qwen3))
> This page is the 32B companion to [gpu-vps-budget-70b.md](./gpu-vps-budget-70b.md).

---

## 1. Why this is much cheaper than 70B

A 32B model has ~45% as many parameters as a 70B one, so it needs roughly **half the
GPU memory**. That matters because it drops the job onto **smaller, far cheaper
cards** — often a single 24 GB consumer-class GPU instead of a datacenter 48–80 GB card.

Same two dials as before (see the 70B doc): **(1)** how much you compress the model,
and **(2)** whether you run it 24/7 or only on demand.

---

## 2. GPU memory Qwen3-32B needs

"Quantization" = how many bits per weight. Fewer bits → smaller → cheaper card, with
minor quality loss (4-bit keeps ~97–99% of quality on models this size).

| Precision | Weights size (32B) | Realistic VRAM (incl. KV cache + overhead) | GPU you need |
|-----------|--------------------|--------------------------------------------|--------------|
| **FP16** (full) | ~64 GB (32B × 2 bytes) | ~75 GB | 1× 80 GB (A100/H100) or 2× 48 GB |
| **8-bit** | ~32 GB | ~38–42 GB | **1× 48 GB** (A6000 / L40S) |
| **4-bit** (Q4_K_M) | ~19–20 GB | ~22–24 GB (short/medium context) | **1× 24 GB** (RTX 4090 / L4) |

Sources: [VRLA Tech LLM hardware sizing](https://vrlatech.com/llm-hardware-requirements-guide/),
[Techsy VRAM guide](https://techsy.io/en/blog/llm-vram-requirements-guide).

> [!WARNING] Context length is the catch at 24 GB
> At 4-bit on a **24 GB** card, the weights (~20 GB) leave little room for the **KV
> cache** (scratch memory that grows with prompt/conversation length). It's fine for
> short-to-medium prompts, but long context (32k+ tokens) or many simultaneous users
> can overflow 24 GB. If you need that headroom, step up to a **48 GB** card.

---

## 3. Live GPU prices (per GPU-hour, late 2026)

| GPU | VRAM | Cheap (Vast.ai marketplace) | Community (RunPod) | Secure/stable |
|-----|------|-----------------------------|--------------------|---------------|
| RTX 4090 | 24 GB | ~$0.14–0.37 | ~$0.34 | ~$0.69 |
| L4 / A5000 | 24 GB | ~$0.27+ | ~$0.40–1.10 | ~$1.10 |
| RTX A6000 | 48 GB | ~$0.28–0.33 | — | ~$0.53 |
| A100 | 80 GB | ~$1.19 | — | ~$1.49–1.79 |

Sources: [RunPod pricing](https://www.runpod.io/gpu-cloud/pricing),
[Hostinger 2026 comparison](https://www.hostinger.com/tutorials/?p=158956),
[Hack'celeration RunPod pricing](https://www.hackceleration.com/labs/runpod-pricing),
[ComputePrices — RTX 4090 on Vast](https://computeprices.com/providers/vast/gpus/rtx4090).
Prices move weekly — confirm on the live dashboard before renting.

A month of 24/7 = **730 hours**. "On-demand" below = ~8 hrs/day on weekdays ≈ **174 hours**.

---

## 4. The budget

### 4-bit on a single 24 GB GPU (recommended — cheapest viable)

| Tier | Rate/hr | **24/7 / month** | **On-demand (174 hr) / month** |
|------|---------|------------------|--------------------------------|
| Vast.ai (cheapest, interruptible) | $0.20 | ~$146 | **~$35** |
| RunPod Community | $0.34 | ~$248 | ~$59 |
| RunPod Secure (stable) | $0.69 | ~$504 | ~$120 |

### 8-bit on a single 48 GB GPU (more quality/context headroom)

| Tier | Rate/hr | **24/7 / month** | **On-demand / month** |
|------|---------|------------------|------------------------|
| A6000, Vast.ai | $0.33 | ~$241 | ~$57 |
| A6000, RunPod Secure | $0.53 | ~$387 | ~$92 |

### FP16 full precision on a single 80 GB GPU (max quality, rarely needed)

| Tier | Rate/hr | **24/7 / month** | **On-demand / month** |
|------|---------|------------------|------------------------|
| A100 80 GB | $1.19 | ~$869 | ~$207 |
| A100 80 GB (secure) | $1.79 | ~$1,307 | ~$311 |

**Storage:** add ~$4–8/mo to "park" the model between sessions. A 4-bit Qwen3-32B is
~20 GB; with OS + dependencies ~60–80 GB total, at ~$0.05–0.10/GB/month. You pay this
even while the GPU is off.

---

## 5. Recommendation

**Start here:** 4-bit Qwen3-32B on a **single 24 GB RTX 4090**, run **on-demand**.
- **~$35–60/month** if you spin it up only during working hours.
- **~$150–250/month** if you keep it running 24/7.

Step up to **8-bit on a 48 GB A6000** (~$57–92/mo on-demand) only if you hit quality
limits or need long-context / multi-user headroom.

Same three caveats as the 70B doc still apply:
1. **"VPS" ≠ GPU cloud** — use a GPU cloud provider (RunPod, Vast.ai, Lambda), not a
   plain DigitalOcean/Linode VPS.
2. **Consider a per-token API** if usage is light or bursty — often cheaper than
   renting a GPU 24/7, with no ops work.
3. The cheapest Vast.ai hosts are **interruptible** — fine for experiments, less so
   for an always-on service.

---

## 6. 32B vs 70B at a glance (on-demand, 4-bit)

| Model | Card | **On-demand / mo** | **24/7 / mo** |
|-------|------|--------------------|---------------|
| **Qwen3-32B** | 1× 24 GB RTX 4090 | **~$35–60** | ~$150–250 |
| Qwen2.5-72B | 1× 48 GB A6000 | ~$92 | ~$250–400 |

Qwen3-32B is the materially cheaper and simpler choice — one commodity GPU, no
multi-GPU setup — and is the newer model line.

---

## Sources
- [Hugging Face — Qwen3 model docs](https://huggingface.co/docs/transformers/en/model_doc/qwen3)
- [RunPod GPU pricing](https://www.runpod.io/gpu-cloud/pricing)
- [Hostinger — Cloud GPU pricing compared 2026](https://www.hostinger.com/tutorials/?p=158956)
- [Hack'celeration — RunPod pricing](https://www.hackceleration.com/labs/runpod-pricing)
- [ComputePrices — RTX 4090 on Vast.ai](https://computeprices.com/providers/vast/gpus/rtx4090)
- [VRLA Tech — LLM hardware requirements](https://vrlatech.com/llm-hardware-requirements-guide/)
- [Techsy — LLM VRAM requirements guide](https://techsy.io/en/blog/llm-vram-requirements-guide)
