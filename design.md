
# KhetMitra AI — design.md

## 0. Overview
**Objective:** Deliver an offline-first, explainable agronomy copilot with (a) disease diagnosis, (b) water/fertilizer scheduling, (c) market & harvest hints, **(d) scheme/subsidy guidance**, and **(e) crop profitability recommendations** for smallholder farmers.

---

## 1. Architecture (High-level)

```
Android App (offline-first, Kotlin/Flutter)
 ├── Voice & Local Language UI (TTS/STT, on-device where possible)
 ├── Camera & Media (leaf segmentation, capture guides)
 ├── Farmer Profile & Field Map (GPS polygon, soil type, crop, sowing date, irrigation)
 ├── Edge AI Pack (TFLite/ONNX-quantized)
 │    • Vision: Disease/deficiency classifier + leaf segmentation
 │    • Tiny LLM + RAG: safety-checked agronomy cards
 │    • Schedulers: irrigation/fertilizer rules (ET0/Kc heuristics)
 │    • Crop Profitability: lightweight nowcast + rule-based filters
 │    • Scheme Engine: rule graph for eligibility & checklists
 └── Sync Manager (when online)
      • Price & weather fetchers
      • Scheme rule updates & portal metadata
      • Telemetry (privacy-preserving)

Cloud/Hub (optional, intermittent)
 ├── Data adapters: Agmarknet/eNAM, DES Cost datasets, weather APIs
 ├── Knowledge Graph: crop-stage → symptom → remedy → safety/PHI
 ├── Model registry + active learning (vision feedback)
 ├── Case triage console (extension workers)
 └── FPO dashboard
```

---

## 2. Modules & Responsibilities

### 2.1 Vision Diagnosis
- **Model:** EfficientNet-Lite / MobileNetV3 + DeepLabv3-lite segmentation.
- **Outputs:** Top‑k classes with confidence; causal explanation snippets retrieved from knowledge cards; uncertainty threshold → human escalation.
- **On-device budget:** ~10–20 MB model, int8 quantized.

### 2.2 Scheduler (Water/Fertilizer)
- **Method:** ET0 (Penman‑Monteith approximation) × crop coefficient (Kc) + rainfall forecast. Heuristic rules for soil type and phenology.
- **Data:** Weather (IMD/3rd-party), soil (SHC), crop stage from farmer inputs.

### 2.3 Market & Harvest Hints
- **Price Data:** Agmarknet daily prices; local mandi mapping. Price trend nowcasting with seasonal decomposition (median filter + STL-like smoothing); alerts for favorable bands.
- **Harvest Window:** Align with expected degree-days; warn against rainfall events.

### 2.4 Crop Profitability Advisor (CPA)
- **Goal:** Recommend crops that maximize **expected net margin/acre** given constraints.
- **Inputs:**
  - Prices: last 12–36 months mandi data for crops grown locally.
  - Costs: **DES Cost of Cultivation/Production** baselines by state/crop (A2+FL/C2); adjust for inflation & local factors (diesel, irrigation).
  - Yield priors: from DES/Agri Stats at a Glance; user historical yields where available.
  - Soil/Water constraints: SHC parameters (pH, OC, NPK), irrigation availability/type.
- **Pipeline:**
  1) **Feasibility filter** (soil pH range, water needs, sowing window).  
  2) **Price nowcast** (robust trend + seasonality; conservative CI).  
  3) **Cost & yield model** → compute **Net Margin = (Price_nowcast × Yield) − Cost(A2+FL)**; also compute **C2** for conservative view.  
  4) **Risk adjustment:** rainfall variability index + pest risk prior → penalty.  
  5) Rank Top‑3 with sensitivity (±10% price) and show **why** (soil fit, market trend, input budget).
- **Explainability:** Show components and assumptions; provide references to data sources in-app.

### 2.5 Schemes & Subsidies Engine
- **Scope (Phase‑1):**
  - **PM‑KISAN** — income support & eKYC status helper.  
  - **PMFBY** — crop insurance enrollment windows, premium calculator link, claim & grievance helplines.  
  - **PM‑KUSUM** — Components A/B/C, subsidy pattern, vendor lists & rate cards (state‑wise).  
  - **KCC** (Interest Subvention) — credit at 7% with prompt repayment incentive; doc checklist & bank locator.  
  - **AIF** — 3% interest subvention up to ₹2 crore, credit guarantee; eligible projects; beneficiary registration.  
  - **Soil Health Card** — lab locator, sample submission, SHC retrieval.  
  - **eNAM** — farmer registration, benefits, training material.  
  - **UP State** — seed & plant‑protection subsidies, Million Farmers School, contingency advisories.  
