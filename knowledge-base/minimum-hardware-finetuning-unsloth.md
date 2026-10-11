# Minimum Hardware to Learn Fine-Tuning with Unsloth

**Date:** 2026-10-11
**Question:** "What's the minimum hardware to meaningfully learn about training
models with Unsloth etc.?"

**Short answer:** **$0.** A free **Google Colab (Tesla T4, 16 GB)** or **Kaggle
(2× T4 ≈ 30 GB)** notebook is enough to run the complete QLoRA fine-tuning loop with
Unsloth. The skill you learn there is identical to what you'd do on a $5,000 rig —
start free, buy hardware only when free tiers' limits actually block you.

Companion to the GPU budget pages in this [knowledge base](./INDEX.md).

---

## 1. "Training" here means fine-tuning (QLoRA), not from scratch

- **Training from scratch** = building a car from raw metal. Needs a cluster; not a
  learning exercise.
- **Fine-tuning with QLoRA** = repainting and tuning an existing car. This is what
  Unsloth specializes in and what ~99% of people mean by "learn to train models."

QLoRA freezes the base model in 4-bit and trains tiny "adapter" layers on top, which
is why it fits on small GPUs.

---

## 2. Official Unsloth VRAM minimums (QLoRA, 4-bit)

Source: [Unsloth requirements docs](https://docs.unsloth.ai/get-started/beginner-start-here/unsloth-requirements).
These are floors; real runs want headroom (and OOM errors usually mean batch size is
too high — set it to 1–3).

| Model size | QLoRA (4-bit) floor | Comfortable (ctx 2048, batch 1) |
|------------|---------------------|---------------------------------|
| 3B | 3.5 GB | ~6 GB |
| 7B | 5 GB | ~8–10 GB |
| 8B | 6 GB | ~10–12 GB |
| 70B | 41 GB | ~48 GB |

Plain **LoRA (16-bit)** needs far more (~19 GB for 7B, ~22 GB for 8B) — use **QLoRA**
for learning. Third-party guides cite ~8–12 GB for a 7B QLoRA run, consistent with
"floor + headroom." ([Pinggy](https://pinggy.io/blog/finetune_and_selfhost_llms_locally_with_unsloth/),
[VRLA Tech](https://vrlatech.com/how-much-vram-do-you-need-for-llm-fine-tuning-in-2026/))

**GPU support:** any NVIDIA GPU from 2018 on, CUDA capability 7.0+ (T4, V100, RTX
20/30/40/50, A100, H100, L40).

---

## 3. Hardware tiers (cheapest → most capable)

| Tier | Hardware | Cost | What you can learn on it |
|------|----------|------|--------------------------|
| **0 — start here** | **Free Google Colab (T4 16 GB)** or **Kaggle (2× T4 ≈ 30 GB)** | **$0** | Full QLoRA loop on 3B–14B models |
| 1 — reliable cloud | Colab Pro, or rent RTX 4090 24 GB (RunPod/Vast) | ~$10/mo, or ~$0.20–0.40/hr | Longer runs, no disconnects, up to ~14B comfortably |
| 2 — own the floor | Used **RTX 3060 12 GB** | ~$250–300 | QLoRA on 7–8B locally, offline, unlimited time |
| 3 — own the sweet spot | Used **RTX 3090 24 GB** | ~$600–800 | 14B comfortably, dabble with 32B — classic fine-tuning card |

Unsloth ships prebuilt **free T4 Colab/Kaggle notebooks** for Qwen and Llama (add your
dataset → "Run All" → export to GGUF/vLLM/Hugging Face). ([Unsloth docs](https://docs.unsloth.ai/get-started/beginner-start-here/unsloth-requirements))

---

## 4. Limits of the free tiers (when to move up)

- **Colab free:** T4 16 GB, but sessions disconnect on idle/time limits (~12 h max),
  and GPU availability isn't guaranteed. Colab Pro (~$10/mo) is more reliable/longer.
- **Kaggle:** 2× T4 (≈30 GB combined) or P100, ~30 GPU-hrs/week, 12 h sessions —
  often the better free option.

Hit these walls (overnight runs, bigger models, fully-local/private data) → that's the
signal to rent a 24 GB GPU by the hour, or buy.

---

## 5. Recommendation

1. **Start free** on Kaggle/Colab with an Unsloth notebook. Do the full loop on a
   3B–8B model: load 4-bit → attach LoRA → fine-tune on a small dataset → evaluate →
   export → run. That *is* the skill.
2. **Rent by the hour** (RTX 4090, ~$0.20–0.40/hr) when you need uninterrupted or
   longer runs — cents to a few dollars per session.
3. **Buy only when free/rented tiers block you.** Practical floor: **RTX 3060 12 GB**.
   Comfortable: **RTX 3090 24 GB** — VRAM is the one spec that most expands what you
   can fine-tune.

Analogy: learn in the free driving-school car first; buy your own only once you know
you'll keep driving.

---

## Sources
- [Unsloth — requirements (official VRAM minimums & supported GPUs)](https://docs.unsloth.ai/get-started/beginner-start-here/unsloth-requirements)
- [Pinggy — fine-tune & self-host LLMs locally with Unsloth](https://pinggy.io/blog/finetune_and_selfhost_llms_locally_with_unsloth/)
- [VRLA Tech — how much VRAM for LLM fine-tuning in 2026](https://vrlatech.com/how-much-vram-do-you-need-for-llm-fine-tuning-in-2026/)
