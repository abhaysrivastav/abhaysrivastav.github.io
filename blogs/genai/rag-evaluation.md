---
layout: post
title: "How to Evaluate a RAG System: Metrics, Frameworks, and Production Signals"
date: 2025-05-19
permalink: /blogs/genai/rag-evaluation/
categories: [generative-ai, rag, evaluation]
tags: [RAG, RAGAS, TruLens, LLM evaluation, retrieval, faithfulness]
author: Abhay Srivastava
---

> This post is a companion to [RAG Architecture: A Practical Blueprint for Grounded AI Applications](https://abhaysrivastav.github.io/blogs/rag/).  
> That post covers the pipeline design. This one answers a different question: **once your RAG system is running, how do you know if it is working?**

---

## The core evaluation problem

A RAG system has two moving parts: a **retriever** and a **generator**. Either can fail independently.

- The retriever fetches the wrong chunks → the generator produces a correct-sounding answer from irrelevant context.
- The retriever fetches the right chunks → the generator ignores them and hallucinates.

This means a single end-to-end correctness score is not enough. You need to measure retrieval quality and generation quality separately, then track their combined effect.

---

## Layer 1: Retrieval evaluation

The retriever's job is to surface the chunks that contain the answer. If it fails here, no amount of LLM quality can compensate.

### Context precision

Of all chunks retrieved, what fraction were actually relevant?

```
Precision = Relevant chunks retrieved / Total chunks retrieved
```

Low precision means the retriever is flooding the prompt with noise, which increases cost, degrades answer quality, and can distract the model.

### Context recall

Of all relevant chunks that exist in the knowledge base, what fraction did the retriever surface?

```
Recall = Relevant chunks retrieved / Total relevant chunks in corpus
```

Low recall means the answer exists in your documents but the retriever is not finding it — the system will incorrectly say "I don't know."

### MRR (Mean Reciprocal Rank)

For each query, the reciprocal rank is 1/k where k is the rank position of the first relevant chunk.

```
MRR = (1/|Q|) × Σ (1/rank_k)
```

MRR tells you whether the most relevant chunk is appearing at the top of the ranked list or buried. A low MRR with decent recall means the right chunk exists in the retrieval window but is not being prioritized — a signal to improve your reranker.

### NDCG (Normalized Discounted Cumulative Gain)

NDCG evaluates the full ordering of retrieved chunks, not just the first hit. Highly relevant chunks appearing near the top contribute more to the score than relevant chunks appearing at position 15.

Use MRR for coarse ranking signal; use NDCG when you want to measure how well the entire ranked list is ordered.

---

## Layer 2: Generation evaluation

Given the retrieved context, did the LLM produce a correct, grounded answer?

### Faithfulness

Faithfulness measures whether every claim in the generated answer is supported by the retrieved context. A model that makes a factually correct statement that is not present in the context is still considered unfaithful — this is what distinguishes RAG faithfulness from general factual accuracy.

**How to measure it:**  
Decompose the answer into individual claims. For each claim, check whether the retrieved chunks support it. This can be automated with an LLM evaluator (see the frameworks section below).

```
Faithfulness = Supported claims / Total claims in answer
```

Faithfulness is your primary hallucination detection signal.

### Answer relevance

Does the answer actually address the question asked, or did the model go off-topic?

A model given retrieved chunks about employee leave policy might produce a technically faithful answer that still does not answer the specific question. Answer relevance catches this.

**How to measure it:**  
Ask an evaluator LLM to reverse-engineer the question from the answer. If the generated question matches the original question semantically, the answer is relevant.

### Answer correctness

Does the answer match the ground-truth answer?

This requires a labeled evaluation set — pairs of (question, expected answer). Measurement approaches:

| Method | When to use |
|---|---|
| Exact match | Short answers, IDs, named entities |
| Token F1 | Free-text answers where partial overlap matters |
| Semantic similarity | Use embedding cosine distance when paraphrasing is acceptable |
| LLM-as-judge | When correctness is contextual and hard to reduce to a score |

---

## Real-world example: customer support RAG

Consider a SaaS company that builds a RAG system over its product documentation to answer customer support questions.

**Scenario:** A user asks, *"How do I export my data in bulk?"*

| Failure mode | Root cause | Metric that catches it |
|---|---|---|
| Answer says "I don't know" when docs have the answer | Retriever missed the chunk (chunking boundary split the relevant section) | Context recall |
| Answer describes a deprecated export method | Retriever fetched an old doc version; reranker ranked it first | MRR / NDCG |
| Answer confidently describes a feature that does not exist | Model hallucinated beyond the retrieved context | Faithfulness |
| Answer explains exports in general but not bulk exports | Model answered the broader topic, not the specific question | Answer relevance |
| Answer is technically correct but contradicts the UI | Ground truth is stale; evaluation set not updated with releases | Answer correctness |

Each failure maps to a specific metric. This is why a single pass/fail score is insufficient.

---

## Layer 3: Automated evaluation frameworks

### RAGAS

[RAGAS (Retrieval-Augmented Generation Assessment)](https://docs.ragas.io/) is the most widely used open-source framework for RAG evaluation. It computes Faithfulness, Answer Relevance, Context Precision, and Context Recall automatically using an LLM evaluator — no human labelers required for initial setup.

**Input format:**

```python
from datasets import Dataset
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall

data = {
    "question": ["How do I export data in bulk?"],
    "answer": ["You can export data in bulk via Settings > Data > Export All."],
    "contexts": [["bulk export is available under Settings > Data", "export formats include CSV and JSON"]],
    "ground_truth": ["Navigate to Settings > Data > Export All to perform a bulk export."]
}

dataset = Dataset.from_dict(data)
result = evaluate(dataset, metrics=[faithfulness, answer_relevancy, context_precision, context_recall])
print(result)
```

RAGAS is the right starting point for any RAG evaluation pipeline. Official docs: [https://docs.ragas.io/](https://docs.ragas.io/)

### TruLens

[TruLens](https://www.trulens.org/) evaluates RAG systems using what it calls the **RAG Triad**: Answer Relevance, Context Relevance, and Groundedness (equivalent to faithfulness).

TruLens wraps your RAG chain and records every retrieval and generation step, which makes it useful for debugging as well as scoring.

```python
from trulens_eval import TruChain, Feedback, Huggingface, Tru
from trulens_eval.feedback import Groundedness

tru = Tru()
grounded = Groundedness(groundedness_provider=...)
f_groundedness = Feedback(grounded.groundedness_measure_with_cot_reasons).on_input_output()
```

Official docs: [https://www.trulens.org/trulens_eval/getting_started/](https://www.trulens.org/trulens_eval/getting_started/)

### LLM-as-judge

When ground truth labels are unavailable or when evaluation requires contextual judgment, you can use a capable LLM (GPT-4, Claude) as the evaluator.

**Prompt template for faithfulness scoring:**

```
You are an evaluation assistant.

Given the retrieved context and the generated answer, score the answer's faithfulness from 1 to 5.
Faithfulness means: every claim in the answer is directly supported by the context.

Context:
{context}

Answer:
{answer}

Return: {"score": <1-5>, "reasoning": "<one sentence>"}
```

Calibrate your LLM judge against human annotations on a sample set before trusting it at scale. A well-calibrated LLM judge achieves ~0.85 correlation with human raters on faithfulness (per the RAGAS paper).

---

## Production monitoring signals

Evaluation frameworks operate on labeled datasets. Production monitoring tracks quality on real traffic — no labels required.

| Signal | What it detects | How to collect |
|---|---|---|
| Retrieval hit rate | Queries returning zero useful chunks | Log retrieval result counts per query |
| Faithfulness score (sampled) | Drift in hallucination rate over time | Run LLM-as-judge on a 5–10% sample of live traffic |
| User corrections or thumbs-down | Real-world answer failures | Instrument UI feedback buttons |
| Query clustering | Topics where knowledge base has gaps | Embed queries and cluster; inspect empty-result clusters |
| Latency per pipeline stage | Retriever or reranker bottleneck | Log time at each stage separately |
| Cost per query | Token usage in retrieval context + generation | Track prompt token counts per stage |

**Key insight:** A rising faithfulness score on your evaluation set combined with a rising thumbs-down rate in production is a signal that your evaluation set no longer represents real user questions. Refresh your eval set regularly with queries sampled from production logs.

---

## Evaluation pipeline architecture

```
                    ┌─────────────────────────────────────┐
                    │           Evaluation pipeline        │
                    └─────────────────────────────────────┘

  Labeled test set          Real traffic (sampled)
        │                           │
        ▼                           ▼
  ┌──────────┐              ┌──────────────┐
  │  RAGAS   │              │ LLM-as-judge │
  │ TruLens  │              │   (online)   │
  └──────────┘              └──────────────┘
        │                           │
        ▼                           ▼
  ┌─────────────────────────────────────────┐
  │          Metric dashboard               │
  │  Precision · Recall · MRR · NDCG        │
  │  Faithfulness · Relevance · Correctness │
  │  Hit rate · Latency · Cost              │
  └─────────────────────────────────────────┘
        │
        ▼
  Feedback loop → retune chunking, retrieval, prompts
```

---

## Evaluation set construction

The quality of your evaluation is bounded by the quality of your evaluation set. Guidelines:

- **Minimum size:** 100–200 labeled (question, answer, ground-truth) triples for meaningful signal.
- **Source:** Sample from real user queries, not synthetically generated questions — real questions have a distribution that synthetic ones miss.
- **Coverage:** Include adversarial cases — questions where the answer is not in the knowledge base, ambiguous questions, and questions that require combining multiple chunks.
- **Refresh cadence:** Update the eval set quarterly, or whenever significant new content is added to the knowledge base.
- **Stratify by domain:** For a multi-domain knowledge base, ensure the eval set covers each domain proportionally.

**Synthetic evaluation bootstrapping:**  
If you have no labeled data at all, use an LLM to generate (question, ground-truth) pairs from your documents as a bootstrap. Treat these as low-confidence labels — they are useful for catching obvious failures but should be supplemented with human-labeled data before making production decisions.

---

## Common evaluation mistakes

| Mistake | Why it fails | Fix |
|---|---|---|
| Measuring only end-to-end correctness | Cannot diagnose whether retriever or generator is the bottleneck | Measure retrieval and generation independently |
| Evaluating only on the training domain | System may fail on edge cases not in the eval set | Adversarially sample edge cases and out-of-domain queries |
| Using the same LLM for generation and evaluation | The evaluator LLM may rate its own outputs favorably | Use a different model family for evaluation |
| Static eval set | Real user query distribution shifts over time | Refresh eval set from production logs every quarter |
| Ignoring retrieval latency | A faithful system that takes 8 seconds per query will see user abandonment | Include latency and cost in your evaluation dashboard |

---

## Summary

A RAG system can fail at retrieval, at generation, or at both. A single accuracy metric cannot distinguish between these failure modes. The correct evaluation approach is:

1. **Retrieval layer:** Measure Context Precision, Context Recall, MRR, NDCG.
2. **Generation layer:** Measure Faithfulness, Answer Relevance, Answer Correctness.
3. **Automated pipeline:** Use [RAGAS](https://docs.ragas.io/) or [TruLens](https://www.trulens.org/) to compute these at scale.
4. **Production monitoring:** Track hit rate, faithfulness drift, user feedback, and latency on live traffic.
5. **Close the loop:** Use evaluation results to retune chunking strategy, retriever parameters, reranker selection, and prompt design.

Evaluation is not a one-time exercise. It is a continuous feedback loop that determines whether your RAG system earns production trust.

---

## References and further reading

- [RAGAS: Automated Evaluation of Retrieval Augmented Generation (paper)](https://arxiv.org/abs/2309.15217)
- [RAGAS official documentation](https://docs.ragas.io/)
- [TruLens documentation](https://www.trulens.org/trulens_eval/getting_started/)
- [LlamaIndex RAG evaluation guide](https://docs.llamaindex.ai/en/stable/optimizing/evaluation/evaluation/)
- [LangSmith evaluation framework (LangChain)](https://docs.smith.langchain.com/evaluation)
- [Benchmarking LLMs as evaluators (paper)](https://arxiv.org/abs/2306.05685)

---


