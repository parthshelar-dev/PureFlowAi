# 🌊 PureFlowAi

**An Agentic AI Platform for SDG 6 — Clean Water & Sanitation**

*IBM Project-Based Internship Submission*

PureFlowAi is a multi-agent AI system, built entirely in [n8n](https://n8n.io), that helps municipal water utilities detect and respond to three of the sector's most persistent problems in real time: **water leakage & non-revenue water (NRW)**, **water quality contamination**, and **reactive equipment maintenance**.

Rather than three disconnected tools, PureFlowAi runs three specialist AI agents alongside a fourth **Orchestrator Agent** that reasons across all of them together — producing one prioritized operations briefing, the way a human utility manager would triage across departments.

---

## The Problem

Water utilities routinely lose treated water, miss early signs of contamination, and repair equipment only after it fails — because detection, monitoring, and maintenance are handled separately and reactively.

| # | Problem | Why it's still unsolved |
|---|---|---|
| 1 | **Water Leakage & Non-Revenue Water** | Detection tools exist, but operational response is slow — leaks are found, not necessarily *acted on* quickly. |
| 2 | **Water Quality & Contamination Monitoring** | Testing is often periodic, not continuous — contamination can go unnoticed between checks. |
| 3 | **Reactive Infrastructure Maintenance** | Pumps, valves, and pipelines are repaired after failure rather than before, causing avoidable service interruptions. |

PureFlowAi treats these as one connected operational problem, not three separate ones.

---

## System Architecture

![PureFlowAi Workflow](./images/PureFlow%20Ai%20Workflow.png)

```
Simulated Sensor Data (flow, pressure, pH, turbidity, chlorine)
        │
        ├──▶ Leak Detection Agent ───────┐
        ├──▶ Water Quality Agent ────────┼──▶ Orchestrator Agent ──▶ Severity-Gated Email Alert
        └──▶ Maintenance Agent ──────────┘
```

Each specialist agent independently detects anomalies in its own domain, reasons about probable cause and severity, and logs a structured incident. The **Orchestrator Agent** then reads all open incidents across all three domains and produces a single ranked briefing — flagging what genuinely needs attention first, regardless of which "department" it came from.

---

## How Each Agent Works

### 🔧 Leak Detection Agent
Calculates Non-Revenue Water percentage from simulated inflow/outflow readings per zone (`NRW% = (inflow − outflow) / inflow × 100`). Flags anything above a 10% threshold, then reasons about whether the cause is a genuine pipe leak, a faulty meter, or theft — and assigns severity (Low / Medium / High) with a specific field action.

> **Example (real test output):** Zone_C — Inflow 111.43 LPM, Outflow 70.82 LPM, Pressure 2.18 bar → **NRW 36.44%** → Severity: **High** → *"Investigate Zone_C immediately, use acoustic sensors to locate leak source."*

### 💧 Water Quality Agent
Runs a multi-factor safety check on pH, turbidity, and chlorine residual — flagging a zone if *any* parameter falls outside safe range. It then interprets what the specific combination implies (e.g. low chlorine alone suggests a disinfection failure; multiple parameters off together suggest a genuine contamination event) and recommends whether a do-not-consume advisory is warranted.

> **Example (real test output):** Zone_A — pH 9.44, Turbidity 12.63 NTU, Chlorine 0.12 mg/L (all three flagged) → Severity: **High** → *"Alert treatment plant operators and public health authorities, consider a do-not-consume advisory."*

### ⚙️ Predictive Maintenance Agent
Computes a weighted risk score for each piece of equipment from its age, time since last service, total runtime hours, and prior failure count — then predicts the likely failure mode and priority before a breakdown occurs.

```
risk_score = (age_years × 2) + (max(0, days_since_maintenance − 180) × 0.3)
           + (runtime_hours / 1000 × 1.5) + (failure_count × 8)
```

> **Example (real test output):** VALVE_01 — 11 years old, 700 days since maintenance, 4 prior failures → Risk score **258** → Priority: **Critical** → *"Inspect and replace valve seats and seals, consider unplanned shutdown."*

### 🧭 Orchestrator Agent
Pulls every open incident from all three domains and ranks them by true operational urgency — public health risks generally outrank equipment risks, which generally outrank pure water-loss risks, unless an equipment issue threatens imminent total service failure.

> **This is the strongest evidence of genuine multi-agent reasoning:** while triaging a critical valve failure, the Orchestrator independently noted it *"could disrupt water supply in Zone_A"* and connected it to the **existing water-quality contamination event already flagged in the same zone** — without being explicitly told to cross-reference. That's cross-domain reasoning a single-purpose script wouldn't produce.

Only when the briefing contains a **Critical** or **High** item does it trigger an email alert — deliberately avoiding alert fatigue. The email itself, however, always shows the *complete* picture (including lower-priority items), since real operational decisions need full context, not just the triggering incident.

---

## Tech Stack

| Component | Tool |
|---|---|
| Orchestration / agent logic | [n8n](https://n8n.io) |
| LLM reasoning (all 4 agents) | Groq API — Llama 3.3 70B / Llama 3.1 8B |
| Data storage | Google Sheets |
| Alerting | Gmail |

Built entirely on free-tier tooling — no paid infrastructure required to reproduce or extend.

---

## Data Approach

No real IoT sensor network is available for this prototype, so sensor data is **synthetically generated** with realistic value ranges and randomly injected anomalies (leaks, contamination events, equipment degradation). Safe-range thresholds for pH, turbidity, and chlorine are grounded in real-world water quality standards rather than arbitrary values. This is a deliberate, disclosed prototyping choice — the goal of this submission is to demonstrate the agentic reasoning and operational architecture, which is directly transferable to a real sensor feed.

To keep the AI's reasoning honest, ground-truth "this is a simulated anomaly" flags are logged for our own verification but are **never included in what the agents see** — each agent reasons purely from raw sensor values, the same way it would with real hardware.

---

## Repository Contents

```
PureFlowAi/
├── PureFlowAi.json      # Full n8n workflow export (importable, no credentials included)
├── images/               # Workflow architecture screenshot
├── LICENSE
└── README.md
```

### Running it yourself
1. Import `PureFlowAi.json` into your own n8n instance (`⋯ menu → Import from file`).
2. Connect your own credentials: Google Sheets (with matching tabs — see agent details above), a Groq API key, and Gmail.
3. Set up the required Google Sheets tabs: `Sensor_Log`, `Quality_Log`, `Equipment_Registry`, `Leak_Incidents`, `Quality_Incidents`, `Maintenance_Incidents`.
4. Run the Schedule Trigger nodes to start generating and reasoning over data.

---

## Future Work

- Replace simulated data with a real IoT/SCADA sensor feed
- Incorporate weather API data (rainfall, flood alerts) as contextual input for the Leak and Quality agents
- Add GIS/location awareness for pipeline and reservoir mapping
- Citizen-reported issues (via chatbot or form) as a fifth input source, merged into the Orchestrator's briefing
- Daily digest email for Medium/Low priority items to complement real-time Critical/High alerts

---

## SDG 6 Alignment

PureFlowAi directly supports **SDG 6 (Clean Water & Sanitation)** by reducing water loss (financial and environmental), protecting public health through faster contamination response, and improving service reliability through predictive rather than reactive maintenance.

---

## Team

- **Parth Shelar**
- **Atharva Gangurde**
- **Khushi Chandak**

This project was designed, built, and tested collaboratively as a team — from workflow architecture to agent logic to documentation.

*IBM Project-Based Internship*

---

## License

This project is licensed under the terms of the [MIT License](./LICENSE).
