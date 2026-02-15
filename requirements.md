
# KhetMitra AI — requirements.md

## 0. Context & Goal
**Theme:** AI for Rural Innovation & Sustainable Systems  
**Problem Statement:** Build an AI-powered solution that supports rural ecosystems, sustainability, or resource-efficient systems.  
**Product:** **KhetMitra AI** — An offline-first, multilingual agronomy copilot for smallholder farmers that predicts, advises, and coordinates actions to boost yield, conserve water, reduce input costs, **surface relevant government subsidies/schemes**, and **recommend profitable crop choices**.

---

## 1. Key Personas
- **Smallholder Farmer (primary)** — Low connectivity & digital literacy; prefers voice in local language.
- **FPO/Co-op Lead (secondary)** — Aggregated insights, logistics & market coordination.
- **Extension Worker / KVK (secondary)** — Triage uncertain cases, close the loop.
- **Mandi Trader / Buyer (tertiary)** — Demand signals (future phase).

---

## 2. Functional Requirements (MVP)

### 2.1 Farmer Advisory (Core)
1. **Pest/Disease Diagnosis (Vision):** Capture leaf/plant photos; on-device classifier returns likely issue + confidence; show remedy (dosage, PHI, PPE) with explainable reasoning.
2. **Irrigation & Fertilizer Planner:** Weather-aware daily/weekly schedule based on crop, soil, phenology; nudges via voice & WhatsApp; offline heuristics with periodic sync.
3. **Market & Harvest Hints:** Near-term mandi price bands (nearby eNAM/Agmarknet-linked markets), simple trend indicators, and harvest window suggestions.
4. **Crop Profitability Advisor:** Personalized, district-level **crop selection** recommendations using cost-of-cultivation baselines, local price trends, soil/irrigation constraints, and risk (rain/pest). Outputs **Top 3 crops** with projected **net margin/acre**, **input budget**, and **risk score**.
5. **Govt Schemes & Subsidies Finder:** Eligibility engine maps farmer profile (state, district, landholding size, irrigation, crop, ownership/lease, KYC) to Central & State schemes. Provides **step-by-step application**, required documents, local office/portal, and status tracking.
6. **Voice-first Multilingual UX:** Hindi + local languages; offline TTS/STT; IVR fallback for non-smartphones. Daily 2‑min audio briefing.
7. **Escalations:** Low‑confidence or high‑risk cases → extension worker console; farmer can upload audio/photo notes; SLA & follow-up reminders.

### 2.2 FPO/Extension Console
8. **Case Triage:** Queue of escalations with images, location, crop stage; label/annotate to improve models.
9. **Aggregated Insights:** Heatmaps of pest pressure, irrigation demand, price anomalies (privacy-preserving, no PII).
10. **Scheme Outreach:** Bulk messages to eligible members for time-bound programs (e.g., PM-KUSUM, PMFBY enrolment windows).

---

## 3. Functional Requirements (Schemes & Subsidies — Detailed)

- **Coverage (Phase‑1):**
  - **Central:** PM‑KISAN income support; PMFBY crop insurance; PM‑KUSUM solar pumps/RE plants; **Soil Health Card**; **e‑NAM** registration & training links; **Kisan Credit Card (KCC)** with interest subvention; **Agriculture Infrastructure Fund (AIF)** for post‑harvest & infra.
  - **State (UP initial):** Seed subsidy notifications; plant protection subsidies; training (Million Farmers School); contingency plans; state portals for downloads & forms. 
- **Eligibility Engine:** Rule graph encodes beneficiary type, landholding thresholds, Aadhaar/eKYC needs, timings (seasonality windows), crop or asset prerequisites, and subsidy caps. Supports dynamic updates via remote config.
- **Document Checklist & Auto-fill:** Based on scheme, generate checklists (Aadhaar, land records, bank passbook, photos, quotations). Prefill forms where APIs/consents allow.
- **Application Pathways:** Deep links to official portals, nearest CSC, or assisted filing through FPO. Store **proofs & receipts** in a secure device vault.
- **Alerts:** Reminders for PMFBY cut‑off dates, PM‑KISAN eKYC status, AIF loan stages, PM‑KUSUM vendor empanelment windows.

---

## 4. Functional Requirements (Crop Profitability — Detailed)

