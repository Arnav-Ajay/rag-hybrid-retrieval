## `rag-hybrid-retrieval`

### Measuring Whether Sparse Signals Change Retrieval Ranking — Nothing More

---

## Why This Repository Exists

Previous repositories in this series established two empirical facts:

1. **[`rag-minimum-copntrol`](https://github.com/Arnav-Ajay/rag-minimal-control)**: A minimal RAG system can behave *correctly* and still refuse to answer
2. **[`rag-retrieval-eval`](https://github.com/Arnav-Ajay/rag-retrieval-eval)**: Most refusals occur even when the answer exists in the corpus, due to retrieval ranking failure

These findings isolate **dense retrieval ranking** as a bottleneck.

This repository asks **one constrained question**:

> When dense retrieval fails to rank relevant evidence near the top, does adding sparse lexical retrieval change the *ranking position* of that evidence?

This repository does **not** evaluate answer quality.
It measures **retrieval behavior only**.

---

## What This Repository Does

This system introduces **hybrid retrieval**, combining:

* dense semantic retrieval (embeddings)
* sparse lexical retrieval (BM25)

and measures their effect on:

* rank of the first answer-bearing chunk
* Top-K inclusion under a fixed generation budget (K = 4)

All other system components are held constant.

---

## What This Repository Explicitly Does NOT Do

This repository does **not**:

* change chunking strategy
* modify embedding models
* introduce reranking
* tune retrieval weights
* evaluate generation correctness
* claim end-to-end system improvement

Any observed changes apply **only to retrieval ranking**.

---

## System Lineage

This repository builds directly on:

* **`rag-minimal-control`** — establishes a strict RAG baseline
* **`rag-retrieval-eval`** — adds retrieval observability and gold labels

The following remain **frozen**:

* corpus
* chunking
* embeddings
* dense similarity metric
* Top-K passed to the generator (K = 4)

The **only change** is the addition of sparse retrieval and a deterministic merge step.

---

## Retrieval Pipeline

```
Document → Chunk → Embed
                 ↘
                  Dense Retriever
Query ───────────→ Sparse Retriever
                     ↓
                Explicit Hybrid Merge
                     ↓
                 Ranked Candidates
                     ↓
              Top-K → Generator (unchanged)
```

---

## Hybrid Merge (Design, Not Optimization)

Hybrid retrieval is implemented using a **rule-based merge**:

* Dense and sparse retrievers run independently
* Chunks are retained if they fall within predefined rank windows
* No score fusion or learning is applied

This merge is:

* deterministic
* inspectable
* **not** claimed to be optimal

---

## Evaluation Methodology

Evaluation reuses the **unchanged harness** from `rag-retrieval-eval`.

Measured signals:

* rank of first relevant (gold) chunk
* Top-K inclusion (K = 4)
* retrieval outcomes stratified by question intent

No generation output is evaluated.

---

## Results (Strictly What the Data Shows)

### Overall Retrieval Behavior

* Dense retrieval alone almost never places relevant evidence in Top-K
* Sparse retrieval sometimes places relevant evidence at substantially higher ranks
* Hybrid retrieval increases the frequency of Top-K inclusion relative to dense-only retrieval
* In the majority of cases, relevant evidence **still appears below K**

Hybrid retrieval changes **ranking distributions**, not ranking reliability.

---

### Intent-Stratified Observations

When results are grouped by question intent:

* Mechanistic and formulaic questions show higher rates of improved ranking
* Comparative, conceptual, and historical questions remain difficult
* Improvements are uneven and non-universal across intents

These are **descriptive observations**, not causal explanations.

---

## What This Repository Establishes

* Dense retrieval failure is often not due to corpus absence
* Sparse lexical signals can surface evidence that dense retrieval ranks poorly
* Hybrid retrieval increases the probability of Top-K inclusion

---

## What This Repository Does NOT Establish

* Stable Top-K dominance
* Improved answer quality
* Optimal hybrid retrieval strategies
* Generalization beyond this experimental setup

---

## Core Conclusion

> Hybrid retrieval resolves **evidence presence** failures, not **prioritization** failures.

Even when relevant evidence is surfaced, **ranking instability remains the dominant bottleneck**.

---

## Why This Matters

Many RAG systems conflate:

* evidence surfacing
* evidence ranking
* answer quality

This repository keeps them separate.

It demonstrates that improving retrieval inputs alone is insufficient to guarantee useful generation.

---

## Next Step in the Series

The next repository isolates the **ranking problem itself**:

> Given that the correct evidence is already present in the candidate set, can reranking reliably promote it into Top-K?

That question is addressed in **[`rag-reranking-playground`](https://github.com/Arnav-Ajay/rag-reranking-playground)**.

---