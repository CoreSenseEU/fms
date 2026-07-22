# P3 — Knowledge Elicitor

> **Pattern**: Knowledge Elicitor  
> **Epistemic Role**: Unstructured text → structured KB fragments  
> **Pipeline Stage**: `Extract` → `ModelPropose`  
> **FM Eligible**: Yes  
> **FM Excluded From**: `Integrate`, `Evaluate`

---

## Description

A Foundation Model deployed as a **Knowledge Elicitor** extracts structured RDF/OWL triples, entity-relation pairs, or ontology-aligned fragments from unstructured text sources such as scientific papers, technical manuals, news feeds, or web pages.

The extracted triples form the raw material for populating or extending the CoreSense Knowledge Base. They always enter the `CandidateBuffer` first, where the `ConsistencyIntegrator` verifies them before admission.

---

## Epistemic Contract

```
recallTarget          : ≥ 0.80
precisionFloor        : ≥ 0.75
warrantsCorrectness   : false
requiresGrounding     : true
outputSchema          : {subject, predicate, object, confidence, source_span}
```

---

## Input / Output

| Slot | Type | Description |
|------|------|-------------|
| `sourceText` | `String` | Unstructured text document |
| `targetOntology` | `OntologyRef` | The ontology schema to align extractions to |
| `triples` | `CandidateAssertion[*]` | Extracted triples, each tagged with provenance and source span |

---

## Suitable FM Instances

| FM | Deployment Mode | Notes |
|----|-----------------|-------|
| Claude Opus (Anthropic) | Commercial API | Excellent structured output, long context |
| Claude Sonnet (Anthropic) | Commercial API | Good balance cost / quality |
| Llama 3.1-70B (Meta) | Local OSS (Ollama) | Strong at instruction-following extraction |
| Qwen2-72B | Local OSS (Ollama) | Good multilingual extraction |
| Kimi K3 | Commercial API | Strong for broad-domain scientific text |

---

## Prompt Engineering Notes

Knowledge elicitors require careful prompt construction:

1. **Provide the ontology schema** — include the target class/property definitions in the system prompt
2. **Use few-shot examples** — two to three extraction examples from the target domain significantly improve precision
3. **Request confidence scores** — instruct the FM to rate each triple's certainty
4. **Request source spans** — ask the FM to cite the exact text span that evidences each triple

---

## SysMLv2 Snapshot

```sysml
part claudeKEOrgan : KnowledgeElicitorOrgan {
    attribute :>> contract = EpistemicContract {
        recallTarget        = 0.82;
        precisionFloor      = 0.78;
        warrantsCorrectness = false;
        requiresGrounding   = true;
        domainRef           = "https://coresense.eu/ontologies/manufacturing#";
    };
}
```

---

## When to Use

- When the KB must be populated from large text corpora
- When domain experts produce natural-language documentation that must be formalised
- When scraping structured knowledge from web sources or scientific literature

## When NOT to Use

- When the source text is already structured (use a parser or SPARQL CONSTRUCT directly)
- When the target ontology has no machine-readable schema to guide the FM