- **Inputs:**
  - **Prices:** Daily mandi prices (modal/min/max) via Agmarknet/eNAM feeds; cache per crop-market; 12–36‑month history when online.
  - **Costs:** State & crop‑specific **cost of cultivation** baselines (A2+FL, C2) with inflation update; localized adjustments for irrigation/fertilizer/diesel.
  - **Agro‑climate:** Soil Health Card data (pH, OC, NPK), rainfall/ET0, growing degree days, sowing window.
  - **Constraints:** Irrigation type & availability, risk preference, input budget, mechanization.
- **Model:** Rule‑based feasibility filters → price trend nowcasting (robust median + seasonal component) → expected yield & cost curve → **expected net margin** with conservative risk penalty (weather/pest index).
- **Outputs:** Ranked crops with **(₹/acre)** margin bands, sensitivity to price ±10%, input plan, sowing window alignment, and marketing tips (eNAM/nearby mandis).

---

## 5. Non‑Functional Requirements
- **Offline‑first:** All critical functions usable offline; periodic sync; deterministic fallbacks.
- **Performance:** On‑device inference < 1.5s on 2‑3‑year Android devices; app size ≤ 120 MB including models (quantized).
- **Accessibility:** Voice-first, pictograms, large buttons; color‑safe; supports IVR/SMS.
- **Localization:** Hindi + 1–2 local languages at launch; pluggable i18n.
- **Privacy & Security:** Data‑minimization; opt‑in sync; encryption at rest; consented sharing only; audit trails for console actions.
- **Safety:** Always show PHI, PPE, banned-chemical checks; uncertainty deferral to humans.

---

## 6. Data Sources (for implementation)
- **Prices:** Agmarknet daily price datasets (Open Government Data); eNAM dashboards (for coverage and features).  
- **Costs:** DES Cost of Cultivation/Production estimates; Agricultural Statistics at a Glance (tables for yields/prices).  
- **Schemes:** Official portals for PM‑KISAN, PMFBY, PM‑KUSUM, AIF, KCC interest subvention (NABARD); Soil Health Card portal; eNAM; **UP Agriculture** departmental portals for state subsidies & notifications.  

> Exact URLs are listed in *design.md* and wired into data adapters.

---

## 7. Constraints & Assumptions
- Government portals may change endpoints; implement adapter pattern + remote-config for rules.
- Not all farmers have smartphones; WhatsApp/IVR must deliver essential features.
- Legal/compliance: align with label compliance for pesticides; no brand promotion; follow official advisories.

---

## 8. Acceptance Criteria (MVP)
1. **Diagnosis:** ≥75% top‑3 accuracy on pilot validation set; uncertainty deferral on <60% confidence.
2. **Water/Fertilizer Planner:** Demonstrated input savings (≥10%) vs control in 1 season (proxy by schedule adherence & rainfall avoidance days).
3. **Profitability Advisor:** Produces **Top‑3 crop** list with margin bands for at least 10 priority crops per district; matches historical best choice in ≥60% back‑tests.
4. **Schemes Finder:** Correct eligibility & checklist for the 7 targeted schemes (Central) and ≥5 state items; deep links/CSCs verified.
5. **Voice UX:** 80% of test users can complete a task with voice (Hindi) within 3 prompts.
6. **Offline:** All core flows available without network; sync succeeds under 2G.

---

## 9. Compliance & Safety
- **Pesticide safety language** and **PHI** always displayed; block banned substances.
- **Insurance disclaimers:** PMFBY enrollment/claims routed to official channels; app acts as facilitator, not insurer.
- **Data protection:** Consent records stored; opt‑out & data delete supported.

---

## 10. Rollout Plan (12 weeks)
- **Weeks 1–2:** Partner FPO/KVK; recruit 100–150 farmers; collect farm profiles & soil data; set up IVR.
- **Weeks 3–6:** Release MVP; run daily briefings; start scheme outreach (PMFBY cut‑off, PM‑KUSUM vendor camps).
- **Weeks 7–10:** Iterate models; launch crop profitability; validate with receipts & outcomes.
- **Weeks 11–12:** Measure impact; publish pilot report; prep for state‑wide scale.

---

## 11. Future Scope
- Carbon co‑benefits & practice incentives; credit/insurance facilitation; OR‑Tools for collection routing; cooperative marketplace; satellite indices.

