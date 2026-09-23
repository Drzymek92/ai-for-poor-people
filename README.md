# AI for Poor People

**AI that doesn't need a datacentre — or a budget.**
Every project here runs end-to-end on **one consumer GPU — ≤16 GB VRAM**, with local models and no
paid API in the default path. Where a project *can* call a hosted model, that path is optional,
explicitly opted into, and engineered for economy — [see below](#on-outgoing-api-calls).

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
- [x] Any outgoing API path is **optional, explicitly opted into, announced at startup**, and
      cost-engineered when used
- [x] Ships a **measured Hardware Envelope** — peak VRAM, throughput, cold start — produced by a probe, not written by hand
- [x] Carries the `ai-for-poor-people` topic, a real description, and the VRAM badge
- [x] Has a LICENSE, a runnable quickstart, tests and clean lint
- [x] Has something to look at — a recording, a screenshot, or a hosted demo

## Members

| Project | What it does | Peak VRAM |
|---|---|---|
| [interview-copilot](https://github.com/Drzymek92/interview-copilot) | Disclosed, local-first live copilot for AI-engineer interviews — local Whisper STT, Ollama reasoning, loopback dashboard | **13.3 GB** <sub>STT + 14B, both resident</sub> |
| [agentic-ml-lab](https://github.com/Drzymek92/agentic-ml-lab) | Reproducible local workstation for agentic ML on Kaggle competitions | **5.1 GB** <sub>solution-tree search step</sub> |
| [synthetic-data-factory](https://github.com/Drzymek92/synthetic-data-factory) | YAML-defined synthetic datasets with an LLM-as-judge quality harness | **5.1 GB** <sub>generate + judge</sub> |
| [document-librarian](https://github.com/Drzymek92/document-librarian) | Offline document catalog for LLM agents — DuckDB, full-text search, local embeddings | **0.1 GB** <sub>recall path</sub> |
| recruiter-copilot | Interviewer-side copilot — bilingual transcription, evidence-cited post-call analysis, consent-gated | _in progress_ |

Every figure above is a **measured peak on the reference machine**, taken cold, with the card
cleared first — not an estimate, and not a spec-sheet number. The heaviest project in the
collection leaves **2.6 GB spare on a 16 GB card**; the lightest barely touches it.

### Where the heaviest one spends it

`interview-copilot` is the ceiling case, because it holds two models resident at once — speech
recognition never unloads while the reasoning model answers. Measured separately:

| Component | Peak VRAM |
|---|---|
| Reasoning model (`qwen3:14b`, 16k context) | 11.2 GB · 44.7 tok/s · 3.5 s cold start |
| Whisper `large-v3-turbo` (float16, beam 5) | 2.2 GB |
| **Both resident, end to end** | **13.3 GB** |

That is the whole design problem of the collection in one row: an 8B model would leave 9 GB
free, and the project chose the 14B anyway because the answers are better — spending the
headroom deliberately rather than discovering it was gone.

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

## On outgoing API calls

Several projects here speak the OpenAI-compatible API. That is deliberate architecture, not a
loophole: the same client talks to a local Ollama endpoint and to a hosted gateway, so the only
thing separating *runs on my desk* from *runs on a rented frontier model* is one setting. Portable
beats purist.

Three rules keep it honest.

**The shipped default is local.** `interview-copilot` ships `REASONING_BACKEND=local`;
`recruiter-copilot` ships `PROFILE=local`; the rest point at `localhost:11434`. Clone any of them
and you get a working system with no key, no account and no bill.

**Going out is explicit, and it announces itself.** The cloud path is opt-in, needs its own
environment variables, and says so at startup — an interview transcript never leaves the machine
silently. `recruiter-copilot` goes further and *refuses to start* on `PROFILE=api` without a key
rather than quietly degrading. A fallback you didn't ask for is an egress you didn't consent to.

**When a project does call out, the call is engineered for economy.** Prompt caching on the
reused context block, bounded and batched requests, and the cheapest model tier that clears the
quality bar. Cost is treated as a design constraint in exactly the way VRAM is — the two halves of
the same discipline.

So the claim is not *"this code cannot reach the internet."* It is: **local by default, remote by
choice, and cheap when you choose it.**

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
