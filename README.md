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

## System Relationship to Previous Repos

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
* Excels at definitions, lists, formulas, and procedural text
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
* Dense vs Sparse vs Hybrid deltas (descriptive only)

Evaluation is stratified where possible by **question intent**:

* definitional
* mechanistic
* formula
* structural
* comparative
* procedural
* enumeration
* factual
* conceptual

No new metrics are introduced.

---

## Results Summary (Added)

### Overall Retrieval Outcomes (Week 3)

| Metric                                      | Value   |
| ------------------------------------------- | ------- |
| Total questions                             | 54      |
| Questions with any relevant retrieval       | ~50     |
| Median rank (first relevant, any retriever) | ~16–18  |
| % rank > Top-K (K=4)                        | ~85–90% |
| **Top-K success — Dense**                   | ~0–2%   |
| **Top-K success — Sparse**                  | ~25–30% |
| **Top-K success — Hybrid**                  | ~30–35% |

**Key shift from `rag-retrieval-eval`:**
Hybrid retrieval breaks the *zero-success regime* by allowing relevant evidence to reach Top-K in a non-trivial subset of cases.

---

### Intent-Based Retrieval Behavior (Hybrid)

| Intent Type                | n | Median Rank | % Top-K  |
| -------------------------- | - | ----------- | -------- |
| Definition                 | 7 | ~14–16      | ~30%     |
| Mechanistic                | 8 | ~10–12      | **~50%** |
| Formula                    | 5 | ~9–11       | **~60%** |
| Structural                 | 6 | ~12–14      | ~40%     |
| Comparative                | 6 | ~16–18      | ~25%     |
| Procedural / Process       | 5 | ~15–18      | ~20%     |
| Enumeration                | 6 | ~18–20      | ~15%     |
| Factual (datasets / names) | 6 | ~20–25      | ~10%     |
| Conceptual / abstract      | 5 | ~18–22      | ~10%     |
| Historical                 | 1 | ~20+        | 0%       |

**Observed pattern:**
Sparse retrieval disproportionately benefits **mechanistic, formulaic, and explicitly stated content**, while dense retrieval remains unstable across all intents.

---

## Key Findings

* Sparse retrieval frequently surfaces the gold (answer-bearing) chunk at very high rank
* Hybrid retrieval **strictly dominates dense-only retrieval** for evidence surfacing
* However, without reranking, surfaced evidence **often fails to achieve stable Top-K dominance**

**Interpretation:**

> Hybrid retrieval resolves *presence* failures, not *prioritization* failures.

---

## What this Proves

* Dense retrieval failure is often driven by **lexical mismatch**, not missing information
* Sparse signals restore lexical grounding and expose hidden evidence
* Hybrid retrieval alone is **insufficient** to guarantee Top-K inclusion

This establishes **ranking, not retrieval**, as the next bottleneck.

---

## Why This Matters

Most RAG systems conflate:

* Evidence surfacing
* Evidence ranking
* Answer quality

This repository separates them.

It shows that:

* Hybrid retrieval can **expose hidden evidence**
* Exposure alone is insufficient
* Ranking must be treated as a **first-class system component**

This directly motivates the next stage: **explicit reranking under controlled conditions**.

---

## How to Run

```bash
pip install -r requirements.txt
python app.py # simple dense retrieval
python app.py --hybrid-retrieval # hybrid retrieval
python app.py --run-retrieval-eval --questions-csv data/retrieval_eval.csv # export results
```

Outputs:

* `data/results_and_summaries/questions_retrieval_results_inspect_50.csv`
  – Row-level retrieval results

* `data/results_and_summaries/summary_overall.csv`
  – Aggregate retrieval metrics (overall)

* `data/results_and_summaries/summary_by_intent.csv`
  – Aggregate retrieval metrics stratified by question intent


---