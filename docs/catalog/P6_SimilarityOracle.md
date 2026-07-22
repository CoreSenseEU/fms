# P6 — Similarity Oracle

> **Pattern**: Similarity Oracle  
> **Epistemic Role**: Semantic similarity and analogical retrieval  
> **Pipeline Stage**: `Extract` (retrieval-augmented input enrichment)  
> **FM Eligible**: Yes  
> **FM Excluded From**: `Integrate`, `Evaluate`

---

## Description

A Foundation Model deployed as a **Similarity Oracle** encodes observations, assertions, or queries as dense embedding vectors and retrieves semantically similar items from the CoreSense KB or an external corpus. It supports both nearest-neighbour lookup and analogical reasoning ("X is to Y as A is to ?").

Unlike the other FM patterns, the Similarity Oracle does not generate natural-language outputs. It produces ranked similarity scores over existing KB items, which the `Extract` stage uses to enrich the observation context before it reaches `ModelPropose`.

---

## Epistemic Contract

```
recallTarget          : nDCG@10 ≥ 0.80 on domain retrieval benchmarks
warrantsCorrectness   : false (similarity is not logical equivalence)
requiresGrounding     : false
latencySLA_ms         : ≤ 20  (cached) / ≤ 200 (fresh embeddings)
embeddingDimensions   : ≥ 512
```

---

## Input / Output

| Slot | Type | Description |
|------|------|-------------|
| `query` | `String or Assertion` | Query item to find similar items for |
| `corpus` | `CorpusRef` | KB subset or external index to search |
| `topK` | `Integer` | Number of nearest neighbours to return |
| `results` | `List<{item, similarity: Float}>` | Ranked similar items with similarity scores |

---

## Suitable FM Instances

| FM | Deployment Mode | Notes |
|----|-----------------|-------|
| text-embedding-3-large (OpenAI) | Commercial API | High-quality dense embeddings |
| text-embedding-3-small (OpenAI) | Commercial API | Cost-efficient, lower dimensionality |
| BGE-M3 (BAAI) | Local OSS | Best local multilingual embeddings |
| E5-large-v2 | Local OSS | Strong English retrieval |
| nomic-embed-text | Local OSS (Ollama) | Good local alternative |
| mxbai-embed-large | Local OSS (Ollama) | Very good local embeddings |

---

## Architecture Note

The Similarity Oracle typically operates alongside a **vector index** (e.g., FAISS, Qdrant, Milvus, ChromaDB) that stores pre-computed embeddings of KB assertions. At query time, the Oracle embeds the query and performs approximate nearest-neighbour search against the index—no FM call is needed for items already indexed.

```
Query Assertion
      │
      ▼
 FM (embed)──▶ query vector
                    │
                    ▼
          Vector Index (FAISS/Qdrant)
                    │
                    ▼
          Top-K similar assertions
```

---

## SysMLv2 Snapshot

```sysml
part bgeOracle : CognitiveOrgan {
    attribute :>> organType = CognitiveOrganType::SimilarityOracle;
    attribute :>> contract  = EpistemicContract {
        recallTarget        = 0.82;
        precisionFloor      = 0.80;
        warrantsCorrectness = false;
        requiresGrounding   = false;
        domainRef           = "https://coresense.eu/ontologies/general#";
    };
    attribute vectorIndexRef : String = "qdrant://localhost:6333/coresense-kb";
}
```

---

## When to Use

- When the `ModelPropose` stage benefits from analogical context (retrieved similar past cases)
- When the system must find semantically related KB items without exact symbolic match
- When implementing retrieval-augmented generation (RAG) over the CoreSense KB

## When NOT to Use

- When exact logical matching is sufficient (use SPARQL SELECT directly)
- When the KB is small enough that exhaustive comparison is feasible
