# Evidence-grounded TRIZ-RAG for Precision Livestock Farming

This repository contains the compact offline prototype, frozen benchmark statements, worked examples, and archived experimental results associated with the manuscript on hybrid article-patent retrieval and TRIZ-oriented solution generation for precision livestock farming (PLF).

The repository is designed for **auditability first**: retrieved identifiers, exact evidence spans, deterministic TRIZ traces, prompt conditions, model/runtime metadata and result workbooks are kept separate so that retrieval, reasoning and generation can be inspected independently.

![RAG architecture](docs/figures/Figure_1_RAG_architecture.png)

## What is in the repository

- **26 frozen PLF problem statements** (`TC-01` to `TC-26`).
- **Mixed PLF source tables** of publications and patents.
- **Deterministic two-channel retrieval**: TF-IDF lexical retrieval + TF-IDF/TruncatedSVD-384 latent retrieval + Reciprocal Rank Fusion.
- **Auditable evidence packets** with source IDs, titles and stored evidence spans.
- **Deterministic TRIZ scaffold** for IFR, contradiction analysis, Su-Field, compact ARIZ and technical-system evolution traces.
- **S1/S2 controlled GPT experiment** with 780 archived solution texts.
- **S3/S4 controlled Qwen experiment** with 780 archived local-model concepts and CPU runtime metadata.
- **Six local-model compatibility workbooks** for Qwen, Phi, Llama, Gemma and Mistral configurations.
- **Worked TC-14 example** showing a retrieval failure mode and the limits of downstream TRIZ prompting.
- **Machine-readable result summaries** and scripts to regenerate them.

## Important scientific boundary

The archived exploratory retrieval run did **not** use the final frozen 2006-2025 review subset exactly. The broader source files contain 1,738 publication records and 3,793 patent records. Filtering `Year` to 2006-2025 yields exactly **1,594 publications + 3,239 patents = 4,833 records**. A confirmatory manuscript release should rebuild the index and repeat the benchmark on those 4,833 records.

The workbook `results/retrieval/Expert_labelled_retrieval_metrics.xlsx` contains a **development-only stress test with simulated Andrea/Alexey rater profiles**. It is included to reproduce metric calculations and must not be represented as human expert validation. Human-adjudicated relevance labels and blinded engineering-quality scores remain a separate validation stage.

## Repository layout

```text
.
├── configs/                    # prototype configuration
├── data/
│   ├── raw/                    # publication and patent source tables
│   ├── benchmarks/             # 26 frozen TC statements
│   └── processed/              # archived normalized prototype data
├── docs/
│   ├── figures/                # architecture/results figures
│   ├── REPOSITORY_MAP.md
│   ├── REPRODUCIBILITY.md
│   └── RESULTS.md
├── examples/
│   └── TC-14/                  # end-to-end worked example
├── outputs/
│   ├── cards/                  # deterministic solution cards
│   ├── traces/                 # TRIZ/retrieval execution traces
│   └── tables/                 # archived retrieval outputs
├── results/
│   ├── generation/             # S1/S2 and S3/S4 result packages
│   ├── retrieval/              # evidence packet + dev-only metric workbook
│   ├── local_models/           # six CPU-only local-model workbooks
│   └── summary/                # compact CSV summaries
├── scripts/
│   ├── analysis/               # regenerate summaries and figures
│   └── experiments/s3_s4/      # exact-top-10 Qwen runner
├── src/local_triz/             # compact prototype implementation
└── tests/                      # core tests
```

See [`docs/REPOSITORY_MAP.md`](docs/REPOSITORY_MAP.md) for a file-by-file guide.

## Retrieval configuration used in the archived prototype

The executed retriever is deliberately simple and inspectable:

