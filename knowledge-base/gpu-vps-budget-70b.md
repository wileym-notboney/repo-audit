# Monthly Budget: Self-Hosting a 70B-Class LLM on a GPU VPS

**Date:** 2026-10-11
**Question:** "Calculate the monthly budget for a VPS with enough GPU to run Qwen 3 70B."

> [!IMPORTANT] Fact-check on the model
> **There is no "Qwen3 70B."** The Qwen3 family's *dense* models stop at **32B**
> (sizes: 0.6B, 1.7B, 4B, 8B, 14B, 32B). Its "big" models are **Mixture-of-Experts**:
> Qwen3-30B-A3B and Qwen3-235B-A22B. The only ~70B-class Qwen model is the older
> **Qwen2.5-72B**. ([Hugging Face Transformers — Qwen3](https://huggingface.co/docs/transformers/en/model_doc/qwen3), [DataCamp Qwen3 overview](https://www.datacamp.com/blog/qwen3))
>
> This doc budgets for a **70B/72B-class dense model** (what was asked). If you
> actually want **Qwen3**, the realistic target is **Qwen3-32B**, which is roughly
> half the memory and cost — see the "Cheaper alternative" section.

---

## 1. The two things that decide your bill

Think of renting a GPU like renting a car. Two questions set the price far more
than which rental company you pick:

1. **How big an engine do you need?** (= how much GPU memory / VRAM) — decided by
   the model size *and* how aggressively you compress it ("quantization").
2. **Do you keep it running 24/7, or only when you drive?** (= always-on vs
   on-demand hours) — this alone can swing the monthly bill by 4–5×.

Everything below is just these two dials turned to different settings.

---

## 2. Dial 1 — How much GPU memory a 70B model needs

A model's weights are a pile of numbers. "Quantization" is how many bits you spend
storing each number — fewer bits = smaller file = less VRAM, at a small quality
cost. Analogy: the same photo saved as a huge RAW file vs. a compressed JPEG. The
JPEG is far smaller and, for most uses, looks the same.

| Precision | Rough weights size (70B) | Realistic VRAM needed (incl. KV cache + overhead) | GPU you need |
|-----------|--------------------------|---------------------------------------------------|--------------|
| **FP16** (full) | ~140 GB (70B × 2 bytes) | ~160 GB+ | 2× 80 GB cards (2× A100/H100) |
| **8-bit** | ~70 GB | ~90 GB | 1× 80 GB (tight) or 2× 48 GB |
| **4-bit** (Q4_K_M) | ~38–40 GB | ~45–50 GB for short context | **1× 48 GB card** (RTX A6000 / L40S) |

Sources: [VRLA Tech LLM hardware sizing](https://vrlatech.com/llm-hardware-requirements-guide/),
[Techsy VRAM guide](https://techsy.io/en/blog/llm-vram-requirements-guide). Note: the
**KV cache** (scratch memory that grows with how long your prompts/conversations are)
can add tens of GB at long context — budget headroom if you need 32k+ token windows.

**Quality note:** For a 70B model, 4-bit (Q4_K_M) is widely reported at ~97–99% of
full-precision quality — most users cannot tell the difference on everyday tasks.
([VRLA Tech](https://vrlatech.com/llm-hardware-requirements-guide/)) So **4-bit on a
single 48 GB GPU is the sensible default** unless you have a specific reason not to.

---

## 3. Dial 2 — Live GPU rental prices (per GPU-hour, late 2026)

Prices vary by provider and by tier. "Community/marketplace" (e.g. Vast.ai, RunPod
Community) is cheapest but can be interrupted; "Secure" tiers cost more but are stable.

| GPU | VRAM | Cheap (community) | Typical (secure/on-demand) |
|-----|------|-------------------|----------------------------|
| RTX A6000 | 48 GB | ~$0.28–0.33 | ~$0.53 |
| L40S | 48 GB | ~$0.47–0.61 | ~$0.86–0.99 |
| A100 | 80 GB | ~$1.19 | ~$1.49–1.79 |
| H100 | 80 GB | ~$1.99 | ~$2.99–3.99 |

Sources: [RunPod pricing](https://www.runpod.io/gpu-cloud/pricing),
[Hostinger 2026 cloud GPU comparison](https://www.hostinger.com/tutorials/?p=158956),
[ThunderCompute vs Lambda](https://thundercompute.com/blog/lambda-labs-vs-thunder-compute),
[Introl 2025 provider comparison](https://introl.com/blog/lambda-paperspace-vast-gpu-cloud-comparison-2025).
**These move constantly — confirm on the live pricing page before committing.**

A month of 24/7 running = **730 hours** (24 × 365 ÷ 12).

---

## 4. The budget

### Scenario A — Always-on, 24/7 (730 hrs/month)

| Setup | GPU(s) | Rate/hr | GPU cost/mo | + Storage | **≈ Total/mo** |
|-------|--------|---------|-------------|-----------|----------------|
| **4-bit, budget** | 1× A6000 48 GB (Vast.ai) | $0.33 | $241 | ~$10 | **~$250** |
| **4-bit, stable** | 1× A6000 48 GB (RunPod secure) | $0.53 | $387 | ~$10 | **~$400** |
| **8-bit** | 1× A100 80 GB (RunPod) | $1.19 | $869 | ~$15 | **~$885** |
| **FP16 full** | 2× A100 80 GB | $2.38 | $1,737 | ~$20 | **~$1,760** |
| **FP16, fastest** | 2× H100 80 GB | $3.98 | $2,905 | ~$25 | **~$2,930** |

### Scenario B — On-demand, ~8 hrs/day weekdays (~174 hrs/month)

If you only need the model during working hours and shut it down otherwise:

| Setup | Rate/hr | **≈ GPU cost/mo** |
|-------|---------|-------------------|
| 4-bit, 1× A6000 (RunPod) | $0.53 | **~$92** |
| 8-bit, 1× A100 | $1.19 | **~$207** |
| FP16, 2× A100 | $2.38 | **~$414** |

> The same hardware costs ~4× less if you don't leave it idling overnight. Most
> single-user and dev workloads fit Scenario B.

**Storage/egress caveat:** GPU rental is the big line item, but add:
- **Persistent/network storage:** ~$0.05–0.10 per GB/month (a 4-bit 70B model ≈ 40 GB
  + OS/deps ≈ 100 GB total → ~$5–10/mo). You pay this even while the GPU is off, so
  it's what keeps your model "parked" between sessions.
- **Egress (data out):** often free or small, but check — some providers charge per GB.

---

## 5. Recommendation & push-back

**Simplest, cheapest viable path for a 70B-class model:**
**1× 48 GB GPU (RTX A6000) running the model 4-bit quantized**, on-demand, via
RunPod or Vast.ai. Budget **~$90–250/month** depending on hours and tier.

Three honest challenges to the plan, in the spirit of "solve the root problem":

1. **Do you actually need 70B?** If you wanted *Qwen3* specifically, **Qwen3-32B**
   is the real flagship dense model and benchmarks strongly. It fits 4-bit in ~20 GB
   (a 24 GB card like an RTX 4090/L4, ~$0.20–0.40/hr), roughly **halving** every
   number above. See below.

2. **Do you need to self-host at all?** If usage is bursty or light, a **per-token
   API** (many hosts serve Qwen models) often costs *far* less than renting a GPU
   24/7, with zero ops work. Self-hosting wins when you have steady high volume,
   strict data-privacy needs, or want full control. For a beginner doing occasional
   work, an API is usually the simpler and cheaper root solution.

3. **"VPS" vs "GPU cloud."** A traditional VPS (DigitalOcean/Linode-style) generally
   does **not** include a datacenter GPU big enough for a 70B model. You want a
   **GPU cloud** provider (RunPod, Vast.ai, Lambda) — that's what the prices above
   reflect.

### Cheaper alternative — Qwen3-32B (if "Qwen3" is what you meant)

| Setup | GPU | Rate/hr | 24/7 /mo | On-demand (174 hr) /mo |
|-------|-----|---------|----------|------------------------|
| 4-bit Qwen3-32B | 1× 24 GB (RTX 4090/L4) | ~$0.35 | ~$256 | ~$61 |
| 8-bit Qwen3-32B | 1× 48 GB (A6000) | ~$0.53 | ~$387 | ~$92 |

---

## 6. Open questions that would sharpen this estimate

To turn these ranges into one number, I'd need:
1. **Exact model** — Qwen2.5-72B, or Qwen3-32B, or something else?
2. **Quantization** you're comfortable with (4-bit is my default recommendation).
3. **Usage pattern** — truly 24/7, or on-demand during working hours?
4. **Context length / concurrency** — long documents or many simultaneous users push
   VRAM (and therefore cost) up.
5. **Provider preference** — cheapest-possible (Vast.ai) vs. stable/enterprise (Lambda).

---

## Sources
- [Hugging Face — Qwen3 model docs](https://huggingface.co/docs/transformers/en/model_doc/qwen3)
- [DataCamp — Qwen3 overview](https://www.datacamp.com/blog/qwen3)
- [RunPod GPU pricing](https://www.runpod.io/gpu-cloud/pricing)
- [Hostinger — Cloud GPU pricing compared 2026](https://www.hostinger.com/tutorials/?p=158956)
- [ThunderCompute — Lambda vs Thunder](https://thundercompute.com/blog/lambda-labs-vs-thunder-compute)
- [Introl — Lambda/Paperspace/Vast comparison 2025](https://introl.com/blog/lambda-paperspace-vast-gpu-cloud-comparison-2025)
- [VRLA Tech — LLM hardware requirements](https://vrlatech.com/llm-hardware-requirements-guide/)
- [Techsy — LLM VRAM requirements guide](https://techsy.io/en/blog/llm-vram-requirements-guide)
