# P1 — Abductive Engine

> **Pattern**: Abductive Engine  
> **Epistemic Role**: High-recall candidate model fragment proposer  
> **Pipeline Stage**: `ModelPropose`  
> **FM Eligible**: Yes  
> **FM Excluded From**: `Integrate`, `Evaluate`

---

## Description

A Foundation Model deployed as an **Abductive Engine** is the most important CoreSense FM pattern. It is inspired directly by the deployment philosophy behind Kimi K3 and similar broad-domain reasoning FMs.

The FM is encapsulated **not** as an "intelligent system" but as a typed cognitive organ—an *amortized abductive engine* that proposes candidate model fragments at high recall over a broad domain, with no warrant of correctness. It is placed at the `ModelPropose` stage of the awareness pipeline and deliberately excluded from `Integrate` and `Evaluate`, since a corpus-grounded model cannot check its own commutativity against the world.

Every exertion yields a **provenance-tagged candidate assertion** plus a process record, entering the knowledge base only through consistency-maintaining integration. CoreSense supplies the model discipline the engine structurally lacks.

---

## Epistemic Contract

```
recallTarget          : ≥ 0.90
precisionFloor        : ≥ 0.30   (deliberately low — coverage is the goal)
warrantsCorrectness   : false
requiresGrounding     : true
```

The low precision floor is intentional. The Abductive Engine is a recall-maximising device; it is the job of the `ConsistencyIntegrator` to filter out false proposals.

---

## Input / Output

| Slot | Type | Description |
|------|------|-------------|
| `observationSet` | `ObservationSet` | Current features + contextual KB snapshot |
| `candidates` | `CandidateAssertion[1..*]` | Proposed model fragments, each tagged with provenance |

---

## Suitable FM Instances

| FM | Deployment Mode | Notes |
|----|-----------------|-------|
| Kimi K3 (MoonshotAI) | Commercial API | Broad domain, strong abductive reasoning |
| Claude Opus (Anthropic) | Commercial API | High recall, nuanced hypotheses |
| GPT-4o (OpenAI) | Commercial API | Strong general abduction |
| Llama 3.1-70B | Local OSS (Ollama) | Good cost-free alternative for medium domains |
| DeepSeek-R1 | Local OSS (Ollama) | Strong chain-of-thought reasoning locally |

---

## SysMLv2 Snapshot

```sysml
part kimiAbductiveOrgan : AbductiveEngineOrgan {
    attribute :>> contract = EpistemicContract {
        recallTarget        = 0.92;
        precisionFloor      = 0.28;
        warrantsCorrectness = false;
        requiresGrounding   = true;
        domainRef           = "https://coresense.eu/ontologies/robotics#";
    };
}
```

---

## Interaction Sequence

```
Pipeline        FMOrganAdapter      Kimi K3 API
   │                   │                 │
   │  propose(O)       │                 │
   │──────────────────▶│                 │
   │                   │ buildPrompt(O)  │
   │                   │────────────────▶│
   │                   │ rawText         │
   │                   │◀────────────────│
   │                   │ parse + tag     │
   │◀──────────────────│                 │
   │  CandidateAssertion[*]              │
```

---

## When to Use

- When the observation space is large and the domain is broad
- When the cost of a missed true hypothesis (false negative) greatly exceeds the cost of a spurious proposal (false positive)
- When formal search over the hypothesis space would be computationally prohibitive
- When working in early-stage exploratory reasoning with incomplete world models

## When NOT to Use

- When correctness is required immediately (use a formal reasoner instead)
- When the domain is fully formalised and closed-world (use ASP or Prolog directly)
- When the FM's broad-domain training creates domain confusion risks
