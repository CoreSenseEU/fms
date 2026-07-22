# FMS — Foundation Model Service for CoreSense

A reference implementation of a **CoreSense cognitive structure** for deploying AI Foundation Models (FM)—both open-source models for local deployments and closed systems accessed through commercial APIs—as typed cognitive organs inside a CoreSense awareness pipeline.

---

## Overview

Modern AI Foundation Models (LLMs, VLMs, and multimodal FMs) are powerful but structurally incomplete as autonomous reasoners: they lack grounded consistency guarantees, cannot self-verify commutativity against the world, and are prone to hallucination.  
CoreSense treats an FM not as an "intelligent system" but as a **typed cognitive organ**—a specialised component placed at well-defined stages of the awareness pipeline, with its outputs entering the knowledge base only through consistency-maintaining integration.

This repository provides:

| Artefact | Purpose |
|---|---|
| [`README.md`](./README.md) | Top-level orientation and catalog index |
| [`docs/FMS_Architecture.md`](./docs/FMS_Architecture.md) | Full architecture description, SysMLv2 models and diagrams |
| [`docs/catalog/`](./docs/catalog/) | One document per FM use pattern |

---

## The CoreSense Awareness Pipeline

```
  ┌─────────┐    ┌─────────┐    ┌───────────────┐    ┌───────────┐    ┌──────────┐
  │  Sense  │───▶│ Extract │───▶│ Model-Propose │───▶│ Integrate │───▶│ Evaluate │
  └─────────┘    └────┬────┘    └──────┬────────┘    └─────┬─────┘    └────┬─────┘
                      │                │                    │               │
              ◀─ FM eligible ─▶        ◀─ FM eligible ─▶   ◀── FM excluded ──▶
```

Foundation Models are admitted **only** at the `Extract` and `Model-Propose` stages.  
They are deliberately **excluded** from `Integrate` and `Evaluate`, because a corpus-grounded model cannot check its own commutativity against the world.  
Every FM exertion yields a **provenance-tagged candidate assertion** plus a process record, entering the knowledge base only through the consistency-maintaining integrator.

---

## Catalog of FM Use Patterns

Eight canonical patterns are defined, spanning both deployment mode and epistemic role:

| # | Pattern | Epistemic Role | Deployment | Document |
|---|---------|----------------|------------|----------|
| 1 | **Abductive Engine** | High-recall candidate model fragment proposer | API / Local | [P1](./docs/catalog/P1_AbductiveEngine.md) |
| 2 | **Feature Extractor** | Signal → symbolic feature vector | Local / Edge | [P2](./docs/catalog/P2_FeatureExtractor.md) |
| 3 | **Knowledge Elicitor** | Unstructured text → structured KB fragments | API | [P3](./docs/catalog/P3_KnowledgeElicitor.md) |
| 4 | **NL Interface** | Natural-language ↔ formal query bridge | API / Local | [P4](./docs/catalog/P4_NLInterface.md) |
| 5 | **Code/Logic Generator** | Produce executable symbolic programs | API | [P5](./docs/catalog/P5_CodeLogicGenerator.md) |
| 6 | **Similarity Oracle** | Semantic similarity and analogical retrieval | Local | [P6](./docs/catalog/P6_SimilarityOracle.md) |
| 7 | **Hypothesis Generator** | Scientific/causal hypothesis draft | API | [P7](./docs/catalog/P7_HypothesisGenerator.md) |
| 8 | **Explanation Narrator** | Translate formal derivations into NL explanations | API / Local | [P8](./docs/catalog/P8_ExplanationNarrator.md) |

---

## Supported FM Deployment Modes

| Mode | Examples | Key Characteristic |
|------|----------|--------------------|
| **Local OSS** | Llama 3, Mistral, Phi-3, Qwen 2, DeepSeek | Full data residency, tunable latency |
| **Commercial API** | Claude (Anthropic), GPT-4o (OpenAI), Gemini (Google), Kimi K3 (Moonshot) | State-of-the-art capability, usage-metered |
| **Cloud ML Platform** | Azure ML, AWS Bedrock, Google Vertex AI | Enterprise governance, managed endpoints |
| **Hybrid (draft+verify)** | Local draft → Cloud verify | Cost-latency optimisation |

---

## Key Design Principles

1. **Type discipline** — every FM is assigned a `CognitiveOrganType` with an explicit epistemic contract.
2. **Provenance** — every FM output carries a `ProvenanceRecord` (model id, version, temperature, prompt hash, timestamp).
3. **Quarantine** — FM outputs enter a `CandidateBuffer`; only after passing the `ConsistencyIntegrator` do they enter the `KnowledgeBase`.
4. **Pipeline placement** — FM organs are wired via `FMPort` connectors only to `Extract` and `ModelPropose` pipeline blocks.
5. **Observability** — each exertion emits a `ProcessRecord` for audit.

---

## Repository Structure

```
fms/
├── README.md                          ← this file
├── docs/
│   ├── FMS_Architecture.md            ← full architecture + SysMLv2 + diagrams
│   └── catalog/
│       ├── P1_AbductiveEngine.md
│       ├── P2_FeatureExtractor.md
│       ├── P3_KnowledgeElicitor.md
│       ├── P4_NLInterface.md
│       ├── P5_CodeLogicGenerator.md
│       ├── P6_SimilarityOracle.md
│       ├── P7_HypothesisGenerator.md
│       └── P8_ExplanationNarrator.md
└── LICENSE
```

---

## References

- CoreSense project: <https://github.com/CoreSenseEU>
- Kimi K3 (MoonshotAI): <https://www.moonshot.cn>
- Azure ML Foundation Models: <https://learn.microsoft.com/en-us/azure/machine-learning/how-to-use-foundation-models?view=azureml-api-2>
- SysMLv2 specification: <https://www.omg.org/spec/SysML/2.0>
- Ollama local model serving: <https://ollama.com>