1. title repeated ×3, followed by abstract and keyword/classification text;
2. lowercase Unicode word/hyphen tokenization;
3. word unigrams and bigrams;
4. sublinear TF + smoothed IDF + L2 normalization;
5. lexical TF-IDF cosine ranking;
6. TruncatedSVD projection to 384 dimensions (`random_state=42`) + cosine ranking;
7. top-50 from each channel;
8. Reciprocal Rank Fusion with `k=60`;
9. top-10 archived evidence records per TC;
10. deterministic 500-character evidence span.

The SVD channel is **not** a pretrained multilingual embedding model. A future confirmatory version may add 384-dimensional multilingual sentence embeddings + hnswlib as a separate ablation.

## Controlled generation systems

| System | Model | Evidence | Explicit TRIZ structure |
|---|---|---|---|
| S1 | GPT-5.6 Sol | No local packet | Prompted solution slots |
| S2 | GPT-5.6 Sol | Same archived top-10 | Prompted solution slots |
| S3 | Qwen2.5-3B-Instruct Q4_K_M | Same archived top-10 | No explicit TRIZ scaffold |
| S4 | Qwen2.5-3B-Instruct Q4_K_M | Same archived top-10 | IFR, Contradiction, Su-Field, ARIZ, Evolution |

S1/S2 contain 26 TC × 3 replicate labels × 5 outputs per system. S3/S4 use the same design. For S1/S2, the interface did not expose a controllable random seed; replicate labels 1-3 should not be interpreted as guaranteed API seeds.

## Results snapshot

The completed S3/S4 CPU experiment contains 78 valid runs per system and 390 concepts per system. The archived workbooks yield:

- S3 runtime: approximately **46.38 s/run**;
- S4 runtime: approximately **46.87 s/run**;
- unique exact mechanism strings per TC: **12.19 ± 2.76** (S3) vs **13.96 ± 1.48** (S4);
- unique cited sources per run: **4.47 ± 1.03** (S3) vs **4.04 ± 1.26** (S4).

These are execution/diversity/provenance metrics. They do **not** establish superior engineering quality. See [`docs/RESULTS.md`](docs/RESULTS.md) and the CSV files under `results/summary/`.

## Worked example: TC-14

The folder [`examples/TC-14`](examples/TC-14/) contains:

- the camera-lens contamination contradiction;
- the exact top-10 evidence packet;
- one matched S1-S4 replicate;
- deterministic card and trace.

TC-14 is useful because the retrieved packet contains many computer-vision/farm-monitoring sources but few direct lens-cleaning mechanisms. It demonstrates a core limitation: correct citation provenance is not equivalent to problem relevance, and TRIZ prompting cannot reliably recover a missing mechanism if retrieval does not surface it.

## Quick start

Core prototype:

```bash
python -m pip install -e .
local-triz build
local-triz solve --problem-id TC-14 --top-k 10
local-triz validate-citations outputs/cards/TC-14.json
```

Analysis utilities:

```bash
python -m pip install -r requirements-analysis.txt
python scripts/analysis/summarize_results.py
python scripts/analysis/make_figures.py
```

Local S3/S4 reproduction requires a llama.cpp-compatible executable and a locally available `Qwen2.5-3B-Instruct Q4_K_M` GGUF. See [`docs/S3_S4_preflight_and_protocol.md`](docs/S3_S4_preflight_and_protocol.md).

## Evidence policy

A stored identifier and exact quote establish provenance only. They do not by themselves establish semantic relevance or causal support. Engineering combinations are treated as inference unless the supplied evidence directly states the mechanism being claimed.

## Citation and release metadata

`CITATION.cff.template` is provided as a template. Complete the final author list, repository DOI, release date and article DOI before publication. Repository-wide licensing is intentionally left pending in `LICENSE_PENDING.md` until code and data reuse terms are selected by the authors.

## Frozen-corpus sensitivity rerun

Version 0.2.0 adds a retrieval-only sensitivity rerun on the exact review-defined 4,833-record corpus (1,594 publications + 3,239 patents; 2006–2025). The rerun is stored in `results/retrieval/frozen_4833/`. The archived S1–S4 generation experiments were **not** rerun on the new packet, so the two experiment states are kept separate.
