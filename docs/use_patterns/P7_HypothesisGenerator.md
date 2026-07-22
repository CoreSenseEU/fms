# P7 — Hypothesis Generator

> **Pattern**: Hypothesis Generator  
> **Epistemic Role**: Draft scientific or causal hypotheses  
> **Pipeline Stage**: `ModelPropose`  
> **FM Eligible**: Yes  
> **FM Excluded From**: `Integrate`, `Evaluate`

---

## Description

A Foundation Model deployed as a **Hypothesis Generator** drafts scientific or causal hypotheses by combining the current KB state with the broad world knowledge encoded in its parameters. It is conceptually similar to P1 (Abductive Engine) but is scoped specifically to scientific and causal reasoning contexts.

Where the Abductive Engine proposes general model fragments, the Hypothesis Generator targets explanatory structures: "Why did X happen?", "What mechanism connects A and B?", "What prediction follows if C is true?"

Each hypothesis is a candidate assertion annotated with:
- An uncertainty estimate
- Supporting citations (from FM training knowledge or provided context)
- The reasoning chain that led to the hypothesis

---

## Epistemic Contract

```
recallTarget          : ≥ 0.85 over the reachable hypothesis space
warrantsCorrectness   : false
requiresGrounding     : true
uncertaintyEstimate   : required (per hypothesis)
citationRequired      : true (supporting evidence for each hypothesis)
```

---

## Input / Output

| Slot | Type | Description |
|------|------|-------------|
| `observationSet` | `ObservationSet` | Current observations triggering hypothesis generation |
| `kbSnapshot` | `KBSnapshot` | Committed knowledge the FM may use as context |
| `hypotheses` | `List<Hypothesis>` | Candidate hypotheses with uncertainty and citations |

### Hypothesis Structure

```
Hypothesis:
  claim              : Assertion           -- the proposed causal/explanatory claim
  uncertainty        : Float [0, 1]        -- 0 = certain, 1 = speculative
  supportingCitations: List<String>        -- source references
  reasoningChain     : String              -- CoT trace if available
  provenance         : ProvenanceRecord
```

---

## Suitable FM Instances

| FM | Deployment Mode | Notes |
|----|-----------------|-------|
| Claude Opus (Anthropic) | Commercial API | Strong scientific reasoning, faithful citations |
| Kimi K3 (MoonshotAI) | Commercial API | Broad scientific domain, long context |
| GPT-4o (OpenAI) | Commercial API | Good causal reasoning |
| DeepSeek-R1 | Local OSS (Ollama) | Strong local reasoning via chain-of-thought |
| Llama 3.1-70B | Local OSS (Ollama) | Good for domain-constrained hypothesis generation |

---

## SysMLv2 Snapshot

```sysml
part kimiHypGen : CognitiveOrgan {
    attribute :>> organType = CognitiveOrganType::HypothesisGenerator;
    attribute :>> contract  = EpistemicContract {
        recallTarget        = 0.87;
        precisionFloor      = 0.40;
        warrantsCorrectness = false;
        requiresGrounding   = true;
        domainRef           = "https://coresense.eu/ontologies/science#";
    };
}
```

---

## Downstream Validation

Hypotheses from this pattern must pass causal-model validation before entering the KB:

1. **Causal consistency** — the proposed causal mechanism must not contradict committed causal structure
2. **Empirical anchoring** — at least one KB-committed observation must be explainable by the hypothesis
3. **Parsimony check** — the hypothesis should not be dominated by a simpler committed explanation
4. **Falsifiability check** — the hypothesis must generate testable predictions

---

## When to Use

- In scientific discovery and anomaly-explanation workflows
- When the system observes a novel pattern with no committed causal explanation
- When generating research hypotheses to guide active experimentation

## When NOT to Use

- When a complete causal model already covers the observation (query the model instead)
- When the domain has insufficient training representation in available FMs
