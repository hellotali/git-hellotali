**Proposal and Learning Roadmap: AI-Assisted Mood and Cognitive Monitoring**

---

## 1. Objective

To build an AI-powered **Mood and Cognitive Monitoring** feature as the first step in a mobile or wearable companion app supporting **mental health** and **dementia care**. The feature will detect early cognitive decline and emotional distress using AI-driven analysis of user interactions.

The App name is proposed to be hellotali

---

## 2. Target Users

- Individuals at risk of dementia or cognitive decline
- Patients with mental health conditions (e.g., depression, anxiety)
- Family caregivers
- Clinical researchers and healthcare providers

---

## 3. Platform & Stack

### Frontend

- Angular 21 (with standalone components)
- RxJS for reactive programming
- Angular Material for responsive UI
- Ionic/Capacitor (for mobile build)

### Backend

- .NET 10 Web API
- Clean Architecture
- Entity Framework Core (data storage)
- Azure Cognitive Services or OpenAI for AI
- PostgreSQL or Azure SQL for production-ready database

### Optional Hardware

- Wearables (e.g., Fitbit, custom BLE sensor, smart glasses, smart watch)
- Microphone (mobile/wearable) for voice-based mood analysis
- Smartphone

### Coding Version Control

- GitHub: hellotali <https://github.com/hellotali> with email <hellotaliharry@gmail.com>

### Proposed Domain Name

- hellotali.com

### Delopment

- temporarily deploy to a raspberry

---

## 4. Feature Workflow (First Feature)

1. User receives daily notification or voluntarily opens app
2. App prompts 3–5 simple mood and cognitive check-in questions
3. User responds via text or voice
4. Backend processes responses:
   - NLP (sentiment/emotion analysis)
   - Pattern analysis over time (e.g., disorientation, memory gaps)
5. Results:
   - Daily report shown to user
   - Optional caregiver alerts
   - Trend visualization over time

---

## 5. AI Models / APIs to Use

### 1. NLP-based Mood Analysis

**Purpose:** Detect mood from text or voice check-ins

**Tools:**

| Option | Provider/Tool | Notes |
|--------|---------------|-------|
| **Managed APIs** | Azure Text Analytics (incl. health) | Enterprise-ready, HIPAA-eligible |
| | OpenAI GPT-4o-mini (preferred) | Semantic analysis, flexible prompts |
| | Google Cloud Natural Language | Good sentiment/entity detection |
| **Open-source (MIT-friendly)** | spaCy | Fast, customizable NLP pipeline |
| | VADER (NLTK) | Rule-based sentiment, no training needed |
| | ONNX Runtime + DistilBERT/RoBERTa | Emotion classification model |
| **On-device** | ONNX or TensorFlow Lite | Private inference, offline capable |

### 2. Cognitive Decline Detection (Longitudinal)

**Purpose:** Track response patterns over time to detect subtle cognitive changes

**Signals to monitor:**

- Response latency trends
- Word variety / vocabulary richness
- Missed prompts or incomplete responses
- Confusion indicators (repetition, off-topic responses)

**Approach:**

- Supervised learning (train custom model on question/answer consistency)
- Feature inputs: timestamps, word use, forgetfulness, question complexity

**Implementation path:**

1. Train supervised model in Python on response patterns
2. Export to ONNX format
3. Serve in .NET via ONNX Runtime (or TFLite on mobile)

**Tools:**

- Python-trained model exported to ONNX → served in .NET backend
- TensorFlow Lite for mobile (if running on-device)

### 3. Digital Phenotyping (Optional, Opt-in)

**Purpose:** Low-cost behavioral signals that supplement mood scores

| Signal | Source | Privacy Notes |
|--------|--------|---------------|
| Step count / sleep | HealthKit / Google Fit | Requires explicit consent |
| Screen-on rhythm | Device APIs | Aggregated only, no content |
| Basic mobility ("home vs out") | Location services | Coarse location only |

**Analysis approach:**

- Combine with mood scores using trend/anomaly detection
- Methods: z-scores, EWMA (Exponentially Weighted Moving Average)

---

## 6. Analytics & Decisioning

### Trend & Anomaly Detection

| Technique | Purpose |
|-----------|---------|
| Personal baselines | Establish "normal" for each user |
| Moving averages | Smooth daily fluctuations |
| Z-scores | Detect significant deviations |
| Multi-signal agreement | "Nudge only when multiple signals agree" |

### Confidence Scoring & Guardrails

- Require **high confidence** for any alert or nudge
- **Human-in-the-loop** for high-stakes situations
- Graceful degradation when confidence is low

---

## 7. Big Data Consideration (Future)

### Should You Use Big Data?

**Yes, if:**

- You’ll have many users over time
- You want to run cohort analysis, longitudinal trend modeling
- You’re planning research/clinical analysis

