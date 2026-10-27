<p align="center">
  <a href="https://huggingface.co/emonHasan01/Bangla-Gemma-Pro">
    <img src="https://img.shields.io/badge/%F0%9F%A4%97%20Model-Bangla--Gemma--Pro-FFD21E?style=flat&labelColor=24292e" alt="Model">
  </a>
  <a href="https://huggingface.co/emonHasan01/Bangla-Gemma-Pro-GGUF">
    <img src="https://img.shields.io/badge/%F0%9F%A4%97%20GGUF-llama.cpp-FFD21E?style=flat&labelColor=24292e" alt="GGUF">
  </a>
  <a href="https://huggingface.co/Qwen/Qwen3.5-0.8B">
    <img src="https://img.shields.io/badge/Base-Qwen3.5--0.8B-FFD21E?style=flat&labelColor=24292e" alt="Base model">
  </a>
  <img src="https://img.shields.io/badge/Method-QLoRA%204--bit-8A2BE2?style=flat&labelColor=24292e" alt="Method">
  <img src="https://img.shields.io/badge/Benchmarks-at%20chance-lightgrey?style=flat&labelColor=24292e" alt="Benchmarks">
  <img src="https://img.shields.io/badge/License-Apache%202.0-blue?style=flat&labelColor=24292e" alt="License">
</p>

<img src="assets/Bangla-Gemma-Pro.png" alt="emonHasan01/Bangla-Gemma-Pro" width="100%">

<div align="center">

