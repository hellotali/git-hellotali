# AI Mental Health & Dementia Care Definitions

This document defines the AI components, implementation options, and guiding principles for hellotali's Mood & Cognitive Monitoring features.

---

## 1. Core AI Features

### 1.1 NLP Mood Analysis (Text/Voice Check-ins)

Analyze user check-in responses to detect mood, sentiment, and emotional state.

| Option | Provider/Tool | Notes |
|--------|---------------|-------|
| **Managed APIs** | Azure Text Analytics (incl. health) | Enterprise-ready, HIPAA-eligible |
| | OpenAI GPT-4o-mini | Semantic analysis, flexible prompts |
| | Google Cloud Natural Language | Good sentiment/entity detection |
| **Open-source (MIT-friendly)** | spaCy | Fast, customizable NLP pipeline |
| | VADER (NLTK) | Rule-based sentiment, no training needed |
| | ONNX Runtime + DistilBERT/RoBERTa | Emotion classification model |
| **On-device** | ONNX or TensorFlow Lite | Private inference, offline capable |

### 1.2 Cognitive Change Detection (Longitudinal)

Track patterns over time to detect subtle cognitive changes.

**Signals to monitor:**
- Response latency trends
- Word variety / vocabulary richness
- Missed prompts or incomplete responses
- Confusion indicators (repetition, off-topic responses)

**Implementation path:**
1. Train supervised model in Python on response patterns
2. Export to ONNX format
3. Serve in .NET via ONNX Runtime (or TFLite on mobile)

### 1.3 Digital Phenotyping (Optional, Opt-in)

Low-cost behavioral signals that can supplement mood scores.

| Signal | Source | Privacy Notes |
|--------|--------|---------------|
| Step count / sleep | HealthKit / Google Fit | Requires explicit consent |
| Screen-on rhythm | Device APIs | Aggregated only, no content |
| Basic mobility ("home vs out") | Location services | Coarse location only |

**Analysis approach:**
- Combine with mood scores using trend/anomaly detection
- Methods: z-scores, EWMA (Exponentially Weighted Moving Average)

---

## 2. Analytics & Decisioning

### 2.1 Trend & Anomaly Detection

| Technique | Purpose |
|-----------|---------|
| Personal baselines | Establish "normal" for each user |
| Moving averages | Smooth daily fluctuations |
| Z-scores | Detect significant deviations |
| Multi-signal agreement | "Nudge only when multiple signals agree" |

### 2.2 Confidence Scoring & Guardrails

- Require **high confidence** for any alert or nudge
- **Human-in-the-loop** for high-stakes situations
- Graceful degradation when confidence is low

---

## 3. Privacy & Safety Principles

| Principle | Implementation |
|-----------|----------------|
| **Privacy-first** | Store scores/aggregates; avoid raw journals/voice by default |
| **Consent & transparency** | Granular opt-ins per data source |
| **Non-diagnostic** | Supportive nudges only; no medical claims |
| **Crisis routing** | Copy routes to local helplines; not a crisis tool |
| **Data minimization** | Pseudonymous userId; PII minimization |

---

## 4. Implementation Stack (hellotali MVP)

| Layer | Technology | Version | Notes |
|-------|------------|---------|-------|
| **Frontend** | Angular + Material | 21 (LTS) | PWA, standalone components |
| **Mobile (optional)** | Capacitor | Latest | iOS/Android wrapper |
| **Backend** | .NET Web API + EF Core | 10 (LTS) | Minimal APIs, vertical slices |
| **AI Runtime (Cloud)** | OpenAI / Azure / Google APIs | — | Default: GPT-4o-mini |
| **AI Runtime (Local)** | ONNX Runtime in .NET | — | Fallback, offline capable |
| **AI Runtime (Mobile)** | TensorFlow Lite | — | On-device inference |
| **Database** | PostgreSQL | 16+ | All environments |
| **Future (optional)** | Cosmos / Synapse | — | Cohort/longitudinal research |

---

## 5. Open-Source Models to Consider

| Model | Use Case | License | Notes |
|-------|----------|---------|-------|
| DistilBERT (emotion fine-tuned) | Sentiment/emotion classification | Apache 2.0 | Export to ONNX |
| RoBERTa (emotion fine-tuned) | Multi-emotion detection | MIT/Apache | Hugging Face available |
| EmotionBERT variants | Multi-emotion classification | Check license | Research-backed |
| VADER | Rule-based sentiment | MIT | No training needed |

---

## 6. AI Provider Strategy (hellotali)

```
Feature Handler
      ↓
  IAiAnalyzer (interface)
      ↓
┌─────────────┬─────────────┬─────────────┐
│  OpenAI     │   Azure     │   Local     │
│  Analyzer   │  Analyzer   │  Fallback   │
└─────────────┴─────────────┴─────────────┘
```

**Switching via `AI_PROVIDER` environment variable:**
- `openai` (default) → OpenAiAnalyzer (GPT-4o-mini)
- `azure` → AzureTextAnalyzer
- `local` → LocalRuleBasedAnalyzer (zero cost, offline)

---

## References

- [AI Provider Options – hellotali](../docs/AI-provider-options–hellotali.md)
- [Architecture](../docs/architecture.md)
- [App Plan](../docs/app-plan.md)
