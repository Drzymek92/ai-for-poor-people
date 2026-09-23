# AI for Poor People

**AI that doesn't need a datacentre — or a budget.**
Every project here runs end-to-end on **one consumer GPU — ≤16 GB VRAM**, with local models and no
paid API in the default path.

> *Poor* refers to the hardware budget, not to anyone using it. This is a collection about what you
> can build on the machine already under your desk.

---

## Why the constraint

Most published AI work assumes a rented datacentre. That assumption quietly excludes students,
hobbyists, small teams, anyone in a country where an API bill in dollars is a real cost — and
anyone whose data simply cannot leave the building.

The constraint is also good engineering discipline. A 16 GB ceiling forces honest choices about
model size, quantization, batching and what actually has to be a language model at all. Most of
these projects got *better* when they stopped being allowed to call a frontier API.

## The bar for membership

A repository joins this collection when it meets all of:

- [x] Runs **end-to-end on one consumer GPU, ≤16 GB VRAM**, with no paid API in the default path
- [x] Ships a **measured Hardware Envelope** — peak VRAM, throughput, cold start — produced by a probe, not written by hand
- [x] Carries the `ai-for-poor-people` topic, a real description, and the VRAM badge
- [x] Has a LICENSE, a runnable quickstart, tests and clean lint
- [x] Has something to look at — a recording, a screenshot, or a hosted demo

## Members

| Project | What it does | Peak VRAM |
|---|---|---|
| [interview-copilot](https://github.com/Drzymek92/interview-copilot) | Disclosed, local-first live copilot for AI-engineer interviews — local Whisper STT, Ollama reasoning, loopback dashboard | _pending_ |
| [document-librarian](https://github.com/Drzymek92/document-librarian) | Offline document catalog for LLM agents — DuckDB, full-text search, local embeddings | **0.1 GB** <sub>(recall path)</sub> |
| [synthetic-data-factory](https://github.com/Drzymek92/synthetic-data-factory) | YAML-defined synthetic datasets with an LLM-as-judge quality harness | _pending_ |
| [agentic-ml-lab](https://github.com/Drzymek92/agentic-ml-lab) | Reproducible local workstation for agentic ML on Kaggle competitions | _pending_ |
| recruiter-copilot | Interviewer-side copilot — bilingual transcription, evidence-cited post-call analysis, consent-gated | _in progress_ |

Rows still marked _pending_ have not been measured yet. They stay blank rather than estimated —
a collection whose entire claim is a VRAM ceiling does not get to guess at its own numbers.

### Reference measurement

The baseline every project is judged against — an 8B model answering one prompt, measured cold:

```
GPU           NVIDIA GeForce RTX 5060 Ti — 16 GB · driver 595.84
Peak VRAM     5.1 GB of 15.9 GB   ·   headroom 10.8 GB
Attributable  5.1 GB   (baseline 0.0 GB excluded)
Models        llama3.1:8b
Throughput    82.7 tok/s   ·   cold start 1.92 s
```

An 8B model leaves **two thirds of a 16 GB card free**. That headroom is the space the projects
in this collection actually live in.

## How the numbers are made

Every envelope comes from one probe, run on the reference machine, never written by hand. It
watches the *device* rather than the process, so it works the same for PyTorch, Ollama and
faster-whisper.

`nvidia-smi` reports device-wide memory, which is the trap: a desktop compositor and a
neighbour's resident model are in that number too. So each result carries both figures —

- **Peak VRAM** — device total at its peak. The honest answer to *does it fit in 16 GB?*
- **Attributable** — peak minus the baseline taken before the workload started. The honest
  answer to *what does this project cost?*

— and refuses to produce a publishable block in three cases:

| Guard | Why it exists |
|---|---|
| **Failed run** | An envelope for a workload that exited non-zero measures nothing |
| **Contaminated** | A materially large baseline means the peak describes the card, not the project |
| **Under-sampled** | A workload shorter than a few sampling intervals can peak between reads |

The middle guard was not theoretical. The first real run of `document-librarian` reported
**5.6 GB peak / 0.0 GB attributable** — the previous probe's 8B model was still resident and the
librarian was being billed for it. Cold, the true figure is **0.1 GB**.

## Reference machine

Every envelope in this collection is measured on the same box, and the box is an ordinary one:

```
GPU     NVIDIA GeForce RTX 5060 Ti — 16 GB, Blackwell (sm_120)
CPU     AMD Ryzen 7 9700X, 8C/16T
RAM     32 GB
OS      Ubuntu 24.04 LTS
Stack   PyTorch (CUDA 12.8) · Ollama · faster-whisper
```

If a project needs more than this, it does not belong here.

---

Built by [Michał Drzymała](https://github.com/Drzymek92) · MIT licensed unless a repo says otherwise
