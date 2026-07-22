# P5 — Code / Logic Generator

> **Pattern**: Code / Logic Generator  
> **Epistemic Role**: Produce executable symbolic programs  
> **Pipeline Stage**: `ModelPropose`  
> **FM Eligible**: Yes  
> **FM Excluded From**: `Integrate`, `Evaluate`, and direct execution without validation

---

## Description

A Foundation Model deployed as a **Code / Logic Generator** produces executable symbolic programs: Prolog rules, Answer Set Programs (ASP), SPARQL update queries, Python scripts, or OWL axiom sets. These programs encode domain knowledge or reasoning procedures that can extend or query the CoreSense KB.

Generated code is treated as a candidate artefact—it enters a validation pipeline (static analysis, test suite, or formal verification) before execution. An FM-generated Prolog rule that passes validation may be accepted into the CoreSense rule base; one that fails is rejected and logged.

This pattern is particularly valuable for encoding expert knowledge that is too costly to formalise manually but can be expressed as executable logic.

---

## Epistemic Contract

```
warrantsCorrectness         : false
requiresGrounding           : true (formal validation mandatory)
temperature                 : 0    (must be deterministic)
mandatoryValidation         : true (static analysis + correctness tests)
outputLanguages             : [Prolog, ASP, SPARQL, Python, OWL/Manchester]
```

---

## Input / Output

| Slot | Type | Description |
|------|------|-------------|
| `specification` | `String` | NL description + type signature / schema |
| `targetLanguage` | `CodeLanguage` | Target programming / logic language |
| `code` | `CodeArtifact` | Generated source code string + language tag |
| `validationStatus` | `ValidationResult` | pass / fail / partial with error details |

---

## Validation Pipeline

```
FM Output (code string)
        │
        ▼
  ┌─────────────┐
  │ Static Lint  │  (e.g., SWI-Prolog syntax check, Python ast.parse)
  └──────┬──────┘
         │ pass
         ▼
  ┌─────────────┐
  │ Type / Schema│  (check against target ontology schema)
  └──────┬──────┘
         │ pass
         ▼
  ┌─────────────┐
  │  Test Suite  │  (unit tests on representative examples)
  └──────┬──────┘
         │ pass
         ▼
  ┌─────────────────┐
  │ ConsistencyCheck│  (no contradiction with existing KB)
  └──────┬──────────┘
         │ pass
         ▼
    Accept to Rule Base
```

---

## Suitable FM Instances

| FM | Deployment Mode | Notes |
|----|-----------------|-------|
| Claude Sonnet 4.5 | Commercial API | Excellent code quality and instruction following |
| Claude Opus (Anthropic) | Commercial API | Best for complex logic / formal proofs |
| DeepSeek-Coder-V2 | Local OSS (Ollama) | Strong code generation locally |
| GPT-4o (OpenAI) | Commercial API | Good at SPARQL and Python |
| CodeLlama-34B | Local OSS (Ollama) | Local code generation |
| Qwen2.5-Coder-32B | Local OSS (Ollama) | Strong local alternative |

---

## SysMLv2 Snapshot

```sysml
part claudeCodeOrgan : CognitiveOrgan {
    attribute :>> organType = CognitiveOrganType::CodeLogicGenerator;
    attribute :>> contract  = EpistemicContract {
        recallTarget        = 0.80;
        precisionFloor      = 0.70;
        warrantsCorrectness = false;
        requiresGrounding   = true;
        domainRef           = "https://coresense.eu/ontologies/rules#";
    };
}
```

---

## When to Use

- When domain experts can describe rules in natural language but not in formal logic
- When extending the CoreSense rule base with FM-assisted knowledge encoding
- When generating SPARQL queries or OWL axioms that are too complex to write by hand

## When NOT to Use

- When the code will execute in a security-sensitive context without sandboxing
- When no validation pipeline is in place (never accept FM-generated code without validation)
- When the logic language is non-standard and the FM lacks training data for it
