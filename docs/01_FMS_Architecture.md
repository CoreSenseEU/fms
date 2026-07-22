# FMS Architecture — CoreSense Foundation Model Service

> **Scope**: This document specifies the architectural structure of the Foundation Model Service (FMS), which provides typed FM cognitive organs for use inside CoreSense awareness pipelines. It covers conceptual architecture, deployment topology, a catalog of use patterns, SysMLv2 textual models, and ASCII diagrams.

---

## Table of Contents

1. [Motivation and Design Philosophy](#1-motivation-and-design-philosophy)
2. [CoreSense Awareness Pipeline](#2-coresense-awareness-pipeline)
3. [FM as a Typed Cognitive Organ](#3-fm-as-a-typed-cognitive-organ)
4. [FMS Structural Architecture](#4-fms-structural-architecture)
5. [Deployment Topology](#5-deployment-topology)
6. [Catalog of FM Use Patterns](#6-catalog-of-fm-use-patterns)
7. [SysMLv2 Textual Models](#7-sysmlv2-textual-models)
8. [Integration Protocol](#8-integration-protocol)
9. [References](#9-references)

---

## 1. Motivation and Design Philosophy

### 1.1 The Problem with Raw FM Integration

A Foundation Model (FM) such as Claude Opus, GPT-4o, Kimi K3, or Llama 3 is a statistical language engine trained on vast corpora. It is an extraordinarily capable generator of plausible text, code, structured data, and reasoning traces. However, it has fundamental structural limitations that make naive integration into a cognitive architecture dangerous:

| Limitation | Consequence |
|-----------|-------------|
| No grounded world model | Cannot verify commutativity of assertions against physical reality |
| Hallucination by design | Generates plausible-looking but false claims with no intrinsic signal |
| No persistent state | Unaware of what the system already knows or has committed to |
| No logical consistency guarantees | Can contradict itself within a long context window |
| Opacity | Cannot explain *why* a claim is true in terms the system can verify |

### 1.2 The CoreSense Solution: Typed Cognitive Organs

CoreSense treats an FM not as an "intelligent system" but as a **typed cognitive organ**—a specialised processing element with:

- A precisely defined **epistemic role** (what it can legitimately claim to do)
- An explicit **quality contract** (recall, precision, latency, cost bounds)
- A mandatory **quarantine protocol** (outputs are candidates, never direct KB inserts)
- A **provenance discipline** (every output is tagged with model identity, prompt, parameters)

This is analogous to how the visual cortex is a typed organ: it produces high-recall edge and shape proposals that higher cognitive centres validate against the rest of the perceptual scene. The visual cortex does not update long-term memory directly.

### 1.3 Epistemic Asymmetry

```
FM organ  →  high recall, broad domain, low precision, no correctness warrant
Integrator →  low recall, narrow domain, high precision, correctness-maintaining
```

CoreSense supplies the model discipline that the FM structurally lacks.

---

## 2. CoreSense Awareness Pipeline

### 2.1 Pipeline Stages

The CoreSense awareness pipeline processes an incoming observation stream through five ordered stages:

```
                                    World
                                      │
                              ┌───────▼───────┐
                              │     SENSE      │  Raw signal acquisition
                              │  (perception)  │  (sensors, APIs, streams)
                              └───────┬───────┘
                                      │  raw signal
                              ┌───────▼───────┐
                              │    EXTRACT     │  Feature & symbol extraction
                              │  (attention)   │  ◀══ FM ELIGIBLE ══▶
                              └───────┬───────┘
                                      │  feature vector / symbol soup
                              ┌───────▼───────┐
                              │  MODEL-PROPOSE │  Candidate model fragment generation
                              │  (abduction)   │  ◀══ FM ELIGIBLE ══▶
                              └───────┬───────┘
                                      │  candidate assertions (tagged)
                              ┌───────▼───────┐
                              │   INTEGRATE    │  Consistency-maintaining KB update
                              │  (induction)   │  ◀══ FM EXCLUDED  ══▶
                              └───────┬───────┘
                                      │  integrated model state
                              ┌───────▼───────┐
                              │   EVALUATE     │  Model quality assessment
                              │  (deduction)   │  ◀══ FM EXCLUDED  ══▶
                              └───────┬───────┘
                                      │  evaluated hypotheses
                              ┌───────▼───────┐
                              │      ACT       │  Action selection & execution
                              └───────────────┘
```

### 2.2 FM Placement Rationale

FMs are admitted only at **Extract** and **Model-Propose** because:

- These stages produce *candidate* outputs that are subsequently filtered.
- A false positive at this stage is costly but recoverable (it is rejected by the integrator).
- These stages benefit from broad-domain recall, which is the FM's core strength.

FMs are excluded from **Integrate** and **Evaluate** because:

- These stages modify committed knowledge; a false insertion may cascade.
- Logical consistency checking requires a ground-truth oracle, not a corpus-trained model.
- An FM cannot reliably detect its own contradictions against the existing KB.

### 2.3 Candidate Buffer and Quarantine

```
  FM Output
     │
     ▼
┌────────────────────────────────────────────────┐
│  CandidateBuffer                               │
│  ┌──────────────────────────────────────────┐ │
│  │  CandidateAssertion                      │ │
│  │  ├── content: Assertion                  │ │
│  │  ├── provenance: ProvenanceRecord        │ │
│  │  ├── confidence: Float [0,1]             │ │
│  │  └── status: {pending, accepted, rejected}│ │
│  └──────────────────────────────────────────┘ │
└────────────────┬───────────────────────────────┘
                 │
                 ▼
        ConsistencyIntegrator
                 │
        ┌────────┴────────┐
        │                 │
      Accept            Reject
        │
        ▼
   KnowledgeBase
```

---

## 3. FM as a Typed Cognitive Organ

### 3.1 CognitiveOrganType

Each FM deployed in a CoreSense system is assigned a `CognitiveOrganType` value from the following taxonomy:

| Type | Description | Typical FM Example |
|------|-------------|-------------------|
| `AbductiveEngine` | High-recall candidate generator; epistemic goal is coverage | Kimi K3, GPT-4o |
| `FeatureExtractor` | Transforms raw signal to symbolic feature vector | CLIP, Whisper, Phi-3-vision |
| `KnowledgeElicitor` | Extracts structured triples from unstructured text | Claude Opus, Llama 3.1 |
| `NLInterface` | Bridges NL queries to formal SPARQL/Prolog/OWL | GPT-4o, Gemini 1.5 Pro |
| `CodeLogicGenerator` | Generates executable symbolic programs | Claude Sonnet, DeepSeek-Coder |
| `SimilarityOracle` | Provides semantic similarity and analogical retrieval | text-embedding-3-large, BGE |
| `HypothesisGenerator` | Drafts scientific/causal hypotheses | Claude Opus, Kimi K3 |
| `ExplanationNarrator` | Translates formal derivations to NL | Any instruction-tuned LLM |

### 3.2 Epistemic Contract

Every `CognitiveOrgan` instance carries an `EpistemicContract`:

```
EpistemicContract:
  recallTarget   : Float        -- minimum expected recall over domain
  precisionFloor : Float        -- minimum expected precision (can be low for AbductiveEngine)
  domain         : OntologyRef  -- scope of valid operation
  warrantsCorrectness : Boolean -- ALWAYS false for FM-backed organs
  requiresGrounding   : Boolean -- whether outputs must be grounded before KB entry
```

### 3.3 ProvenanceRecord

```
ProvenanceRecord:
  modelId      : String          -- e.g. "claude-opus-4-5", "kimi-k3-8b"
  modelVersion : String
  provider     : ProviderEnum    -- Anthropic | OpenAI | Google | Moonshot | Local
  promptHash   : SHA256
  temperature  : Float
  maxTokens    : Integer
  timestamp    : ISO8601DateTime
  pipelineStage: StageEnum       -- Extract | ModelPropose
  sessionId    : UUID
```

---

## 4. FMS Structural Architecture

### 4.1 Component Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│  CoreSense System                                                   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Awareness Pipeline                                         │   │
│  │                                                             │   │
│  │  ┌──────────┐   ┌──────────────────────────────────────┐   │   │
│  │  │  Sense   │──▶│  Extract Stage                       │   │   │
│  │  └──────────┘   │  ┌────────────────────────────────┐  │   │   │
│  │                 │  │  FMOrganAdapter (Extract)       │  │   │   │
│  │                 │  │  ╔═══════════════════════════╗  │  │   │   │
│  │                 │  │  ║  CognitiveOrgan           ║  │  │   │   │
│  │                 │  │  ║  type: FeatureExtractor   ║  │  │   │   │
│  │                 │  │  ╚═══════════════╤═══════════╝  │  │   │   │
│  │                 │  └──────────────────┼──────────────┘  │   │   │
│  │                 └──────────────────── │ ────────────────┘   │   │
│  │                                       │                      │   │
│  │                 ┌─────────────────────▼────────────────┐    │   │
│  │                 │  Model-Propose Stage                  │    │   │
│  │                 │  ┌────────────────────────────────┐  │    │   │
│  │                 │  │  FMOrganAdapter (Propose)       │  │    │   │
│  │                 │  │  ╔═══════════════════════════╗  │  │    │   │
│  │                 │  │  ║  CognitiveOrgan           ║  │  │    │   │
│  │                 │  │  ║  type: AbductiveEngine    ║  │  │    │   │
│  │                 │  │  ╚═══════════════╤═══════════╝  │  │    │   │
│  │                 │  └──────────────────┼──────────────┘  │    │   │
│  │                 └─────────────────────┼─────────────────┘    │   │
│  │                                       │ CandidateAssertions   │   │
│  │                 ┌─────────────────────▼────────────────┐    │   │
│  │                 │  CandidateBuffer                      │    │   │
│  │                 └─────────────────────┬────────────────┘    │   │
│  │                                       │                      │   │
│  │                 ┌─────────────────────▼────────────────┐    │   │
│  │                 │  ConsistencyIntegrator (FM-free)      │    │   │
│  │                 └─────────────────────┬────────────────┘    │   │
│  │                                       │                      │   │
│  │                 ┌─────────────────────▼────────────────┐    │   │
│  │                 │  KnowledgeBase                        │    │   │
│  │                 └──────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  FMS (Foundation Model Service)                             │   │
│  │                                                             │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │   │
│  │  │  FMRegistry  │  │  FMRouter    │  │ ProvenanceLogger │  │   │
│  │  └──────────────┘  └──────────────┘  └──────────────────┘  │   │
│  │                                                             │   │
│  │  ┌──────────────────────────────────────────────────────┐  │   │
│  │  │  Adapter Layer                                        │  │   │
│  │  │  ┌────────────┐ ┌────────────┐ ┌──────────────────┐  │  │   │
│  │  │  │ LocalOSSA. │ │ APIAdapter │ │ CloudMLAdapter   │  │  │   │
│  │  │  └─────┬──────┘ └─────┬──────┘ └────────┬─────────┘  │  │   │
│  │  └────────┼──────────────┼─────────────────┼────────────┘  │   │
│  └───────────┼──────────────┼─────────────────┼───────────────┘   │
└──────────────┼──────────────┼─────────────────┼───────────────────┘
               │              │                  │
        ┌──────▼──┐    ┌──────▼──────┐   ┌───────▼──────┐
        │  Ollama  │    │  API (HTTPS)│   │  Azure ML /  │
        │  llama.  │    │  Anthropic  │   │  AWS Bedrock │
        │  cpp     │    │  OpenAI     │   │  Vertex AI   │
        └──────────┘    │  Moonshot   │   └──────────────┘
                        └─────────────┘
```

### 4.2 FMS Internal Components

| Component | Responsibility |
|-----------|----------------|
| `FMRegistry` | Maintains the catalog of available FM organs with their EpistemicContracts |
| `FMRouter` | Selects the appropriate FM organ for a given pipeline stage and task |
| `ProvenanceLogger` | Records every FM exertion with full ProvenanceRecord |
| `CandidateBuffer` | Holds FM outputs in quarantine pending integration |
| `LocalOSSAdapter` | Interfaces with Ollama / llama.cpp / LM Studio endpoints |
| `APIAdapter` | Interfaces with commercial API providers (Anthropic, OpenAI, Moonshot, Google) |
| `CloudMLAdapter` | Interfaces with Azure ML, AWS Bedrock, Google Vertex AI managed endpoints |

---

## 5. Deployment Topology

### 5.1 Mode A — Local OSS Deployment

```
┌──────────────────────────────────────────────┐
│  CoreSense System (on-premises / edge)        │
│                                              │
│  ┌──────────────────┐   ┌────────────────┐  │
│  │  Awareness        │   │  FMS           │  │
│  │  Pipeline         │◀──│  LocalOSSAd.   │  │
│  └──────────────────┘   └───────┬────────┘  │
│                                 │            │
│  ┌──────────────────────────────▼─────────┐  │
│  │  Local Model Server (Ollama / llama.cpp)│  │
│  │  Models: Llama 3.1-70B, Mistral-7B,    │  │
│  │          Phi-3-medium, Qwen2-72B,      │  │
│  │          DeepSeek-R1, Gemma-2          │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
```

**Characteristics:**
- Full data residency; no external calls
- Tunable latency via model size selection
- Hardware-constrained: requires GPU with ≥16 GB VRAM for capable models
- No usage cost; infrastructure cost only
- Suitable for: sensitive data, air-gapped deployments, high-frequency Extract calls

### 5.2 Mode B — Commercial API Deployment

```
┌──────────────────────────────────────────────┐
│  CoreSense System                            │
│                                              │
│  ┌──────────────────┐   ┌────────────────┐  │
│  │  Awareness        │   │  FMS           │  │
│  │  Pipeline         │◀──│  APIAdapter    │  │
│  └──────────────────┘   └───────┬────────┘  │
│                                 │ HTTPS      │
└─────────────────────────────────┼────────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
    ┌─────────▼──────┐  ┌────────▼───────┐  ┌────────▼───────┐
    │  Anthropic      │  │  MoonshotAI    │  │  OpenAI        │
    │  Claude Opus    │  │  Kimi K3       │  │  GPT-4o        │
    │  Claude Sonnet  │  │  Kimi K2       │  │  o1-preview    │
    └─────────────────┘  └────────────────┘  └────────────────┘
```

**Characteristics:**
- State-of-the-art capability
- Usage-metered (cost per token)
- Network latency overhead (~200–800 ms)
- Rate-limited: requires backoff and retry logic
- Suitable for: high-value Model-Propose calls, complex KnowledgeElicitor tasks

### 5.3 Mode C — Cloud ML Platform Deployment

```
┌──────────────────────────────────────────────┐
│  CoreSense System                            │
│                                              │
│  ┌──────────────────┐   ┌────────────────┐  │
│  │  Awareness        │   │  FMS           │  │
│  │  Pipeline         │◀──│  CloudMLAd.    │  │
│  └──────────────────┘   └───────┬────────┘  │
│                                 │            │
└─────────────────────────────────┼────────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
    ┌─────────▼──────┐  ┌────────▼───────┐  ┌────────▼───────┐
    │  Azure ML       │  │  AWS Bedrock   │  │  Google        │
    │  Foundation     │  │  Claude /      │  │  Vertex AI     │
    │  Models         │  │  Llama / Titan │  │  Gemini        │
    └─────────────────┘  └────────────────┘  └────────────────┘
```

**Characteristics:**
- Enterprise governance (VNet injection, private endpoints, RBAC)
- Fine-tuning and custom model hosting
- Managed MLOps pipelines (model versioning, A/B routing)
- Compliance certifications (SOC 2, ISO 27001, HIPAA where applicable)
- Suitable for: regulated industries, enterprise CoreSense deployments

### 5.4 Mode D — Hybrid (Draft + Verify)

```
┌─────────────────────────────────────────────────────┐
│  CoreSense System                                   │
│                                                     │
│  ┌────────────────────────────────────────────────┐ │
│  │  FMS Hybrid Router                             │ │
│  │                                               │ │
│  │  Phase 1 — Draft (Local, fast, cheap):        │ │
│  │  ┌────────────────────────────────────────┐   │ │
│  │  │  LocalOSSAdapter → Phi-3-mini or       │   │ │
│  │  │                    Llama-3.1-8B        │   │ │
│  │  └────────────────────────────────────────┘   │ │
│  │                    │ draft candidates          │ │
│  │  Phase 2 — Verify (Cloud, slow, high quality):│ │
│  │  ┌────────────────────────────────────────┐   │ │
│  │  │  APIAdapter → Claude Opus or Kimi K3   │   │ │
│  │  │  (only for low-confidence drafts)      │   │ │
│  │  └────────────────────────────────────────┘   │ │
│  └────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

**Characteristics:**
- 60–80 % cost reduction vs. all-cloud
- Latency optimised for high-confidence drafts
- Requires confidence scoring on draft outputs
- Suitable for: high-volume Extract stage workloads

---

## 6. Catalog of FM Use Patterns

### P1 — Abductive Engine

**Inspiration**: Kimi K3 deployment pattern.

**Epistemic Role**: High-recall proposer of candidate model fragments over a broad domain. Goal is *coverage*, not precision. Acts as an amortized abductive engine that surfaces plausible hypotheses faster than any formal search would allow.

**Pipeline Stage**: `ModelPropose`

**Quality Contract**:
- `recallTarget` ≥ 0.90
- `precisionFloor` ≥ 0.30 (deliberately low)
- `warrantsCorrectness` = false

**Input / Output**:
```
Input:  ObservationSet (feature vectors + context)
Output: List<CandidateModelFragment> with ProvenanceRecord
```

**Excluded From**: `Integrate`, `Evaluate`

**Reference FM instances**: Kimi K3, Claude Opus, GPT-4o, Llama 3.1-70B

---

### P2 — Feature Extractor

**Epistemic Role**: Transforms raw perceptual signals (text, image, audio, sensor data) into symbolic feature vectors consumable by downstream pipeline stages.

**Pipeline Stage**: `Extract`

**Quality Contract**:
- `recallTarget` ≥ 0.95 (miss rate must be very low)
- Latency SLA: ≤ 100 ms for local, ≤ 500 ms for API
- `warrantsCorrectness` = false (features are still candidates)

**Input / Output**:
```
Input:  RawSignal (text | image | audio | sensor)
Output: FeatureVector + ProvenanceRecord
```

**Reference FM instances**: CLIP, Whisper, Phi-3-vision, BLIP-2 (local); GPT-4o Vision (API)

---

### P3 — Knowledge Elicitor

**Epistemic Role**: Extracts structured RDF/OWL triples, entity-relation pairs, or ontology fragments from unstructured text sources. All extracted knowledge enters the `CandidateBuffer` for validation.

**Pipeline Stage**: `Extract` → `ModelPropose`

**Quality Contract**:
- Triple extraction F1 ≥ 0.75 on the target domain
- Output schema: `{subject, predicate, object, confidence, source}`

**Input / Output**:
```
Input:  UnstructuredText (documents, web pages, scientific papers)
Output: List<KGTriple> with ProvenanceRecord
```

**Reference FM instances**: Claude Opus, Llama 3.1-70B (Instruct), Qwen2-72B

---

### P4 — Natural Language Interface

**Epistemic Role**: Bridges natural-language queries from human operators or external systems to formal queries (SPARQL, Prolog, OWL-DL) against the CoreSense Knowledge Base. Also translates formal results back to NL for human consumption.

**Pipeline Stage**: Outside core pipeline — acts as a bidirectional interface

**Quality Contract**:
- NL→Formal translation accuracy ≥ 0.85 on domain vocabulary
- Must not modify KB directly; generates only query strings

**Input / Output**:
```
Input:  NLQuery (string)
Output: FormalQuery (SPARQL | Prolog | OWL-DL) + ProvenanceRecord

Input:  FormalResult (query result set)
Output: NLExplanation (string) + ProvenanceRecord
```

**Reference FM instances**: GPT-4o, Gemini 1.5 Pro, Claude Sonnet (API); Llama 3.1-70B (local)

---

### P5 — Code/Logic Generator

**Epistemic Role**: Generates executable symbolic programs—Prolog rules, Python scripts, SPARQL updates, or OWL axioms—that encode domain knowledge or reasoning procedures. All generated code is validated before execution.

**Pipeline Stage**: `ModelPropose` (generates rule candidates for validation)

**Quality Contract**:
- Generated code must pass static analysis before acceptance
- `warrantsCorrectness` = false; formal verification or test suites are mandatory
- Output must be deterministic given the same prompt (temperature = 0)

**Input / Output**:
```
Input:  Specification (NL description + type signature)
Output: CodeArtifact (source code string + language tag) + ProvenanceRecord
```

**Reference FM instances**: Claude Sonnet 4.5 (code), DeepSeek-Coder-V2, GPT-4o (API); CodeLlama-34B (local)

---

### P6 — Similarity Oracle

**Epistemic Role**: Provides semantic similarity scores, nearest-neighbour retrieval, and analogical reasoning over the CoreSense KB and external corpora via dense vector embeddings.

**Pipeline Stage**: `Extract` (as retrieval-augmented input enrichment)

**Quality Contract**:
- nDCG@10 ≥ 0.80 on retrieval benchmarks relevant to domain
- Embedding dimensionality: ≥ 512 dimensions
- Latency SLA: ≤ 20 ms for cached, ≤ 200 ms for fresh embeddings

**Input / Output**:
```
Input:  Query (string | Assertion)
Output: List<{item: Assertion, similarity: Float}> + ProvenanceRecord
```

**Reference FM instances**: text-embedding-3-large (OpenAI API); BGE-M3, E5-large-v2 (local)

---

### P7 — Hypothesis Generator

**Epistemic Role**: Drafts scientific or causal hypotheses by combining the current KB state with broad FM world knowledge. Hypotheses are candidate assertions; they require causal-model validation.

**Pipeline Stage**: `ModelPropose`

**Quality Contract**:
- `recallTarget` ≥ 0.85 over the hypothesis space
- Must include a structured uncertainty estimate per hypothesis
- `warrantsCorrectness` = false

**Input / Output**:
```
Input:  ObservationSet + KBSnapshot (current committed knowledge)
Output: List<Hypothesis {claim, uncertainty, supporting_citations}> + ProvenanceRecord
```

**Reference FM instances**: Claude Opus, Kimi K3, GPT-4o (API); Llama 3.1-70B (local)

---

### P8 — Explanation Narrator

**Epistemic Role**: Translates formal derivation traces (Prolog proof trees, OWL classification explanations, causal graphs) into natural-language explanations suitable for human operators.

**Pipeline Stage**: Post-`Evaluate` (read-only access to committed KB and derivation traces)

**Quality Contract**:
- Output must be grounded in the provided formal derivation — no FM-hallucinated elaboration
- Faithfulness score ≥ 0.90 (derivation content is preserved)
- Verbosity is adjustable (brief | detailed | technical)

**Input / Output**:
```
Input:  DerivationTrace (proof tree | causal graph | classification justification)
Output: NLExplanation (string, with structured citation anchors) + ProvenanceRecord
```

**Note**: This is the only pattern that operates *after* Evaluate, but only in **read-only** mode. It never writes to the KB.

**Reference FM instances**: Any instruction-tuned LLM (Claude Haiku, GPT-4o-mini, Llama 3.1-8B)

---

## 7. SysMLv2 Textual Models

### 7.1 Core Type Definitions

```sysml
package FMS {

    // ─── Enumerations ───────────────────────────────────────────────────────

    enum def CognitiveOrganType {
        AbductiveEngine;
        FeatureExtractor;
        KnowledgeElicitor;
        NLInterface;
        CodeLogicGenerator;
        SimilarityOracle;
        HypothesisGenerator;
        ExplanationNarrator;
    }

    enum def ProviderType {
        Anthropic;
        OpenAI;
        Google;
        Moonshot;
        Meta;
        Mistral;
        Local;
        AzureML;
        AWSBedrock;
        VertexAI;
    }

    enum def PipelineStage {
        Sense;
        Extract;
        ModelPropose;
        Integrate;
        Evaluate;
        Act;
    }

    enum def CandidateStatus {
        Pending;
        Accepted;
        Rejected;
    }

    // ─── Value Types ────────────────────────────────────────────────────────

    attribute def ProvenanceRecord {
        attribute modelId      : String;
        attribute modelVersion : String;
        attribute provider     : ProviderType;
        attribute promptHash   : String;         // SHA-256 of rendered prompt
        attribute temperature  : Real;
        attribute maxTokens    : Integer;
        attribute timestamp    : String;         // ISO-8601
        attribute pipelineStage: PipelineStage;
        attribute sessionId    : String;         // UUID
    }

    attribute def EpistemicContract {
        attribute recallTarget          : Real;   // [0, 1]
        attribute precisionFloor        : Real;   // [0, 1]
        attribute warrantsCorrectness   : Boolean;
        attribute requiresGrounding     : Boolean;
        attribute domainRef             : String; // ontology IRI
    }

    // ─── Items ──────────────────────────────────────────────────────────────

    item def Assertion {
        attribute subject   : String;
        attribute predicate : String;
        attribute object    : String;
        attribute confidence: Real;
    }

    item def CandidateAssertion {
        attribute content   : Assertion;
        attribute provenance: ProvenanceRecord;
        attribute status    : CandidateStatus default CandidateStatus::Pending;
    }

    item def FeatureVector {
        attribute dimensions: Integer;
        attribute values    : Real[*];
        attribute provenance: ProvenanceRecord;
    }

    // ─── Ports ──────────────────────────────────────────────────────────────

    port def RawSignalIn {
        in item signal : String;
    }

    port def FeatureVectorOut {
        out item features : FeatureVector;
    }

    port def CandidateAssertionOut {
        out item candidate : CandidateAssertion;
    }

    port def FMRequestIn {
        in item prompt   : String;
        in item contract : EpistemicContract;
    }

    port def FMResponseOut {
        out item response   : String;
        out item provenance : ProvenanceRecord;
    }

}
```

### 7.2 Cognitive Organ Part Definitions

```sysml
package FMS::Organs {

    import FMS::*;

    // ─── Abstract Cognitive Organ ────────────────────────────────────────────

    part def CognitiveOrgan {
        attribute organType   : CognitiveOrganType;
        attribute contract    : EpistemicContract;
        attribute isActive    : Boolean default false;

        // Every organ has an FMRequest entry port and a CandidateAssertion exit port
        port fmIn    : FMRequestIn;
        port fmOut   : FMResponseOut;
        port candidateOut : CandidateAssertionOut;

        // Every exertion must produce a provenance record
        action exert {
            in  item prompt    : String;
            out item candidate : CandidateAssertion;
        }
    }

    // ─── Abductive Engine Organ ──────────────────────────────────────────────

    part def AbductiveEngineOrgan :> CognitiveOrgan {
        attribute :>> organType = CognitiveOrganType::AbductiveEngine;
        attribute :>> contract  = EpistemicContract {
            recallTarget        = 0.90;
            precisionFloor      = 0.30;
            warrantsCorrectness = false;
            requiresGrounding   = true;
            domainRef           = "";
        };

        action :>> exert {
            in  item observationSet : String;
            out item candidates     : CandidateAssertion[1..*];
        }
    }

    // ─── Feature Extractor Organ ─────────────────────────────────────────────

    part def FeatureExtractorOrgan :> CognitiveOrgan {
        attribute :>> organType = CognitiveOrganType::FeatureExtractor;
        attribute :>> contract  = EpistemicContract {
            recallTarget        = 0.95;
            precisionFloor      = 0.70;
            warrantsCorrectness = false;
            requiresGrounding   = false;
            domainRef           = "";
        };

        port rawIn : RawSignalIn;
        port vecOut: FeatureVectorOut;

        action :>> exert {
            in  item rawSignal : String;
            out item features  : FeatureVector;
        }
    }

    // ─── Knowledge Elicitor Organ ────────────────────────────────────────────

    part def KnowledgeElicitorOrgan :> CognitiveOrgan {
        attribute :>> organType = CognitiveOrganType::KnowledgeElicitor;
        attribute :>> contract  = EpistemicContract {
            recallTarget        = 0.80;
            precisionFloor      = 0.75;
            warrantsCorrectness = false;
            requiresGrounding   = true;
            domainRef           = "";
        };

        action :>> exert {
            in  item sourceText : String;
            out item triples    : CandidateAssertion[*];
        }
    }

}
```

### 7.3 FM Adapter Definitions

```sysml
package FMS::Adapters {

    import FMS::*;
    import FMS::Organs::*;

    // ─── Abstract FM Adapter ─────────────────────────────────────────────────

    part def FMAdapter {
        attribute provider       : ProviderType;
        attribute endpointUrl    : String;
        attribute authMethod     : String;  // "apikey" | "oauth2" | "none"
        attribute maxConcurrency : Integer;
        attribute timeoutMs      : Integer;

        port adapterIn  : FMRequestIn;
        port adapterOut : FMResponseOut;

        action invoke {
            in  item prompt    : String;
            in  item params    : String;      // JSON-serialised model parameters
            out item rawOutput : String;
            out item prov      : ProvenanceRecord;
        }
    }

    // ─── Local OSS Adapter ───────────────────────────────────────────────────

    part def LocalOSSAdapter :> FMAdapter {
        attribute :>> provider    = ProviderType::Local;
        attribute serverType      : String;  // "ollama" | "llama.cpp" | "lmstudio"
        attribute modelPath       : String;  // local filesystem path
        attribute contextLength   : Integer;
        attribute numGpuLayers    : Integer;

        action :>> invoke {
            in  item prompt      : String;
            in  item params      : String;
            out item rawOutput   : String;
            out item prov        : ProvenanceRecord;
        }
    }

    // ─── Commercial API Adapter ───────────────────────────────────────────────

    part def CommercialAPIAdapter :> FMAdapter {
        attribute modelName     : String;    // e.g. "claude-opus-4-5"
        attribute apiKeyRef     : String;    // reference to secret store key (not value)
        attribute retryPolicy   : String;    // "exponential" | "linear" | "none"
        attribute maxRetries    : Integer;

        action :>> invoke {
            in  item prompt    : String;
            in  item params    : String;
            out item rawOutput : String;
            out item prov      : ProvenanceRecord;
        }
    }

    // ─── Cloud ML Adapter ────────────────────────────────────────────────────

    part def CloudMLAdapter :> FMAdapter {
        attribute :>> provider  = ProviderType::AzureML;
        attribute workspaceName : String;
        attribute deploymentName: String;
        attribute resourceGroup : String;
        attribute subscriptionId: String;

        action :>> invoke {
            in  item prompt    : String;
            in  item params    : String;
            out item rawOutput : String;
            out item prov      : ProvenanceRecord;
        }
    }

}
```

### 7.4 Full FMS System Block

```sysml
package FMS::System {

    import FMS::*;
    import FMS::Organs::*;
    import FMS::Adapters::*;

    // ─── Candidate Buffer ─────────────────────────────────────────────────────

    part def CandidateBuffer {
        part candidates : CandidateAssertion[0..*];

        action enqueue { in item c : CandidateAssertion; }
        action dequeueAll { out item cs : CandidateAssertion[*]; }
        action reject { in item c : CandidateAssertion; }
    }

    // ─── Consistency Integrator (FM-free) ────────────────────────────────────

    part def ConsistencyIntegrator {
        // This component deliberately has no FM adapter.
        // It uses formal consistency checking (OWL-DL reasoning, ASP, Prolog).

        action integrate {
            in  item candidates : CandidateAssertion[*];
            out item accepted   : CandidateAssertion[*];
            out item rejected   : CandidateAssertion[*];
        }
    }

    // ─── FM Registry ─────────────────────────────────────────────────────────

    part def FMRegistry {
        part organs : CognitiveOrgan[0..*];

        action register { in item organ : CognitiveOrgan; }

        action resolve {
            in  item stage    : PipelineStage;
            in  item organType: CognitiveOrganType;
            out item organ    : CognitiveOrgan;
        }
    }

    // ─── Full FMS ─────────────────────────────────────────────────────────────

    part def FoundationModelService {
        part registry         : FMRegistry;
        part candidateBuffer  : CandidateBuffer;
        part integrator       : ConsistencyIntegrator;

        // At least one adapter must be present
        part localAdapter  : LocalOSSAdapter[0..1];
        part apiAdapter    : CommercialAPIAdapter[0..1];
        part cloudAdapter  : CloudMLAdapter[0..1];

        // Constraint: at least one adapter is active
        constraint { localAdapter.isActive or apiAdapter.isActive or cloudAdapter.isActive }

        action pipeline {
            action extract       { /* FM organs eligible */ }
            action modelPropose  { /* FM organs eligible */ }
            action integrate     { /* FM organs EXCLUDED — ConsistencyIntegrator only */ }
            action evaluate      { /* FM organs EXCLUDED */ }

            // Succession
            then modelPropose;
            then integrate;
            then evaluate;
        }
    }

}
```

### 7.5 Kimi K3 Deployment Example (Concrete Instance)

```sysml
package FMS::Examples::KimiK3 {

    import FMS::*;
    import FMS::Organs::*;
    import FMS::Adapters::*;
    import FMS::System::*;

    // Concrete adapter instance pointing to Moonshot API
    part kimiAdapter : CommercialAPIAdapter {
        provider    = ProviderType::Moonshot;
        modelName   = "kimi-k3";
        endpointUrl = "https://api.moonshot.cn/v1/chat/completions";
        apiKeyRef   = "vault://fms/moonshot-api-key";   // reference, not secret
        retryPolicy = "exponential";
        maxRetries  = 3;
        timeoutMs   = 30000;
    }

    // Abductive engine organ wired to the Kimi adapter
    part kimiOrgan : AbductiveEngineOrgan {
        // Override domain for a robotics scenario
        attribute :>> contract = EpistemicContract {
            recallTarget        = 0.92;
            precisionFloor      = 0.28;
            warrantsCorrectness = false;
            requiresGrounding   = true;
            domainRef           = "https://coresense.eu/ontologies/robotics#";
        };
    }

    // Wire organ to adapter
    connection def KimiOrganToAdapter
        connect kimiOrgan.fmOut to kimiAdapter.adapterIn;

    // Instantiate a FMS with Kimi as the primary abductive engine
    part kimiFMS : FoundationModelService {
        :>> apiAdapter = kimiAdapter;
    }

}
```

### 7.6 Local Llama 3 Deployment Example

```sysml
package FMS::Examples::LocalLlama3 {

    import FMS::*;
    import FMS::Organs::*;
    import FMS::Adapters::*;
    import FMS::System::*;

    // Local Ollama endpoint
    part llamaAdapter : LocalOSSAdapter {
        serverType    = "ollama";
        endpointUrl   = "http://localhost:11434/api/generate";
        modelPath     = "llama3.1:70b";
        authMethod    = "none";
        contextLength = 131072;
        numGpuLayers  = 80;
        timeoutMs     = 60000;
    }

    // Feature extractor organ for text extraction
    part llamaFeatureOrgan : FeatureExtractorOrgan {
        attribute :>> contract = EpistemicContract {
            recallTarget        = 0.95;
            precisionFloor      = 0.72;
            warrantsCorrectness = false;
            requiresGrounding   = false;
            domainRef           = "https://coresense.eu/ontologies/general#";
        };
    }

    // Knowledge elicitor organ for structured extraction
    part llamaKEOrgan : KnowledgeElicitorOrgan {
        attribute :>> contract = EpistemicContract {
            recallTarget        = 0.82;
            precisionFloor      = 0.76;
            warrantsCorrectness = false;
            requiresGrounding   = true;
            domainRef           = "https://coresense.eu/ontologies/general#";
        };
    }

    // Wire both organs to the same adapter
    connection connect llamaFeatureOrgan.fmOut to llamaAdapter.adapterIn;
    connection connect llamaKEOrgan.fmOut      to llamaAdapter.adapterIn;

    // Full FMS instance (local-only deployment)
    part localFMS : FoundationModelService {
        :>> localAdapter = llamaAdapter;
    }

}
```

---

## 8. Integration Protocol

### 8.1 Sequence: AbductiveEngine Exertion

```
CoreSense                   FMS                    Kimi K3 / FM
Pipeline                    (FMRouter)              API / Local
   │                           │                        │
   │  invokeOrgan(             │                        │
   │    stage=ModelPropose,    │                        │
   │    organType=Abductive,   │                        │
   │    observations=O)        │                        │
   │──────────────────────────▶│                        │
   │                           │ buildPrompt(O)         │
   │                           │───────────────────────▶│
   │                           │                        │ generate()
   │                           │     rawResponse        │
   │                           │◀───────────────────────│
   │                           │ parseToAssertions()    │
   │                           │ tagProvenance()        │
   │                           │ enqueueToBuffer()      │
   │◀──────────────────────────│                        │
   │  List<CandidateAssertion> │                        │
   │                           │                        │
   │  integrate(candidates)    │                        │
   │──────────────────────────▶ConsistencyIntegrator    │
   │                           │ (FM-free)              │
   │  List<accepted/rejected>  │                        │
   │◀──────────────────────────│                        │
   │                           │                        │
   │  KB.insert(accepted)      │                        │
   │                           │                        │
```

### 8.2 Mandatory Checks Before KB Insertion

Before any `CandidateAssertion` transitions from `Pending` to `Accepted`:

1. **Schema validation** — assertion matches the KB ontology schema
2. **Consistency check** — no contradiction with committed assertions (OWL-DL or ASP)
3. **Provenance completeness** — `ProvenanceRecord` has no null fields
4. **Confidence threshold** — `confidence ≥ configuredThreshold` (default 0.5)
5. **Blacklist check** — predicate is not in the `ExcludedPredicates` set for FM organs

### 8.3 Error Handling

| Error Type | FMS Response |
|-----------|-------------|
| API timeout | Retry with exponential backoff; fallback to local model if available |
| FM hallucination detected | CandidateAssertion.status = Rejected; log to ProvenanceLogger |
| Schema violation | CandidateAssertion.status = Rejected; emit warning |
| Consistency conflict | CandidateAssertion.status = Rejected; emit ConflictRecord |
| Rate limit exceeded | Queue request; emit backpressure signal to pipeline |
| Local model OOM | Fallback to smaller model; log degraded-mode warning |

---

## 9. References

1. CoreSense EU Project: <https://github.com/CoreSenseEU>
2. MoonshotAI Kimi K3: <https://www.moonshot.cn>
3. Azure ML Foundation Models: <https://learn.microsoft.com/en-us/azure/machine-learning/how-to-use-foundation-models?view=azureml-api-2>
4. SysMLv2 Specification (OMG): <https://www.omg.org/spec/SysML/2.0>
5. Ollama local model server: <https://ollama.com>
6. Anthropic Claude API: <https://docs.anthropic.com>
7. OpenAI API: <https://platform.openai.com/docs>
8. AWS Bedrock: <https://docs.aws.amazon.com/bedrock>
9. Google Vertex AI: <https://cloud.google.com/vertex-ai/docs>
10. BGE-M3 embedding model: <https://huggingface.co/BAAI/bge-m3>
11. DeepSeek-Coder: <https://github.com/deepseek-ai/DeepSeek-Coder>
