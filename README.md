# Source fidelity of local LLMs: incident notes lab

Lab material for a master's assignment (Module 1). I tested whether small LLMs running locally can answer questions about, and summarize, a set of security incident documents **using only those documents**: without inventing data, without turning a suspicion into a fact, and without mixing up what someone said with what was actually observed.

The factors I changed were model size, quantization and temperature. The full report (in Spanish) is delivered separately; this repository contains the knowledge base, the questions, the setup and every model answer with my evaluation.

> All data in this repository is fictional: the company (NorthLogic S.A.), the incident (INC-2026-0417), the vulnerability (CVE-SIM-2026-0001) and the people. The attacker IP `203.0.113.45` belongs to the documentation range (RFC 5737).

## Contents

| Path | What it is |
|---|---|
| `corpus/` | The six documents loaded into AnythingLLM (the knowledge base) |
| `questions_and_verification_EN.md` | Questions, expected answers, scoring rubric and the full answer log: every model answer with its verdict and my notes |
| `model_matrix.md` | Models, quantizations, temperatures and parameters (Spanish) |
| `system_prompts.md` | System prompt used in the workspace (Spanish instructions, English prompt) |

## The corpus

The documents contradict each other on purpose, as happens in a real incident: things said in the heat of the moment are later corrected.

| Doc | Content |
|---|---|
| 01 | Vendor advisory for the vulnerability (conditional language: "could") |
| 02 | SOC notes and timeline, times in UTC, entries tagged `[OBS]`, `[DEC]`, `[HYP]` |
| 03 | Partial forensic report: one server confirmed, one not confirmed, no attribution |
| 04 | Internal statement and press release ("isolated in less than four hours", "no evidence of a leak") with IR verification notes |
| 05 | Incident chat thread ("all three servers, for sure", "it's the usual group"), later retracted |
| 06 | 72-hour update report with a table that corrects or retracts earlier claims |

## Questions

| Id | Question | Expected answer |
|---|---|---|
| Q1 | How many servers were compromised? | One confirmed (`web-prod-02`), one suspected (`web-prod-03`) |
| Q2 | Was customer data leaked? | Not determined |
| Q3 | Was the server isolated in less than four hours, as the press release says? | No, 4 h 16 min (02:14 to 06:30 UTC) |
| Q4 | What operating system do the servers run? | Not stated in the documents |
| S1 | Executive summary in at most 150 words | Must reflect the above without using retracted claims |

Q4 is the pure hallucination trap: no document mentions the operating system.

## Setup

- Hardware: NVIDIA RTX 3060 12 GB.
- Software: Ollama + AnythingLLM.
- AnythingLLM workspace in chat mode, with the six documents pinned so they are always in context.
- `num_ctx` = 12288. Reasoning disabled with `/no_think` in the system prompt.
- Model defaults for the rest: `top_k` = 20, `top_p` = 0.95, `repeat_penalty` = 1. Seed not fixed.
- One new thread per question.

System prompt:

```
/no_think
You answer only from the provided documents. Distinguish what was observed from what was stated by someone. Do not present possibilities as facts. If the documents do not contain the answer, say "not stated in the documents". If documents contradict each other, say so and prefer the most recent corrected one, citing it.
```

## Models and runs

All models are from the Qwen3 family, so that only size or quantization changes between them.

| Id | Ollama tag | Quantization | Size on disk | Temperatures |
|---|---|---|---|---|
| M3 | `qwen3:1.7b-q4_K_M` | Q4_K_M | 1.4 GB | 0.0 / 0.7 / 1.2 |
| M4 | `qwen3:4b-fp16` | F16 | 8.1 GB | 0.0 / 0.7 |
| M5 | `qwen3:4b-q8_0` | Q8_0 | 4.4 GB | 0.0 / 0.7 |
| M6 | `qwen3:4b-q4_K_M` | Q4_K_M | 2.6 GB | 0.0 / 0.7 / 1.2 |
| M8 | `qwen3:8b-q4_K_M` | Q4_K_M | 5.2 GB | 0.0 / 0.7 / 1.2 |

- Size axis: M3, M6, M8 (all Q4_K_M).
- Quantization axis: M4, M5, M6 (the same 4B model).
- 13 runs × 5 prompts = **65 answers**, one pass per configuration.

`model_matrix.md` and the header of `questions_and_verification_EN.md` describe the original plan, which also included `qwen3:14b` (M9) for 16 runs and 80 answers. M9 was not run, so it does not appear in the results.

## How answers were scored

Each answer was checked by hand against the documents.

- `correct`: `Y` (matches the expected answer), `P` (key facts present but opens with or contradicts itself over a wrong claim, or is incomplete), `N` (wrong conclusion, or answers something the documents do not say). `P` counts as 0.5.
- `invented`: `Y` only if the answer contains data that does not appear in any document.

Status errors (a retracted claim used as current, a suspicion stated as fact, a statement attributed to the wrong person) count against `correct`, not as invention, because the text does appear somewhere in the corpus.

## Results summary

| Model | Accuracy Q1–Q4 | Correct summaries (S1) |
|---|---|---|
| 1.7B Q4_K_M (M3) | 75 % | 0 of 3 |
| 4B Q4_K_M (M6) | 96 % | 0 of 3 |
| 8B Q4_K_M (M8) | 96 % | 1 of 3 (+2 partial) |
| 4B F16 (M4) | 100 % | 1 of 2 |
| 4B Q8_0 (M5) | 94 % | 0 of 2 |

- Short factual questions work well (91 % overall). No model invented an operating system in Q4.
- Summaries mostly fail: only 2 of 13 are correct. The usual error is not invention but status: 9 of 13 summaries repeat the retracted "three servers" claim and 6 attribute the attack to a known group.
- Only 2 of 65 answers contain invented data, both summaries: M4 F16 at T = 0.0 and M5 Q8_0 at T = 0.7.
- With one pass per configuration, no clear effect of quantization or temperature could be shown.

## Limitations

One pass per configuration and no fixed seed, so small differences are not conclusive. A single evaluator (me). No control runs without the system prompt or without documents. M4 and M5 were not run at T = 1.2. The corpus is small and built to contain traps.