**Not necessary at MVP stage**, but design with scalability in mind:

- Azure Cosmos DB
- Azure Synapse
- SQL Server + Azure Blob hybrid setups

---

## 8. Privacy & Safety Principles

| Principle | Implementation |
|-----------|----------------|
| **Privacy-first** | Store scores/aggregates; avoid raw journals/voice by default |
| **Consent & transparency** | Granular opt-ins per data source |
| **Non-diagnostic** | Supportive nudges only; no medical claims |
| **Crisis routing** | Copy routes to local helplines; not a crisis tool |
| **Data minimization** | Pseudonymous userId; PII minimization |

---

## 9. Suggested Hardware (Optional at MVP stage)

| Use Case                    | Hardware                                     | Notes                                          |
| --------------------------- | -------------------------------------------- | ---------------------------------------------- |
| Passive mood tracking       | Microphone (phone or wearable)               | Use for voice tone analysis                    |
| Movement / sleep / vitals   | Wearable (Fitbit, Apple Watch, Garmin, etc.) | Track actigraphy, heart rate variability       |
| Emergency / caregiver alert | IoT pendant with SOS + BLE                   | Sends signals to mobile app via BLE or MQTT    |
| On-device AI                | Android (Edge TPU) / iPhone (Neural Engine)  | Enables private inference without cloud access |

---

## 10. MVP Feature Flow (Your First Feature)

1. User opens app or is prompted once a day
2. App asks a few cognitive + mood questions
3. Voice/text response is sent to backend
4. AI/NLP analyzes mood tone + memory indicators
5. Result is displayed to user & optionally caregiver
6. Longitudinal graph or warning if decline detected

---

## 11. Learning Roadmap to Build the System

### Phase 1: AI & Data Science Fundamentals

- Python for AI/ML scripting
- Mathematics for ML (linear algebra, probability)
- Data analysis with Pandas & NumPy
  - **Courses:** IBM Data Analyst (Coursera), CS50 (Harvard)

### Phase 2: Machine Learning & NLP

- Supervised learning (classification, regression)
- NLP techniques (tokenization, embeddings)
- Sentiment and emotion detection
  - **Courses:** DeepLearning.AI NLP Specialization, Hugging Face NLP Course

### Phase 3: AI in Production

- Model evaluation and deployment
- ONNX Runtime / TensorFlow Lite
- Azure AI Services & OpenAI API usage
  - **Courses:** Full Stack Deep Learning, Azure AI Fundamentals

### Phase 4: Full-Stack Application Development

- Angular + RxJS + Angular Material
- .NET 10 Web API + EF Core
- Mobile app with Ionic/Capacitor
  - **Courses:** Angular and .NET Bootcamps (Udemy/Pluralsight)

### Phase 5: Security & Ethics

- OAuth 2.0 & secure APIs
- Data privacy laws (GDPR, HIPAA)
  - **Courses:** Microsoft Security Compliance Training, OWASP Top 10

---

## 12. Outcome

This first feature will lay the foundation for a larger platform aimed at:

- Proactive mental health care
- Early detection of cognitive decline
- Empowering caregivers with data insights
- Enabling AI-assisted support and escalation

---

## 13. Next Steps

- Finalize wireframes and questions for check-in prompts
- Select cloud provider and AI toolchain (Azure vs OpenAI vs HuggingFace)
- Build backend prototype and integrate with Angular frontend
- Begin pilot testing with consented users or simulated input

---

## 14. Would like to have

### A. Sample System Architecture Diagram

*(to be created on request – includes Angular frontend, .NET API, AI service integration, PostgreSQL, and optional BLE device sync)*

### B. Recommendation for Open-Source Models

| Model | Use Case | License | Notes |
|-------|----------|---------|-------|
| DistilBERT (emotion fine-tuned) | Sentiment/emotion classification | Apache 2.0 | Export to ONNX |
| RoBERTa (emotion fine-tuned) | Multi-emotion detection | MIT/Apache | Hugging Face available |
| EmotionBERT variants | Multi-emotion classification | Check license | Research-backed |
| VADER | Rule-based sentiment | MIT | No training needed |
| T5 | Question understanding | Apache 2.0 | Response patterning |

Use Hugging Face transformers, export as ONNX for integration into .NET

### C. AI Provider Strategy (hellotali)

The system uses a pluggable AI provider architecture:

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

### D. Walkthrough: NLP Backend Setup in .NET + Azure

1. Create .NET 10 Web API project
2. Install Azure.AI.TextAnalytics SDK
3. Store credentials in secure config (KeyVault or appsettings.json)
4. Define NLP analysis service using Azure SDK:
   - Sentiment analysis
   - Entity recognition
5. Integrate endpoint with Angular app via secure API
6. Optionally swap out for OpenAI GPT API for advanced semantics

---
