# `rag-retrieval-eval`

## Why This Repository Exists

[rag-minimal-control](https://github.com/Arnav-Ajay/rag-minimal-control) established a **minimal RAG control system** and showed that, under strict evidence constraints, the model consistently refused to answer.

That refusal was *correct* — but **unexplained**.

This repository exists to answer a narrower and more fundamental question:

> **When a RAG system refuses to answer, is it because the answer is absent — or because retrieval failed?**

This is not an optimization repo.
It is a **retrieval observability and evaluation harness**.

---

## What Problem This System Solves

This system makes retrieval behavior:

* Observable
* Measurable
* Falsifiable

It allows a human to determine whether retrieval failures are due to:

* Corpus absence
* Representation collapse
* Lexical bias
* Ranking cutoff
* Similarity score pathologies

without changing the retrieval system itself.

---

## What This System Explicitly Does NOT Do

This implementation deliberately avoids:

* Changing embedding models
* Improving chunking strategies
* Retrieval reranking
* Semantic search
* LLM-based grading
* Answer correctness evaluation
* Any attempt to “fix” retrieval

If retrieval looks bad, that is the expected and desired outcome.

---

## System Relationship to rag-minimal-control

This repository is behaviorally identical to `rag-minimal-control` with respect to retrieval and generation, with:
* Identical corpus
* Identical chunking
* Identical embeddings
* Identical similarity metric
* Identical Top-K retrieval for generation

The **only difference** is the addition of retrieval observability.

`rag-minimal-control` answers:

> “Does the system behave correctly under constraint?”

`rag-retrieval-eval` answers:

> “Why does retrieval fail when answers exist?”

---

## System Overview

**Repo Contract:**

* Inputs: same static PDF corpus as `rag-minimal-control`
* Query input: deterministic evaluation questions
* Output:

  * Ranked retrieval results (Top-N for inspection)
  * Similarity scores
  * Human relevance labels (external)
* Non-goal: generating correct answers

---

## Retrieval Pipeline (Unchanged)

```
Document → Chunk → Embed → Retrieve → Rank → Top-K → Generate
```

**Important distinction:**

* **Top-K (K=4)** → passed to the LLM (unchanged)
* **Top-N (e.g., 20–50)** → logged for analysis only

---

## What Is Being Measured

This repository supports manual evaluation of:

* **Context Recall @ K**
* **Rank of First Relevant Chunk**
* **Retrieval Miss Rate**
* **False Similarity Signals**

All relevance judgments are made by a human.

No automated scoring is used.

---

## Expected Observations

This system is expected to show:

* High similarity scores for semantically irrelevant chunks
* Near-identical similarity values across many chunks
* Relevant chunks appearing far below Top-K
* Complete retrieval failure even when answers exist

These observations are prerequisites for improvement — not failures to be corrected in this repository.

These are not bugs.
They are **diagnostic signals**.

---

## Why This Matters

Most RAG systems fail silently.

By separating:

* **Retrieval failure**
* **Corpus absence**
* **Generation refusal**

this repository establishes a causal foundation for all future improvements.

No optimization is meaningful until failure is measured.

---

## How to Run
- create a folder `data/` and add pdf files in it. (just 1 is fine)
- create a .env file in root dir and add you OpenAI API key ket as: `OPENAI_API_KEY=<your-api-key>`
- install dependencies: `pip install -r requirements.txt`
- Run a Single Query (Baseline Behavior): This mirrors rag-minimal-control behavior and shows Top-K retrieval + refusal.
```bash
  python app.py --query "What is the purpose of this document?"
```

## Retrieval Observability & Evaluation
- Export Corpus Chunks (Debugging / Ground Truth Construction)
  * Exports all chunks with document IDs to a CSV file.
  ```bash
  python app.py --export-chunks
  ```
  * Output: `data/chunks_debug.csv`
  * This file is used to manually identify gold (answer-bearing) chunks.

- Run Corpus Diagnostics
  * Prints document-level chunk counts and chunk → document mappings.
  ```bash
  python app.py --corpus-diag
  ```
- Run Retrieval Evaluation (Core Week-2 Artifact)
  * Runs retrieval for a predefined question set and logs ranked results.
  ```bash
  python app.py --run-retrieval-eval --questions-csv data/retrieval_eval.csv
  ```
  * Output: `data/retrieval_evaluation_results.csv`
  * This CSV is the primary evaluation artifact used for analysis.

## Command-Line Arguments
  * `--query`: Run a single retrieval + generation query
  * `--export-chunks`: Export all chunks to CSV
  * `--corpus-diag`: Print corpus diagnostics
  * `--run-retrieval-eval`: Run retrieval evaluation harness
  * `--questions-csv`: Path to evaluation question file

## Text Normalization Note

PDF text extraction exhibited common mojibake artifacts
(e.g., mis-decoded dashes, arrows, and mathematical symbols).

These were corrected using deterministic Unicode normalization
and explicit glyph replacement. No semantic rewriting or
content alteration was performed.

## Evaluation Artifacts

This repository produces a structured evaluation table containing:

- Question
- Gold (human-identified) chunk ID
- Ranked retrieved chunk IDs
- Rank of first relevant chunk
- Whether the relevant chunk appears in Top-K

This table is the primary diagnostic output of the system.
