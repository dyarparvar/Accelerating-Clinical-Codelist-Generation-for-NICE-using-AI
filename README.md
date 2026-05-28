# Accelerating Clinical Codelist Generation for NICE using AI

*April 2026*

**Authors:** Darya Yarparvar, Olga Shapovalova, Kseniia Warren, Abdullah Sheikh, Eshwar Meduri, Joe Flanagan

---

## Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Solution](#solution)
- [Architecture](#architecture)
- [Key Features](#key-features)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Pipeline Stages](#pipeline-stages)
- [Evaluation](#evaluation)
- [Future Directions](#future-directions)

---

## Overview

This project introduces an **Agentic Retrieval-Augmented Generation (RAG) pipeline** as an AI co-pilot for clinical analysts at NICE (National Institute for Health and Care Excellence). It automates the generation of SNOMED CT clinical codelists, transforming a laborious manual process into an efficient, auditable, and reproducible workflow — while keeping humans firmly in the loop.

---

## Problem Statement

NICE requires highly accurate, consistent, and auditable clinical codelists — typically expressed as SNOMED CT codes — to identify patient populations for healthcare research and quality indicators. The current process is:

- **Manual and time-consuming** — analysts must individually review thousands of candidate codes
- **Inconsistent** — different analysts may produce different codelists for the same research question
- **Difficult to audit** — decisions are rarely documented with clinical justifications
- **Prone to obsolescence** — medical terminologies evolve, but codelists are rarely updated systematically

These inefficiencies hinder timely and robust healthcare research across the NHS.

---

## Solution

We built an **Agentic RAG pipeline** that acts as an AI co-pilot for clinical analysts. The system is:

- **Fully local** — runs on-premise with no data leaving the organisation
- **Open-weight** — uses open-source models (Phi-4 via Ollama, Nomic Embed Text)
- **Audit-ready** — every recommendation includes a clinical justification and confidence flag
- **Human-in-the-loop** — analysts review AI suggestions and approve/reject with full transparency

<img width="989" height="821" alt="Pipeline architecture overview" src="https://github.com/user-attachments/assets/d453350c-ebdb-4f1f-a541-8d8948179d09" />

*Figure 1. An overview of the Agentic RAG pipeline. The pipeline comprises four sequential stages: a query router; a ReAct agent loop; hybrid retrieval with graph expansion; and LLM-based code classification.*

---

## Architecture

The pipeline comprises four sequential stages:

| Stage | Component | Description |
|-------|-----------|-------------|
| 1 | **Query Router** | Classifies the research question by clinical entity type (diagnosis, medication, procedure, laboratory test, composite) |
| 2 | **ReAct Agent Loop** | Iteratively retrieves and refines candidate codes using a reasoning-and-acting loop (up to 50–200 steps depending on query complexity) |
| 3 | **Hybrid Retrieval** | Combines sparse TF-IDF retrieval and dense semantic embeddings, fused via Reciprocal Rank Fusion (RRF), with SNOMED CT graph expansion |
| 4 | **LLM Classification** | Phi-4 reviews each candidate code and assigns a confidence flag: `include`, `uncertain`, or `flag_for_review` |

---

## Key Features

- **Hybrid retrieval** — TF-IDF (n-gram range 1–5) + Nomic Embed Text v1.5 dense embeddings fused via RRF for high-recall candidate selection
- **SNOMED CT graph expansion** — traverses 16 relationship types (e.g. `is_a`, `finding_site`, `causative_agent`) to surface related codes not matched by text alone
- **ReAct-style agentic loop** — iterative reasoning that can issue multiple retrieval calls to refine coverage before committing to a final codelist
- **Confidence-flagged output** — every recommended code is tagged `include` / `uncertain` / `flag_for_review` with a plain-English clinical justification
- **Deterministic by default** — temperature set to 0 and seed fixed at 42 for reproducible results
- **Release-aware** — automatically detects and tracks SNOMED CT release versions for longitudinal auditability

---

## Project Structure

```
.
├── Clinical_Codelist_Generation_Pipeline.ipynb   # Main pipeline notebook
├── config.yaml                                    # All configurable parameters
├── requirements.txt                               # Pinned Python dependencies
├── .gitignore
└── README.md

# Directories created at runtime (not tracked in git):
├── data/
│   ├── raw/        # Source files: PCD Reference Set, SNOMED CT RF2 releases
│   └── processed/  # Chunked knowledge base and embeddings
└── results/        # Generated codelists and evaluation outputs
```

---

## Prerequisites

### Hardware
- GPU with CUDA 12.8 support (recommended: 16 GB+ VRAM for Phi-4 inference)
- 32 GB+ RAM for SNOMED CT processing

### Software
- Python 3.10+
- [Ollama](https://ollama.com/) installed and running locally
- Ollama model pulled: `ollama pull phi4`
- Google Colab (recommended) or a local Jupyter environment

### Data
The following NHS data files are required but are **not included** in this repository due to licensing restrictions:

| File | Description |
|------|-------------|
| `20250912_PCD_Refset_Content` | NHS Primary Care Domain Reference Set (September 2025) |
| SNOMED CT RF2 Descriptions | SNOMED CT Monolith RF2 PRODUCTION (September 2025) |
| SNOMED CT RF2 Relationships | SNOMED CT relationship hierarchy |
| Code usage statistics | NHS England SNOMED CT usage data (2024–25) |

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/dyarparvar/Accelerating-Clinical-Codelist-Generation-for-NICE-using-AI.git
cd Accelerating-Clinical-Codelist-Generation-for-NICE-using-AI
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Place data files

Copy the required NHS data files into `data/raw/` and update the paths in `config.yaml`.

### 4. Start Ollama and pull the model

```bash
ollama serve &
ollama pull phi4
```

### 5. Open and run the notebook

Open `Clinical_Codelist_Generation_Pipeline.ipynb` in Jupyter or Google Colab and run all cells sequentially. The notebook is fully self-contained with embedded markdown documentation at each stage.

---

## Configuration

All pipeline parameters are controlled via `config.yaml`:

```yaml
randomness_strategy:
  seed: 42                          # Fixed seed for reproducibility

data:
  source: 20250912_PCD_Refset_Content
  chunking:
    max_chars: 28672                # Context window chunk size
    codes_per_subchunk: 50

retrieval:
  embedding:
    model: nomic-ai/nomic-embed-text-v1.5
    device: cuda
  retrieving:
    sparse_min_score: 0.05
    dense_min_score: 0.6
    rrf_k: 60                       # Reciprocal rank fusion constant
  graph_expansion:
    enabled: true
    expansion_top_k: 10

llm:
  model: phi4
  temperature: 0                    # Deterministic output
  num_ctx: 16384
  max_codes_to_llm: 200

agent:
  model: phi4
  max_steps:
    simple: 50
    moderate: 100
    complex: 200
```

Key parameters to adjust for your use case:
- `data.source` — point to your PCD Reference Set version
- `retrieval.retrieving.dense_min_score` — lower to increase recall, raise to improve precision
- `agent.max_steps` — increase for broader queries; decrease to reduce runtime
- `llm.max_codes_to_llm` — caps the number of candidates sent to the LLM per iteration

---

## Pipeline Stages

### Stage 1 — Knowledge Base Preparation
- Loads the PCD Reference Set and enriches it with SNOMED CT taxonomy
- Detects the active SNOMED CT release version
- Builds semantic chunks with relationship expansion and generates dense embeddings using Nomic Embed Text v1.5

### Stage 2 — Query Routing
- Phi-4 classifies the free-text research question into one of five entity types: `diagnosis`, `laboratory_test`, `medication`, `procedure`, or `composite`
- Entity type determines which SNOMED CT relationship axes are prioritised during graph expansion

### Stage 3 — Hybrid Retrieval
- **Sparse retrieval**: TF-IDF with n-grams (1–5) against code descriptions and synonyms
- **Dense retrieval**: Cosine similarity search over Nomic embeddings
- **Graph expansion**: SNOMED CT hierarchy traversal across 16 relationship types
- Results fused via **Reciprocal Rank Fusion (RRF)**

### Stage 4 — ReAct Agent Loop
- Agent iteratively issues retrieval calls, inspects results, and decides whether to continue refining or commit to a final list
- Reasoning traces are logged for auditability
- Loop terminates when the agent is satisfied or `max_steps` is reached

### Stage 5 — LLM Classification
- Phi-4 reviews each candidate code with its description, synonyms, and retrieved context
- Assigns one of three confidence flags: `include`, `uncertain`, `flag_for_review`
- Provides a short plain-English clinical justification for each decision

### Stage 6 — Human Review
- Analyst reviews the AI-recommended codelist with confidence flags as guidance
- Approves or rejects each code; overrides are logged
- Final codelist is committed to git for a full audit trail

---

## Evaluation

The pipeline was benchmarked against **14 research questions** drawn from NHS clinical practice, compared against gold-standard codelists from [OpenCodelists](https://www.opencodelists.org/).

### Overall Performance

| Metric | All codes | Include-only tier |
|--------|-----------|-------------------|
| Macro F1 | 10.4% | 12.9% |
| Precision (median) | ~65% | — |
| Recall (median) | ~8% | — |

The system is deliberately **precision-biased**: it will not hallucinate codes outside the retrieved candidate pool, making it suitable as a first-pass filter rather than a stand-alone generator.

### Standout Results

| Research Question | Precision |
|-------------------|-----------|
| Liver Cirrhosis | 69.6% |
| Radiotherapy | 98.2% |

### Failure Mode Breakdown

| Failure Mode | Share |
|--------------|-------|
| Retrieval failures (relevant codes not retrieved) | 41.9% |
| Knowledge base gaps (codes absent from PCD/SNOMED snapshot) | 31.6% |
| LLM rejections (retrieved but incorrectly excluded) | 26.5% |

---

## Future Directions

1. **GraphRAG** — Replace flat vector retrieval with a Neo4j knowledge graph to enable multi-hop SNOMED CT reasoning and richer relationship traversal
2. **Demographic cluster injection** — Incorporate prevalence and demographic metadata to improve coverage for rare conditions
3. **Active learning loop** — Feed analyst override decisions back into the retrieval model to improve recall over time
4. **Regulatory validation** — Formal clinical review against NHS England and MHRA requirements before production deployment

---

## Acknowledgements

This work was carried out to support NICE's mission to provide evidence-based guidance for the NHS. The pipeline relies on:
- [SNOMED CT](https://www.snomed.org/) — maintained by SNOMED International
- [NHS Primary Care Domain Reference Set](https://digital.nhs.uk/services/terminology-and-classifications/snomed-ct) — NHS England
- [OpenCodelists](https://www.opencodelists.org/) — used as gold-standard evaluation reference
- [Phi-4](https://huggingface.co/microsoft/phi-4) — Microsoft Research
- [Nomic Embed Text v1.5](https://huggingface.co/nomic-ai/nomic-embed-text-v1.5) — Nomic AI
