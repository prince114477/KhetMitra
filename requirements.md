
# KhetMitra AI — requirements.md

> **Main USP (updated):** KhetMitra AI’s **Crop Decision Engine (CDE)**—powered by an on‑device **GrainDB** (climate • location • soil • prices • costs • sowing windows)—tells farmers **which crop to grow *right now*** for maximum profit and lower risk. **All other features remain** (diagnosis, irrigation/fertilizer planner, schemes, market hints, voice offline, human‑in‑loop), but the *primary* story is **crop choice first**.

---

## 0) Context & Objectives
- **Theme:** AI for Rural Innovation & Sustainable Systems
- **Objective:** Improve farmer incomes and national food‑system stability by making **crop selection** decisions data‑driven and localized, while still assisting day‑to‑day agronomy.
- **Tagline:** *Grow the right crop now; manage the field better—all offline.*

---

## 1) Personas
- **Smallholder Farmer (primary):** Low connectivity; wants a simple, Hindi‑first guide to **choose a crop now** and manage inputs.
- **FPO/Co‑op Lead (secondary):** Needs aggregate signals (no PII) to coordinate inputs and marketing.
- **Extension Worker (secondary):** Takes escalations when the app is uncertain; validates tough cases.
- **Trader/Buyer (tertiary, later):** Looks at aggregated supply signals (post‑hackathon).

---

## 2) Functional Requirements (MVP)

### 2.1 Crop Choice (Primary Flow) — **CDE + GrainDB**
1. **GrainDB (on‑device subset):** `prices.csv`, `costs.json` (A2+FL/C2), `yields.json`, `soil_profiles.json`, `climate_normals.json`, `sowing_windows.json`; optional `water_availability.json`.
2. **Scoring:**
   - **Profitability:** `nowcast_price × expected_yield − A2+FL` (show C2 as conservative view).
   - **Suitability:** soil/irrigation fit + sowing window.
   - **Risk:** rainfall variability & simple pest prior → penalty.
   - **Market Balance Index (MBI):** anti‑herd term to avoid gluts/shortages.
   - **Score:** weighted sum; output **Top‑3 crops** with margins (₹/acre), risk band, suitability, **MBI note**, and **±10% sensitivity**.
3. **Q&A:** Ask 3–5 questions (irrigation, risk tolerance, land size, soil profile) before ranking.
4. **Explainability:** Show a compact “Why” card: price→yield→cost bars, sowing strip, and a one‑line MBI note.

### 2.2 Agronomy Copilot (Still in MVP)
5. **Pest/Disease Diagnosis (Vision):** Photo → likely issue + confidence; step‑by‑step remedy with dosage, PHI/PPE; escalate if uncertain.
6. **Irrigation & Fertilizer Planner:** Rule‑based daily/weekly hints (weather‑ready later) by crop/soil/stage; optional voice tips.
7. **Market & Harvest Hints:** Near‑term mandi price bands (from local CSV) + simple trend cues; sow/harvest window reminders.
8. **Schemes & Subsidies Finder:** PM‑KISAN, PMFBY, PM‑KUSUM and state items; eligibility, document checklist, steps, deep links/CSC.
9. **Voice‑first, Multilingual:** Hindi at launch; simple prompts and summaries via device TTS.
10. **Escalations:** Low‑confidence cases routed to an expert console (stub in demo).

---

## 3) Non‑Functional Requirements
- **Offline‑first:** All core flows usable without internet; data bundle ships with the app.
- **Performance:** Top‑3 crop results in ≤ 1.5s on mid‑range Android.
- **Size:** ≤ 120 MB (models + GrainDB‑mini + media).
- **Privacy:** On‑device by default; only anonymous counters (for MBI) if/when sync is enabled.
- **Safety:** Always show PHI, PPE; block banned substances; conservative defaults when unsure.

---

## 4) Data Bundle (Demo)
```
/demo_data/
  prices.csv                 # date,crop,mandi,modal_price
  costs.json                 # { crop: { A2FL, C2 } }
  yields.json                # { crop: { yield_kg_per_acre } }
  soil_profiles.json         # [ { name, ph, oc, texture } ]
  climate_normals.json       # [ { month, rain_mm, tmin, tmax } ]
  sowing_windows.json        # { crop: { start_mmdd, end_mmdd } }
  water_availability.json    # optional, by parcel or persona
  schemes/*.yaml             # pmkisan.yaml, pmfby.yaml, pmkusum_b.yaml (etc.)
```

---

## 5) Acceptance Criteria (judge‑friendly)
- **Crop Choice:** App completes Q&A → **Top‑3** with ₹/acre margins, risk, suitability, MBI note, and sensitivity in **Airplane mode**.
- **Diagnosis/Planner/Schemes:** Each opens and renders at least one realistic example with safety & steps.
- **Explainability:** Each recommendation accompanies a “Why” card (transparent & actionable).
- **Demo assets:** A 90‑sec screen recording and 5–6 slides summarizing the flows.

---

## 6) Build Plan (48–72 hrs)
- **T–72 to T–48:** GrainDB‑mini, CDE scoring, Crop Q&A → Top‑3 UI.
- **T–48 to T–24:** Explainability visuals; Hindi TTS; diagnosis stub; schemes YAML; planner rules.
- **T–24 to T–0:** Polish; local notifications; record demo; finalize slides.

---

## 7) Roadmap (post‑hackathon)
- Live feeds (Agmarknet/OGD, weather) and stronger risk models; satellite/pest indices; expert console; policy/FPO dashboards (aggregated); marketplace & logistics; credit/insurance facilitation aligned to chosen crop.
