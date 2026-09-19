# Technical Appendix: System Architecture & Benchmark Specifications

This document expands on the high-level system description in §3 ("System
Design & Technical Architecture") of the paper, providing the implementation
detail needed for replication.

## A. System Components

- **Client presentation layer:** a native chat-interface client. It performs
  no model inference itself; it dispatches requests to a local backend over
  a loopback HTTP interface on the same machine.
- **Backend server & model distribution:** a local Python process handles
  orchestration. Model weights (a LoRA adapter checkpoint built on
  `Qwen2.5-1.5B-Instruct`) are fetched once from a model hub and cached
  locally on first run rather than bundled with the client.
- **Dynamic in-memory RAG indexing:** when course PDFs are uploaded, text is
  extracted, chunked, and embedded in memory at runtime using
  `sentence-transformers`. Embeddings are computed fresh on each server
  launch rather than persisted to a vector database file.
- **Dual-stage contextual routing:** incoming prompts are first scored by a
  lightweight Maximum Entropy classifier against document-derived sentence
  samples. Prompts scoring above threshold are routed through RAG context
  injection (approximately 3 sentences per chunk, ~9 sentences of injected
  context); prompts scoring below threshold are routed through a trained
  off-topic refusal-and-redirect path instead.

## B. Fine-Tuning Corpus

The fine-tuning dataset for the primary evaluated model comprises 190
manually curated, multi-turn instructional conversation records in
standard ChatML structure.

| Dataset Split | Record Count | Total Size | Mean Tokens / Record | Token Range |
|---|---|---|---|---|
| Training set | 155 | 171.4 KB | 248.9 | 143–648 |
| Validation set | 35 | 40.0 KB | 222.2 | 145–420 |
| **Total corpus** | **190** | **211.4 KB** | **244.0** | **143–648** |

Every record enforces a standardized system-prompt persona restricting the
model to a single course domain, requiring scaffolded (not complete)
solutions, and requiring refusal-and-redirect on off-topic queries.

Manual classification of all 190 records yielded three functional
categories: general on-topic conceptual explanations (57%, 108 records),
code scaffolding and guided inquiry with placeholder tokens (29%, 55
records), and off-topic refusal/redirection training examples (14%, 27
records).

## C. Fine-Tuning Configuration & Hardware Benchmarks

Parameter updates used Low-Rank Adaptation (LoRA; rank $r=8$, $\alpha=16$,
targeting query/value projection layers), updating 5.28M trainable
parameters out of 1,543.7M total (0.342% of the base model) in float16
precision without quantization, across two sequential training phases
totaling 1,500 iterations. Training and inference were benchmarked on an
Apple M3 processor with 8 GB unified memory running Apple MLX — ordinary
consumer laptop hardware, not dedicated training infrastructure.

**Training metrics:**

| Metric | Phase 1 (From Scratch) | Phase 2 (Resumed) |
|---|---|---|
| Iterations | 1,000 | 500 |
| Learning rate | 1×10⁻⁵ | 7×10⁻⁶ (decayed) |
| Wall-clock time | ~44 min (+14 min initial download) | ~13 min |
| Throughput | 0.65–1.35 it/s (avg ~0.85) | 0.65–1.36 it/s |
| Training token velocity | 17–275 tok/s (avg ~230) | ~250–275 tok/s |
| Peak memory footprint | 4.46 GB | 4.46 GB |
| Final train / validation loss | 0.047 / 0.917 | 0.052 / 0.904 |

Total adapter-generation wall-clock time was approximately 57 minutes of
active computation. On the target 8 GB device, system memory swap climbed
to ~3.6/4 GB during Phase 1, causing periodic 5×–15× slowdowns (e.g.,
iterations 80–90 dropped to 0.07–0.2 it/s before recovering); on-device
fine-tuning was fully functional but swap contention introduced 2×–3×
run-to-run wall-clock variance depending on OS background load.

**Inference metrics** (benchmarked via the underlying MLX engine CLI
directly, isolating engine performance from client/UI overhead):

| Task Category | Prompt Tokens | Prefill Speed | Generated Tokens | Decode Speed | Peak Memory |
|---|---|---|---|---|---|
| On-topic explanation (`.groupby`) | 140 | 137.5 tok/s | 85 | 25.7 tok/s | 3.41 GB |
| Off-topic redirect | 139 | 220.0 tok/s | 13 | 27.8 tok/s | 3.41 GB |
| Code-scaffold guidance (`.iloc`) | 143 | 219.2 tok/s | 256 (generation cap) | 24.6 tok/s | 3.42 GB |

Across categories, decode speed held steady at ~26 tok/s and peak memory at
~3.4 GB — about 1 GB lower than during training. At ~26 tok/s, a typical
80–100 token scaffolded response generates in roughly 3–4 seconds.

A second, more heavily constrained model variant was developed for a single
specific programming course during the project; it appears only in the
automated benchmark comparison (see `05-benchmark-validation.md`) and was
not used in the expert evaluation study.