[Hugging Face](https://huggingface.co/emonHasan01) | [GitHub](https://github.com/Md-Emon-Hasan) | [LinkedIn](https://www.linkedin.com/in/md-emon-hasan-695483237/) | [WhatsApp](https://wa.me/8801834363533) | [Portfolio](https://emonlabs-ai.hitechparks.com/)

**Author:** [Md. Emon Hasan](https://huggingface.co/emonHasan01)

</div>

# Bangla-Gemma-Pro

**A QLoRA fine-tune of [`Qwen/Qwen3.5-0.8B`](https://huggingface.co/Qwen/Qwen3.5-0.8B) on Bangla instruction data.**
Adapters on the text tower over a 4-bit base, trained on
[`Bangla-Instruct`](https://huggingface.co/datasets/md-nishat-008/Bangla-Instruct).

> ### Read this before using the model
>
> **This fine-tune did not produce usable Bangla generation.** It writes fluent-looking
> Bangla that is frequently ungrammatical and factually invented, and the benchmark scores
> sit at random-chance level for both this model and its base. It is published as a
> documented negative result — a record of what was tried, what was measured, and why it
> did not work — not as a model to deploy.
>
> If you need Bangla generation today, use a larger model. See
> [What to do instead](#what-to-do-instead).

## What actually happened

Four decoding configurations were tried on a single factual question. None produced a
correct or well-formed answer.

**Prompt:** রবীন্দ্রনাথ ঠাকুর কে ছিলেন? *(Who was Rabindranath Tagore?)*
**Correct answer:** Bengali poet, 1861–1941, first non-European Nobel laureate in Literature.

| Decoding | Output (abridged) | Problem |
|---|---|---|
| Greedy | রবীন্দ্রনাথ ঠাকুরের মৃত্যু হয় ১৯৭৫ সালের ১০ মার্চে। তার মৃত্যু পরিচয়ের মধ্যে তিনি ১৯৭৫ সালের ১০ মার্চে মৃত্যু হয়। *(repeats)* | Wrong year; same sentence looped |
| `repetition_penalty=1.15` | রবীনদ্বারাচ্ছিত রবীনদ্বারাচ্ছিত শহরের পূর্বাবশ্যই ১৯৭৫ সালে তৈরি... ২০০৩ সালে মৃত্যু | Loop replaced by non-words; two invented dates |
| `no_repeat_ngram_size=4` | রবীনদ্রনাধি শ্রেণির একজন বিভিন্ন প্রধানী। তিনি ১৯৩০ সালে মৃত্যু হয় | Name misspelled; third invented year |
| Sampling, T=0.7 | ১৮৯৬ সালে রবীন্দ্রনাথ শিশুদের জন্য ফেলেন... ১৯০৭ সালে শাস্ত্রত্বরো লাত্রাদের আঙুলো | Incoherent |

Four settings, four different fabricated death years, none of them 1941. The repetition
loop under greedy decoding is a symptom; the underlying problem is that the model has no
reliable Bangla factual or grammatical competence to decode from.

## Benchmark results

Five Bangla multiple-choice benchmarks were planned, scored by length-normalised
log-likelihood. **Three failed to load** and two ran.

| Benchmark | Options | Random chance | `Qwen3.5-0.8B` | This model | Δ |
|---|---|---|---|---|---|
| bangla-mmlu | 4 | **0.250** | 0.283 | 0.240 | −0.043 |
| piqa-bn | 2 | **0.500** | 0.500 | 0.530 | +0.030 |
| **average** | | | 0.392 | 0.385 | **−0.007** |

**The random-chance column is the whole table.** On bangla-mmlu the base model scores 0.033
above guessing and this model scores *below* it. On piqa-bn the base model scores exactly
0.500 — a coin flip. Both models are at the floor, so the −0.007 average difference carries
no information at all.

This is a limitation of the evaluation as much as the model: MMLU-style benchmarks do not
discriminate between models at the 0.8B scale. They were chosen because TituLLM uses them,
but TituLLM evaluates models an order of magnitude larger.

Benchmarks that did not load: `hishab/commonsenseqa-bn`, `hishab/openbookqa-bn`,
`hishab/boolq_bn` (its columns are `passage, question, answer, answer_bn`, which the
harness did not map to an option list).

**Perplexity was not measured.** `csebuetnlp/xlsum` is a script-based dataset and recent
`datasets` releases no longer execute loading scripts. With generation broken and the
benchmarks at chance, this would have been the only metric capable of showing a real
difference, and its absence is the biggest gap in this evaluation.

## Four things that went wrong, in order of impact

**1. The base model's Bangla is weak.** This is the dominant cause. The base outputs are
themselves garbled — it answered the same Tagore question with "রবীন্দ্রনাম ঠাকুরের ছিলেন ছিল
১৯৯০ সালে". Qwen3.5 advertises 201 languages, but coverage at 0.8B is thin. Fine-tuning
adapts what a model already has; it cannot install a language.

**2. The chat template injected empty thinking blocks.** Qwen3.5 renders an empty
`<think></think>` pair into the assistant turn. The notebook tried
`enable_thinking=False`, which this template ignores, so every one of the 10,700 training
examples taught the model to emit an empty reasoning block before answering. Visible in the
rendered template:

```
<|im_start|>assistant
<think>

</think>

{answer}<|im_end|>
```

**3. 97% of the training data was discarded.** `Bangla-Instruct` holds 342,391 pairs with a
median length of 729 tokens. Filtering to a 512-token window left **10,700**, and 400 steps
at an effective batch of 16 saw about 6,400 of them. Bangla costs more tokens per word than
English, so a window sized for English truncates Bangla aggressively.

**4. Training ran in fp32.** Unsloth reported `Using float16 precision for qwen3_5 won't
work! Using float32`, so the fp16 path was unavailable on this architecture. Training still
completed, but the 4-bit base bought less than intended.

## Training

| | |
|---|---|
| Base model | `Qwen/Qwen3.5-0.8B` (865,765,440 parameters) |
| Method | QLoRA — 4-bit NF4 base, adapters on the text tower, via Unsloth |
| Trainable | 12,779,520 parameters (1.48%) |
| Rank / alpha / dropout | 32 / 64 / 0 |
| Target modules | `q, k, v, o, gate, up, down` |
| Data | `Bangla-Instruct`, 10,700 of 342,391 pairs within the length limit |
| Max sequence | 512 tokens |
| Learning rate | 2e-4, cosine schedule, 20 warmup steps |
| Batch | 4 × 4 accumulation = 16 effective |
| Steps | 400 (≈6,400 examples seen) |
| Optimizer | `adamw_8bit`, `max_grad_norm=0.3` |
| Precision | fp32 (fp16 unsupported for this architecture) |
| Final training loss | 0.952 |
| Peak VRAM | 2.20 GB of 15.0 |

Loss fell to 0.952, which is the trap worth naming: **the loss curve looked healthy while
the model learned nothing useful.** Loss measures next-token prediction on the training
distribution, not whether the output is true or grammatical.

## Files

| | |
|---|---|
| Adapter | 51.1 MB |
| Merged 16-bit | 1.77 GB |
| `Bangla-Gemma-Pro.Q4_K_M.gguf` | 0.54 GB |
| `Bangla-Gemma-Pro.Q8_0.gguf` | 0.83 GB |

GGUF builds are in
[`emonHasan01/Bangla-Gemma-Pro-GGUF`](https://huggingface.co/emonHasan01/Bangla-Gemma-Pro-GGUF).
The vision projector did not convert (`KeyError: 'image_mean'`); the GGUF files are text-only,
which costs nothing here since only text was trained and evaluated.

## Usage

Provided for reproduction, not for production.

```python
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM

tok = AutoTokenizer.from_pretrained("emonHasan01/Bangla-Gemma-Pro")
model = AutoModelForCausalLM.from_pretrained("emonHasan01/Bangla-Gemma-Pro",
                                             dtype=torch.float32, device_map="auto")

messages = [
    {"role": "system", "content": "আপনি একজন সহায়ক সহকারী। সবসময় স্পষ্ট ও সঠিক বাংলায় উত্তর দিন।"},
    {"role": "user", "content": "একটি ছোট গল্প লিখুন যেখানে একজন কৃষক বৃষ্টির জন্য অপেক্ষা করছেন।"},
]
prompt = tok.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs = tok(prompt, return_tensors="pt", add_special_tokens=False).to(model.device)
out = model.generate(**inputs, max_new_tokens=200, do_sample=True,
                     temperature=0.7, top_p=0.9, repetition_penalty=1.1)
print(tok.decode(out[0][inputs["input_ids"].shape[1]:], skip_special_tokens=True))
```

The Bangla system prompt was present in every training example. Open-ended creative
prompts ("write a short story") degrade more gracefully than factual ones, because there is
no fact to get wrong.

## Limitations

- **It invents facts.** Four decoding settings produced four different wrong death years for
  a well-known figure. Treat every date, number and name as fabricated.
- **Bangla morphology is often broken.** Outputs contain non-words
  (রবীনদ্বারাচ্ছিত, মৃত্তি, ব্যাক্সি) that no Bangla speaker would write.
- **Greedy decoding loops.** `repetition_penalty` stops the loop but replaces it with
  different nonsense rather than with correct text.
- **Benchmarks are uninformative at this scale**, as the chance column above shows.
- **Perplexity, the one metric that might have shown a real signal, is missing.**
- **English ability after fine-tuning was not measured.**
- **No safety, bias or cultural-sensitivity evaluation**, which matters for Bangla content
  touching religion, politics or region.
- **Empty `<think></think>` blocks** were trained in and may appear in output.

Do not use this model where a wrong answer matters, which for a factual Bangla assistant is
everywhere.

## What to do instead

If you want Bangla generation that works:

- **[TigerLLM](https://huggingface.co/datasets/md-nishat-008/Bangla-Instruct)** (1B and 9B,
  LLaMA-3.2 and Gemma-2) is the strongest documented Bangla LLM family, and it reached its
  results through **continual pretraining on a 10M-token Bangla corpus before** instruction
  tuning — not instruction tuning alone.
- For Bangla understanding rather than generation,
  **[BanglaBERT](https://huggingface.co/csebuetnlp/banglabert)** from BUET CSE NLP remains
  strong for classification and extraction.

If you want to retry this experiment, the fixes in priority order:

1. **Start from a model with real Bangla ability**, or do continual pretraining on Bangla
   text first. Instruction tuning cannot create language competence.
2. **Strip the empty thinking block** from the chat template before building the dataset.
3. **Raise the sequence limit to 1024–2048** so the data is not 97% discarded.
4. **Use a benchmark that discriminates at this scale**, or rely on perplexity and human
   reading instead of MMLU-style multiple choice.
5. **Measure perplexity** on a parquet-based Bangla corpus, since script-based datasets no
   longer load.

## Reproducing

Produced the baseline, the training, the charts and this
card. Raw numbers are in [`results.json`](results.json).

## Citation

```bibtex
@inproceedings{raihan-zampieri-2025-tigerllm,
  title     = {{TigerLLM} - A Family of {B}angla Large Language Models},
  author    = {Raihan, Nishat and Zampieri, Marcos},
  booktitle = {Proceedings of ACL},
  year      = {2025}
}
@software{unsloth,
  title  = {Unsloth},
  author = {Daniel Han and Michael Han and Unsloth team},
  year   = {2023},
  url    = {https://github.com/unslothai/unsloth}
}
```