- **Design:** YAML rule packs per scheme: eligibility, documents, steps, deep links/IVR codes, seasonal windows; versioned & hot‑swappable via remote config.
- **Privacy:** Store only what is needed; on-device first; explicit consent for any uploads.

---

## 3. Data Adapters (Authoritative Sources)
- **Prices:**
  - **Agmarknet** daily prices via **data.gov.in** API; periodic batch sync; fall back to cached CSV if offline.  
  - **eNAM** for coverage, registration & training references.  
- **Costs & Yields:**
  - **DES Cost of Cultivation/Production** datasets; **Agricultural Statistics at a Glance** for yields & value-of-output tables.  
- **Schemes:**
  - **PM‑KISAN**, **PMFBY**, **PM‑KUSUM** (MNRE), **AIF**, **NABARD KCC/Interest Subvention**, **Soil Health Card**, **eNAM**, **UP Agriculture** portals.  

> See *References* for URLs to wire into adapters.

---

## 4. Data Model (Simplified)

```yaml
Farmer:
  id: uuid
  language: [hi-IN, en-IN, ...]
  contact: phone
  location: {state, district, block, village}
  land_parcels: [Parcel]
  kyc: {aadhaar_last4, ekyc_status?}
  preferences: {risk_tolerance: low/med/high}

Parcel:
  id: uuid
  geometry: polygon | centroid
  soil: {ph, oc, n, p, k, texture}
  irrigation: {type: rainfed|canal|borewell|drip, hours_per_day}
  crop_plan: {crop, variety, sowing_date, stage}

SchemeEligibility:
  scheme_id: string
  eligible: boolean
  reason: string
  checklist: [Document]
  actions: [link | ivr_code | csc_visit]

ProfitabilityRecord:
  crop: string
  market_refs: [mandi_id]
  price_nowcast: {p50, p10, p90}
  yield_est: {p50, p10, p90}
  cost_est: {A2FL, C2}
  risk_penalty: number
  net_margin: {p50, band}
```

---

## 5. UX Flows (Farmer app)
1. **Onboarding:** Choose language → capture village & land size → select crops → optional SHC import → enroll for briefings.
2. **Diagnose:** Tap camera → capture leaf → get result + remedy + safety + option to escalate.
3. **Schedules:** Home card shows today/tomorrow irrigation/fertilizer tasks (voice + pictograms).
4. **Schemes:** “**Anudan & Yojna**” card lists eligible items; tap → see benefits, steps, docs, portal/IVR; add reminders.
5. **Crop Choice:** Answer a few questions (water, budget, risk). See Top‑3 crops with margins; tap to view assumptions and mandi references.

---

## 6. Security & Privacy
- **On-device first;** encrypted storage; explicit consent for sync; redact PII in dashboards; logs scrubbed.
- **Compliance:** Follow pesticide label rules; block banned chemicals; disclaimers for insurance & finance referrals.

---

## 7. Tech Stack
- **Mobile:** Kotlin (Android) or Flutter; TFLite/ONNX Runtime; Room/SQLite; WorkManager for sync.
- **Voice:** Vosk/Silero class (offline) + Android TTS; optional IVR via Twilio/KooKoo/Exotel (region partner).
- **Server (optional):** FastAPI/Node for adapters; PostgreSQL; Redis cache; Airflow/Prefect for ETL.
- **Console:** React or SvelteKit; MapLibre for heatmaps.

---

## 8. Model Training & Evaluation
- **Vision:** Public agri-disease datasets + local annotations; LoRA fine-tune; evaluate top‑k accuracy; uncertainty calibration.
- **CPA:** Back‑test recommendations vs realized margins (historical prices & DES costs); track regret metrics.
- **Safety:** Red-team prompts for LLM; rule‑tests for scheme engine.

---

## 9. References (authoritative)
- Agmarknet daily prices (OGD API)
- eNAM portal (coverage, benefits, registration)
- DES — Cost of Cultivation/Production estimates & manuals; Agri Stats at a Glance
- PM‑KISAN (portal & beneficiary status)
- PMFBY (portal, operational guidelines)
- PM‑KUSUM (MNRE portal & guidelines)
- NABARD — Interest Subvention / KCC
- AIF (portal, scheme guidelines)
- Soil Health Card (portal)
- UP Agriculture Department (state subsidies, advisories)

> Actual URLs and citations are provided in the accompanying message and can be embedded into the app as metadata.

---

## 10. Roadmap Notes
- **P0:** Vision diagnosis, schedules, scheme finder, basic price bands, crop profitability (Top‑3).
- **P1:** Marketplace coordination (transport/cold‑store slots), credit/insurance facilitation, regenerative practice tracking.
- **P2:** Satellite indices in offline pack; OR‑Tools routing; carbon MRV hooks.

