# `rag-hybrid-retrieval`

## Why This Repository Exists

The previous repositories in this series established two facts:

* A minimal RAG system can behave *correctly* and still refuse to answer
* Most refusals are caused by **retrieval ranking failures**, not missing information

However, one critical question remained unanswered:

> **Are these retrieval failures caused by semantic abstraction and low lexical overlap — and can sparse signals surface evidence that dense retrieval misses?**

This repository exists to answer **only that question**.

This is not an optimization demo.
It is a **controlled retrieval experiment**.

---

## What Problem This System Solves

This repository tests whether **hybrid retrieval (dense + sparse)** improves **evidence surfacing** under the same constraints used in Week 2.

Specifically, it asks:

> *When dense retrieval ranks relevant evidence far below Top-K, does introducing sparse lexical signals move that evidence closer to the top — without changing generation behavior?*

The focus is **retrieval-layer behavior only**.

---

## What This System Explicitly Does NOT Do

This repository deliberately avoids:

* Changing chunking strategy
* Changing embedding models
* Retrieval reranking or learned fusion
* Prompt engineering
* LLM grading or answer correctness evaluation
* Agent behavior or tool use

If you are looking for a system that “answers better,” this is not it.

That comes later.

---

## System Relationship to Previous Weeks

This repository builds directly on:

* [rag-minimal-control](https://github.com/Arnav-Ajay/rag-minimal-control)
  A strict, minimal RAG control system

* [rag-retrieval-eval](https://github.com/Arnav-Ajay/rag-retrieval-eval)
  A retrieval observability and evaluation harness

All `rag-retrieval-eval` components remain **frozen**, including:

* Corpus
* Chunking
* Embeddings
* Dense similarity function
* Top-K passed to the LLM (K = 4)

The **only change** is the introduction of a sparse retriever and an explicit hybrid merge stage.

---

## System Overview

**Repo Contract:**

* Inputs: static PDF corpus (unchanged)
* Query input: deterministic evaluation questions
* Output:

  * Dense, sparse, and hybrid retrieval rankings
  * Rank of first relevant chunk
  * Top-K inclusion flags
* Non-goal: generating correct answers

---

## Retrieval Pipeline `rag-hybrid-retrieval`

```
Document → Chunk → Embed
                 ↘
                  Dense Retriever +
Query ───────────→ Sparse Retriever
                     ↓
                Explicit Hybrid Merge
                     ↓
                 Ranked Candidates
                     ↓
              Top-K → Generator (unchanged)
```

---

## Dense vs Sparse Retrieval (Conceptual)

**Dense retrieval**

* Uses embedding similarity
* Captures semantic relationships
* Rank reliability degrades quickly beyond a small neighborhood

**Sparse retrieval (BM25)**

* Uses exact lexical overlap
* Excels at definitions, lists, and procedural text
* Produces sharp but brittle signals

Hybrid retrieval exists to test whether **lexical anchors can correct semantic misranking**.

---

## Hybrid Merge Logic (Explicit & Inspectable)

This repository uses **rule-based hybrid merging**, not score fusion.

Each chunk is annotated with:

* Dense rank (if present)
* Sparse rank (if present)

Priority rules:

1. Dense ≤ D **and** Sparse ≤ S
2. Sparse ≤ S only
3. Dense ≤ D only
4. Otherwise → dropped

Where:

* **D** = dense trust threshold (semantic reliability window)
* **S** = sparse trust threshold (lexical confidence window)

These are **trust boundaries**, not tuning knobs.

No reranking is performed.

---

## Evaluation Methodology (Unchanged from `rag-retrieval-eval`)

All evaluation uses the **existing `rag-retrieval-eval` harness**.

Metrics reported:

* **Context Recall @ K**
* **Rank of First Relevant Chunk**
* Dense vs Hybrid deltas (descriptive only)

Evaluation is stratified where possible by **question intent**:

* factual
* rationale
* scope / inventory

No new metrics are introduced.

---

## Key Findings

From the controlled experiment:

* Sparse retrieval frequently surfaces the gold (answer-bearing) chunk at very high rank
* Hybrid retrieval consistently improves the **rank position** of relevant evidence compared to dense-only
* However, without reranking, these gains **rarely translate into Top-K (K=4) inclusion**

**Interpretation:**

> Dense retrieval failures are often caused by lexical mismatch rather than missing corpus content.
> Sparse signals successfully surface this evidence, but converting surfaced evidence into Top-K dominance requires a separate reranking stage.


---

## Why This Matters

Most RAG systems conflate:

* Evidence surfacing
* Evidence ranking
* Answer quality

This repository separates them.

It shows that:

* Hybrid retrieval can **expose hidden evidence**
* But exposure alone is insufficient
* Ranking must be treated as a first-class design problem

---

## How to Run

```bash
pip install -r requirements.txt
python app.py --run-retrieval-eval --questions-csv data/retrieval_eval.csv
```

Outputs:

* `data/retrieval_evaluation_results_sanitized.csv`
  - Row-level retrieval results (IDs + ranks only)
* `data/summary_overall.csv`
  - Aggregate retrieval metrics (overall)
* `data/summary_by_intent.csv`
  - Aggregate retrieval metrics stratified by question intent

These artifacts constitute the complete Week-3 evaluation surface.
Published evaluation artifacts exclude document and chunk text to avoid
distributing source content; only rank-based retrieval metrics are included.
