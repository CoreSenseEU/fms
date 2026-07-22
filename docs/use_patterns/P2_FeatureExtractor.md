# P2 — Feature Extractor

> **Pattern**: Feature Extractor  
> **Epistemic Role**: Raw signal → symbolic feature vector  
> **Pipeline Stage**: `Extract`  
> **FM Eligible**: Yes  
> **FM Excluded From**: `Integrate`, `Evaluate`

---

## Description

A Foundation Model deployed as a **Feature Extractor** transforms raw perceptual signals—natural language text, images, audio recordings, or structured sensor streams—into symbolic feature vectors or labelled symbol sets that downstream pipeline stages can consume.

This is often the first FM-eligible stage in the awareness pipeline. The FM acts as a universal transducer: it applies learned perceptual representations to surface structure that would otherwise require hand-crafted feature engineering per modality.

Feature vectors are still candidate outputs—they carry a `ProvenanceRecord` and are treated as high-confidence but not infallible.

---

## Epistemic Contract

```
recallTarget          : ≥ 0.95   (must not miss salient features)
precisionFloor        : ≥ 0.70
warrantsCorrectness   : false
requiresGrounding     : false    (features feed into downstream grounding)
latencySLA_ms         : ≤ 100    (local) / ≤ 500 (API)
```

---

## Input / Output

| Slot | Type | Description |
|------|------|-------------|
| `rawSignal` | `RawSignal` | Text, image bytes, audio, or sensor stream |
| `features` | `FeatureVector` | Dense or sparse symbolic representation |

---

## Suitable FM Instances

| FM | Modality | Deployment Mode |
|----|----------|-----------------|
| CLIP (OpenAI) | Image + text | Local (Ollama / HuggingFace) |
| Phi-3-vision (Microsoft) | Image + text | Local |
| BLIP-2 | Image | Local |
| Whisper (OpenAI) | Audio → text | Local |
| GPT-4o Vision | Image + text | Commercial API |
| Phi-3-mini | Text | Local (edge-capable) |

---

## SysMLv2 Snapshot

```sysml
part clipExtractor : FeatureExtractorOrgan {
    attribute :>> contract = EpistemicContract {
        recallTarget        = 0.95;
        precisionFloor      = 0.72;
        warrantsCorrectness = false;
        requiresGrounding   = false;
        domainRef           = "https://coresense.eu/ontologies/vision#";
    };
}
```

---

## When to Use

- When the system receives multi-modal signals that must be unified into a common symbolic representation
- When hand-crafted feature engineering would be brittle or domain-limited
- For high-frequency, low-latency extraction (use local models to avoid API overhead)

## When NOT to Use

- When raw signals are already symbolic (no FM needed; use a parser directly)
- When feature extraction correctness must be formally verifiable
