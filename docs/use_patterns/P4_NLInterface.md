# P4 — Natural Language Interface

> **Pattern**: Natural Language Interface  
> **Epistemic Role**: Natural-language ↔ formal query bridge  
> **Pipeline Stage**: Bidirectional interface (outside core pipeline)  
> **FM Eligible**: Yes  
> **FM Excluded From**: Direct KB writes

---

## Description

A Foundation Model deployed as a **Natural Language Interface** bridges human operators (or external NL-producing systems) to the CoreSense formal knowledge layer. It operates in two directions:

- **NL → Formal**: translates natural-language questions or commands into SPARQL queries, Prolog goals, or OWL-DL expressions
- **Formal → NL**: translates formal result sets, proof traces, or ontology fragments back into readable explanations

The NL Interface does **not** modify the KB. It generates query strings only, which are then executed by the formal query layer. This maintains the FM exclusion from `Integrate` and `Evaluate`.

---

## Epistemic Contract

```
NL→Formal translation accuracy : ≥ 0.85 on domain vocabulary
warrantsCorrectness             : false (query generation, not fact assertion)
requiresGrounding               : false
mustNotWriteToKB                : true   (hard constraint)
```

---

## Input / Output

### NL → Formal Direction

| Slot | Type | Description |
|------|------|-------------|
| `nlQuery` | `String` | Natural language question or command |
| `kbSchema` | `OntologyRef` | Schema context for the target KB |
| `formalQuery` | `String` | Generated SPARQL / Prolog / OWL-DL query |

### Formal → NL Direction

| Slot | Type | Description |
|------|------|-------------|
| `formalResult` | `QueryResultSet` | Output of formal query execution |
| `verbosity` | `Enum {brief, detailed, technical}` | Desired output detail |
| `nlExplanation` | `String` | Human-readable explanation of the result |

---

## Suitable FM Instances

| FM | Deployment Mode | Notes |
|----|-----------------|-------|
| GPT-4o (OpenAI) | Commercial API | Excellent at SPARQL generation |
| Gemini 1.5 Pro | Commercial API | Strong schema-aware query generation |
| Claude Sonnet (Anthropic) | Commercial API | Good instruction-following for NL→formal |
| Llama 3.1-70B | Local OSS | Viable for simpler query generation |
| Phi-3-medium | Local OSS (Edge) | Suitable for low-resource NL→SPARQL |

---

## SysMLv2 Snapshot

```sysml
part gptNLInterface : CognitiveOrgan {
    attribute :>> organType = CognitiveOrganType::NLInterface;
    attribute :>> contract  = EpistemicContract {
        recallTarget        = 0.85;
        precisionFloor      = 0.85;
        warrantsCorrectness = false;
        requiresGrounding   = false;
        domainRef           = "https://coresense.eu/ontologies/general#";
    };

    // Hard constraint: no KB write path
    constraint { not exists(kbWritePort) }
}
```

---

## When to Use

- When human operators need to query the CoreSense KB in natural language
- When integrating with external NL-producing systems (chatbots, voice interfaces)
- When translating between domain-expert language and formal KB vocabulary

## When NOT to Use

- When the query vocabulary is fixed and small (use a template-based approach instead)
- When response time is critical and the query patterns are known (pre-compile SPARQL templates)
