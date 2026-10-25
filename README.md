# End-to-End LLM-as-a-Judge Pipeline for Clinical Summaries

An enterprise-grade, multi-model evaluation and self-correction architecture designed to audit, refine, and benchmark clinical text summarization using **Anthropic Claude** and **Google Gemini**.

---

## Overview

Summarizing unstructured medical transcriptions safely requires rigorous guardrails against hallucinations, omissions, and unsafe medical advice. This repository implements an end-to-end evaluation pipeline that moves beyond simple LLM calls by introducing deterministic computational sensors, multi-turn feedback revision loops, anchored scoring rubrics, mutation testing, and rigorous statistical validation.

---

## Core Architectural Features

The pipeline is built around eight foundational review mechanisms:

1. **Computational Sensors**: Every generated summary undergoes automated deterministic checks for length (3–5 sentences), number grounding (ensuring no numeric values or dosages are introduced that do not exist in the source note), and non-empty content constraints.


2. **Self-Correction Feedback Loop**: Failed computational checks and low-scoring model rationales are automatically fed back into the generator model for targeted multi-round revisions.


3. **Anchored Rubrics**: Evaluates five distinct clinical dimensions (**Factuality, Completeness, Relevance, Safety, and Presentation**) on a 1–5 scale, backed by concrete behavioral anchors for levels 1, 3, and 5 to minimize subjective drift.


4. **Mutation Testing**: Evaluates judge robustness by intentionally planting errors (e.g., unsupported claims, dropped diagnoses, unsafe advice, off-topic sentences, and disorganized structures) to verify whether each judge detects them.


5. **Gold Set Validation**: Supports exporting frozen summaries for human clinician annotation, establishing a rigorous ground-truth validation framework against human consensus.


6. **Sensor Health & Parse Safety**: Unparseable or malformed LLM outputs are explicitly recorded as missing status (`missing`) rather than dangerously imputed as neutral scores (like a default score of 3).


7. **Honest Reporting & Statistical Rigor**: Employs bootstrap Confidence Intervals (CIs), Quadratic Weighted Kappa (QWK) and Spearman correlation metrics, minimum sample size gates ($N \ge 100$), and constant-score guards.


8. **Cost Staging & Caching**: Integrates local SHA-256 caching for all LLM calls and stages dual-model evaluations (running free computational checks on all notes while strategically staging LLM judges).



---

## Tech Stack & Requirements

* **Languages**: Python 3.12+


* **Libraries**: `pandas`, `numpy`, `scikit-learn`, `scipy`, `requests`


* **LLM Providers**: Anthropic Claude (`claude-sonnet-4-6`) & Google Gemini (`gemini-3.5-flash-lite`)


* **Dataset**: Medical Transcription Sample Dataset (`mtsamples.csv`)



---

## ⚙️ Installation & Usage

### 1. Clone and Setup Environment

Ensure you have your API keys set up as environment variables or attached via Kaggle secrets:

```bash
export ANTHROPIC_API_KEY="your-anthropic-key"
export GOOGLE_API_KEY="your-google-key"

```

### 2. Run the Pipeline

Execute the pipeline script specifying your dataset path and output directory:

```bash
python judge_pipeline.py --data path/to/mtsamples.csv --out results_v3 --n-specialties 10 --notes-per-specialty 10

```

### Command-Line Arguments

* `--data`: Path to the input CSV file (`mtsamples.csv`).


* `--out`: Directory name for saving results, metrics, and logs (default: `results_v3`).


* `--n-specialties`: Number of medical specialties to sample from (default: `10`).


* `--notes-per-specialty`: Number of clinical notes sampled per specialty (default: `10`).


* `--max-rounds`: Maximum revision loop rounds; setting to `1` disables self-correction (default: `2`).


* `--mutation-notes`: Number of samples utilized for mutation testing; setting to `0` disables it (default: `6`).


* `--export-gold`: Flag to export frozen summaries for clinician evaluation.


* `--validate-gold`: Flag to run validation against human clinician gold sets.



---

## Sample Evaluation Output

A typical run across 100 clinical notes generates comprehensive tracking data, agreement matrices, and health summaries:

```text
## Judge agreement (LLM vs LLM; not a validity measure)
   dimension   n  dropped    qwk  qwk_lo  qwk_hi  spearman       label
  factuality 100        0  0.477  -0.039   0.857     0.344   UNCERTAIN
completeness 100        0 -0.027  -0.080  -0.024    -0.056   DISAGREES
   relevance 100        0  0.798   0.793   0.798     1.000      AGREES
      safety 100        0  0.797   0.607   1.000     0.583   UNCERTAIN
presentation 100        0    NaN     NaN     NaN       NaN   NOT MEASURABLE

```

---

## Output Artifacts

* `results.csv`: Detailed per-note metrics, round counts, and evaluation scores.


* `agreement.csv`: Inter-rater agreement statistics and QWK confidence intervals.


* `health.csv`: Tracking metrics for parser health and missing rate monitoring.


* `mutation_summary.csv`: Performance metrics tracking how effectively judges catch planted errors.


* `report.md`: Summary markdown report containing execution details and caveats.



---

## 📝 License

This project is released under the MIT License.
