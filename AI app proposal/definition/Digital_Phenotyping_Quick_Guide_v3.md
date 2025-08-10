
# Digital Phenotyping — Quick Guide for Your App

## What is Digital Phenotyping?
**Digital phenotyping** is the measurement of a person’s behavior, physiology, and routines using data from everyday devices—mainly smartphones and wearables—to infer health states (e.g., mood, cognition, sleep) in real time.

---

## How It Works
- **Passive signals** (automatic, no user input):  
  - GPS (mobility patterns), accelerometer/gyroscope (activity, gait)  
  - Screen on/off, app usage durations, notification counts  
  - Call/text **counts** (not content)  
  - Keystroke dynamics (speed, variability)  
  - Microphone features (speech tempo/energy; avoid storing raw audio)  
  - Wearables: heart rate/HRV, sleep duration/fragmentation
- **Active signals** (brief user prompts / EMA):  
  - Daily mood check-ins, micro cognitive tasks  
  - Short voice or text journals

- **Models & analytics**:  
  - Time-series ML, personalized baselines, anomaly detection  
  - Multisignal fusion (e.g., circadian disruption + lower mobility + negative sentiment → possible downturn)

---

## Why It’s Useful (Your App Context)
- **Mental health**: track mood trends, detect relapse risk, nudge timely coping skills.  
- **Dementia care**: monitor routine/mobility changes, gait slowing, nighttime wandering, response latency/language shifts—flag potential decline and alert caregivers.

---

## Starter Signals to Consider
- **Mobility entropy**: variety of places visited per day (aggregate, not raw paths)  
- **Circadian regularity**: stability of sleep/wake and phone-usage rhythm  
- **Activity level**: steps and accelerometer summaries  
- **Social rhythm**: counts of calls/messages; no content  
- **Keystroke dynamics**: typing speed/variability (opt-in)  
- **Speech prosody**: energy/rate from opt-in voice notes (no raw storage)  
- **Wearable HRV & sleep**: restorative vs. fragmented sleep

---

## Privacy, Ethics, and Pitfalls
- **Consent & transparency**: clear opt-in; explain what, why, and how long.  
- **Minimize & secure**: collect the least needed; encrypt at rest/in transit; rotate identifiers.  
- **On-device first**: compute features locally; send only aggregates where possible.  
- **Bias & context**: travel/illness can look like “risk”; include human-in-the-loop for high-stakes alerts.  
- **Battery & gaps**: sample intelligently; design models to handle missing data.

---

## MVP Integration Checklist
1. **Start with active check-ins** (daily mood + 1–2 micro cognitive prompts).  
2. **Add 2–3 passive features** with low battery cost:  
   - Screen on/off rhythm, step count (HealthKit/Google Fit), GPS “home vs. out” time.  
3. **Compute simple features** on device or server:  
   - Weekly trend, deviation from personal baseline (z-scores), composite “change” score.  
4. **Alerting**: trigger only when multiple signals agree and confidence is high; show a user-facing explanation (“why we alerted”).  
5. **Governance**: align with HIPAA/GDPR; retention limits; audit logs; opt-out controls.  
6. **Evaluation**: monitor precision/recall of alerts; collect user feedback; iterate thresholds.

---

## Implementation Notes (Tech Hints)
- **Mobile**: use HealthKit/Google Fit, ActivityRecognition APIs, BackgroundTasks/WorkManager.  
- **Backend**: store aggregates; schedule feature computation; anomaly detection with EWMA/z-score.  
- **Models**: start simple (rule + thresholds), then graduate to Random Forest/XGBoost; personalize baselines per user.  
- **UX**: privacy-first onboarding; granular toggles; battery usage disclosure; clear benefits (“what you get”).

---

*This guide is tailored for your AI-assisted mood & cognitive monitoring MVP. Expand gradually, validate ethically, and keep users in control.*
