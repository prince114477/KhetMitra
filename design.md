
# KhetMitra AI — design.md
---

## 0) Architecture (High‑level)
```
Android App (offline-first, Hindi-first)
 ├─ UI & Voice: large buttons, pictograms, device TTS for summaries
 ├─ GrainDB-mini (on-device CSV/JSON bundle)
 │    • prices.csv, costs.json, yields.json
 │    • soil_profiles.json, climate_normals.json
 │    • sowing_windows.json (+ water_availability.json opt)
 ├─ Core Engines (on-device)
 │    • Crop Decision Engine (CDE): profit, suitability, risk, MBI → Top-3
 │    • PlannerLite: irrigation/fertilizer rules by crop/soil/stage
 │    • DiagnosisStub / Vision-lite: leaf issue + remedy + PHI/PPE
 │    • SchemesEngine: YAML rule packs for PM-KISAN/PMFBY/PM-KUSUM + state items
 └─ Storage & Sync
      • Encrypted local store; optional future sync for anonymous counters & rule updates

(Cloud / Post-hackathon)
 ├─ Adapters: price (OGD Agmarknet), weather, soil labs, eNAM
 ├─ Model registry & active learning for vision
 ├─ Dashboards (FPO/policy) with aggregated analytics (no PII)
 └─ Rule/Config distribution for GrainDB and schemes
```

---

## 1) Data Model (GrainDB‑mini)
- **prices** `(date, mandi, crop, modal_price)`
- **costs** `{ crop: { A2FL, C2 } }`
- **yields** `{ crop: { yield_kg_per_acre } }`
- **soil_profiles** `{ name, ph, oc, texture }`
- **climate_normals** `{ month, rain_mm, tmin, tmax }`
- **sowing_windows** `{ crop: { start_mmdd, end_mmdd } }`
- **water_availability** `{ irrigation_type, hours_per_day }`
- **schemes** YAML rule packs `{ id, eligibility, documents, steps, links }`

---

## 2) Algorithms
### 2.1 Price Nowcast
`nowcast_price = median(modal_price over last 30 days)`; fallback to seasonal median or last known.

### 2.2 Profitability (₹/acre)
`profit_A2FL = nowcast_price × expected_yield − A2FL`  
`profit_C2 = nowcast_price × expected_yield − C2` (conservative view)

### 2.3 Suitability (0..1)
- Soil pH/texture match, sowing window check, irrigation adequacy → weighted sum.

### 2.4 Risk Penalty (0..1)
- Rainfall variability proxy + static pest prior per crop; conservative clamp.

### 2.5 Market Balance Index (MBI)
- Anti‑herd factor using anonymous share of current selections in the block/cluster; capped to avoid over‑penalization.

### 2.6 Final Score & Top‑3
`score = w1·normalize(profit_A2FL) + w2·suitability + w3·(1 − risk) + w4·MBI`  
Return **Top‑3** + explanations + ±10% sensitivity.

---

## 3) UX Flows
1) **Crop Choice (primary):** Q&A → Top‑3 with ₹/acre, suitability, risk, MBI; voice summary.  
2) **Planner:** Today/Tomorrow irrigation/fertilizer hints keyed to the selected crop + soil/stage.  
3) **Diagnosis:** Camera → label + remedy + PHI/PPE; “Not sure?” → escalate.  
4) **Schemes:** Eligible programs & steps; link to portals/CSC; remind me.

---

## 4) Files & Contracts
```
/demo_data/
  prices.csv, costs.json, yields.json
  soil_profiles.json, climate_normals.json
  sowing_windows.json, water_availability.json
  schemes/pmkisan.yaml, pmfby.yaml, pmkusum_b.yaml, ...
  config.json (weights, thresholds)
```

---

## 5) Security, Privacy, Safety
- **On‑device by default**, encryption at rest; no PII uploaded.  
- Anonymous counters only for MBI when sync is enabled.  
- Safety text always shows PHI/PPE; block banned pesticides; explainability by design.

---

## 6) Evaluation & Metrics
- **Technical:** time‑to‑Top‑3 ≤ 1.5s; offline success.  
- **User:** NPS of crop choice flow; % actions following Top‑3; adherence to planner hints.  
- **Impact (pilot):** input savings, realized margins vs last season, reduced same‑crop clustering within a block.

---

## 7) Roadmap Notes
- Plug in live feeds (OGD price, weather); richer risk/pest models; full vision model; extension console; dashboards; marketplace & logistics; credit/insurance aligned to chosen crop.
