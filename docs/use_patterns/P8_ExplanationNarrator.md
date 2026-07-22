# P8 — Explanation Narrator

> **Pattern**: Explanation Narrator  
> **Epistemic Role**: Translate formal derivations to natural-language explanations  
> **Pipeline Stage**: Post-`Evaluate` — read-only access only  
> **FM Eligible**: Yes (read-only)  
> **FM Excluded From**: All KB writes; `Integrate`; `Evaluate`

---

## Description

A Foundation Model deployed as an **Explanation Narrator** translates formal derivation traces—Prolog proof trees, OWL-DL classification justifications, ASP stable-model witnesses, or causal graphs—into natural-language explanations that human operators can read and verify.

This is the only pattern that operates *after* `Evaluate`, but strictly in **read-only** mode. The FM has access to committed KB assertions and derivation traces, but no path to modify them. Its output (the NL explanation) is a human-facing artefact, not a KB write.

The explanation is grounded in the provided formal derivation: the FM must not add information from its own parametric knowledge, as this would undermine the auditability guarantee.

---

## Epistemic Contract

```
warrantsCorrectness         : false (NL is an approximation)
requiresGrounding           : false (derivation is already grounded)
mustBeGroundedInDerivation  : true  (no parametric elaboration)
faithfulnessScore           : ≥ 0.90
kbWritePermission           : false (hard constraint)
verbosityOptions            : [brief, detailed, technical]
```

---

## Input / Output

| Slot | Type | Description |
|------|------|-------------|
| `derivationTrace` | `DerivationTrace` | Formal proof tree / causal graph / justification |
| `verbosity` | `VerbosityEnum` | brief / detailed / technical |
| `audience` | `String` | e.g., "domain expert", "end user", "developer" |
| `explanation` | `NLExplanation` | Human-readable narrative with citation anchors |

### NLExplanation Structure

```
NLExplanation:
  text           : String                     -- the natural-language narrative
  citationAnchors: List<{nodeId, span}>       -- back-references to derivation nodes
  verbosity      : VerbosityEnum
  provenance     : ProvenanceRecord
```

---

## Suitable FM Instances

Any instruction-tuned LLM is suitable because the task is translation, not generation:

| FM | Deployment Mode | Notes |
|----|-----------------|-------|
| Claude Haiku (Anthropic) | Commercial API | Fast, low cost, very good at structured→NL |
| GPT-4o-mini (OpenAI) | Commercial API | Cost-efficient, good quality |
| Llama 3.1-8B | Local OSS (Ollama) | Very fast local narration |
| Phi-3-mini | Local OSS (Ollama/Edge) | Runs on CPU, suitable for edge deployment |
| Gemma-2-9B | Local OSS (Ollama) | Good balance quality / resource |

---

## SysMLv2 Snapshot

```sysml
part haikuNarrator : CognitiveOrgan {
    attribute :>> organType = CognitiveOrganType::ExplanationNarrator;
    attribute :>> contract  = EpistemicContract {
        recallTarget        = 0.99;   // must not omit derivation steps
        precisionFloor      = 0.90;
        warrantsCorrectness = false;
        requiresGrounding   = false;
        domainRef           = "https://coresense.eu/ontologies/general#";
    };

    // Hard constraint: no write port to KB
    constraint { not exists(kbWritePort) }
}
```

---

## Faithfulness Enforcement

To ensure the FM does not add parametric elaboration:

1. **Grounded prompting** — the system prompt explicitly forbids adding facts not present in the derivation trace
2. **Citation anchoring** — every sentence must reference a specific node in the provided trace
3. **Post-hoc faithfulness check** — an automated check verifies that cited nodes exist and match the narrative claim
4. **Verbosity control** — `brief` mode reduces the risk of elaboration by constraining output length

---

## When to Use

- When human operators need to understand why the CoreSense system reached a conclusion
- When audit trails must be presented in plain language to non-technical stakeholders
- When translating OWL-DL explanations for domain experts unfamiliar with formal logic

## When NOT to Use

- When the derivation trace is not available (explanation would be baseless)
- When formal correctness of the explanation matters more than readability (provide the trace directly)
