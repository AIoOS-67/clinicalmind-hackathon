# ClinicalMind 🏥🤖

> **AI shouldn't ask you to trust it blindly — especially in medicine.**

ClinicalMind is a multi-agent AI system that determines whether a hospitalized patient is truly ready for discharge. Built on **Google Cloud Agent Builder** + **Gemini 2.5 Pro** + **MongoDB Atlas**, it produces transparent, evidence-cited clinical decisions that physicians can verify before signing off.

**Submission for:** [Google Cloud Rapid Agent Hackathon 2026](https://rapid-agent.devpost.com/) — **MongoDB Track**

---

## 🎯 The Problem

Every year, premature hospital discharges lead to preventable readmissions and patient harm. Discharge decisions require synthesizing vitals trends, lab trajectories, medication safety, nursing assessments, and PT evaluations — simultaneously. A single overworked physician cannot reliably catch every drug interaction, Beers Criteria violation, or subtle vital sign pattern. ClinicalMind is the AI co-pilot that does.

---

## 🏗️ Architecture

Three specialized AI agents work in a chain — each with a distinct role, escalating responsibility:

```
User / Physician
      |
      v
┌─────────────────────────────────────────┐
│  🟢 DischargeReady                       │
│  AI Discharge Readiness Assessor         │
│  Model: Gemini 2.5 Pro                   │
│  Reads: 4-day vitals, labs, nursing,     │
│  PT assessment, medication profile       │
│  Output: READY / CONDITIONAL / NOT READY │
└──────────────┬──────────────────────────┘
               │ A2A transfer
               v
┌─────────────────────────────────────────┐
│  🟠 DischargePlanner                     │
│  Care Transition Orchestrator            │
│  Model: Gemini 2.5 Pro                   │
│  Tools: Google Search + URL Context      │
│  Researches: drug interactions, Beers    │
│  Criteria, medication adjustments        │
│  Output: Full structured discharge plan  │
└──────────────┬──────────────────────────┘
               │ A2A transfer (standalone)
               v
┌─────────────────────────────────────────┐
│  🔴 RxSafe                               │
│  Medication Safety Reviewer              │
│  Model: Gemini 2.5 Pro                   │
│  Tools: Google Search + URL Context      │
│  Checks: DDI, Beers Criteria, STOPP/     │
│  START, polypharmacy, duplicate therapy  │
│  Output: risk_level LOW|MEDIUM|HIGH      │
└─────────────────────────────────────────┘
```

---

## 🍃 MongoDB Atlas Integration

MongoDB Atlas is the FHIR-compliant patient data backbone:

- **Cluster:** `clinicalmind-cluster` (Google Cloud, us-central1, Free M0)
- **Database:** `clinicalmind`
- **Collections:** `patients` (FHIR Bundle), `observations` (vitals/labs), `medications`
- **MCP Server:** `https://mcp.mongodb.com/sse` — enables agents to query patient data via the Model Context Protocol
- **Vector Search:** Atlas Vector Search indexes on `observations` for semantic similarity queries
- **Sample Data:** 100MB+ of realistic synthetic FHIR patient data loaded for development

---

## ☁️ Google Cloud Setup

| Component | Details |
|-----------|---------|
| **Project** | `clinicalmind-hackathon` |
| **Region** | `us-west1` |
| **Agent Builder** | Vertex AI Agent Builder (Agent Studio) |
| **Model** | Gemini 2.5 Pro |
| **APIs** | Vertex AI, Agent Builder, Cloud Storage, Secret Manager |

### Agent IDs (Vertex AI Agent Builder)
```
RxSafe:           agent_1778705313896
DischargePlanner: agent_1778705469401
DischargeReady:   agent_1778705717368
```

---

## 🧪 Live Test Results

**Test Patient: Margaret Chen, 78F — Hip Fracture s/p ORIF, Day 4**

```json
{
  "discharge_recommendation": "CONDITIONAL",
  "medication_safety": {
    "beers_violations": [
      "diphenhydramine: anticholinergic risk in elderly",
      "ibuprofen: GI bleeding + renal risk"
    ],
    "drug_interactions": [
      "warfarin + ibuprofen: HIGH bleeding risk",
      "ibuprofen + lisinopril: reduced antihypertensive effect",
      "ibuprofen + furosemide: reduced diuretic effect",
      "lisinopril + furosemide: excessive BP drop risk",
      "furosemide + metformin: lactic acidosis risk",
      "atorvastatin + amlodipine: myopathy risk"
    ]
  },
  "physician_sign_off_required": [
    "Discontinuation of ibuprofen and diphenhydramine",
    "Confirmation of follow-up appointments",
    "Acknowledgement of medication interaction risks"
  ]
}
```

**Multi-demographic testing:**
- ✅ Margaret Chen, 82F (elderly polypharmacy) — CONDITIONAL verdict, 7 tool calls
- ✅ Li Na Wang, 30F (H. pylori gastritis) — ibuprofen MAJOR risk correctly flagged
- ✅ Wei Zhang, 18M (trauma) — DDI checking applicable
- ✅ Xiao Ming Baby, 10mo M (pediatric) — Beers Criteria correctly N/A for non-elderly

---

## 📁 Repository Structure

```
clinicalmind-hackathon/
├── agents/
│   ├── rxsafe/
│   │   ├── system_prompt.md
│   │   └── agent_code.py
│   ├── discharge_planner/
│   │   ├── system_prompt.md
│   │   └── agent_code.py
│   └── discharge_ready/
│       ├── system_prompt.md
│       └── agent_code.py
├── data/
│   ├── margaret_chen_fhir.json
│   └── sample_observations.json
├── ui/                             # Physician-facing Next.js UI (WIP)
├── LICENSE
└── README.md
```

---

## 🚀 Quick Start

```bash
git clone https://github.com/AIoOS-67/clinicalmind-hackathon.git
cd clinicalmind-hackathon
```

1. Create a free MongoDB Atlas M0 cluster on Google Cloud
2. Load FHIR patient data into the `clinicalmind` database
3. Enable MongoDB MCP Server and copy your API key
4. Go to [Vertex AI Agent Builder](https://console.cloud.google.com/agent-platform/studio)
5. Create agents using system prompts from `/agents/` — configure A2A chain
6. Send test case to DischargeReady agent

---

## 🏆 Hackathon Track

**Google Cloud Rapid Agent Hackathon 2026** — MongoDB Partner Track

**Why MongoDB:**
- FHIR patient records are document-oriented — perfect fit for MongoDB
- Atlas Vector Search enables semantic similarity queries across patient cohorts
- MongoDB MCP Server provides real-time FHIR data access without custom API code

**Judging criteria alignment:**
- ✅ **Technical Implementation**: 3-agent A2A chain, Google Search, MCP integration
- ✅ **Innovation**: First discharge-safety AI with full transparency chain + evidence citations
- ✅ **Completeness**: End-to-end working system tested with real clinical scenarios
- ✅ **Business Value**: $26B hospital readmission cost problem; directly addresses CMS penalties

---

## 👥 Team

**Ken Liao** — FoodyePay Technology, Inc.
- Email: kenliao@foodyepay.com

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

*Built with Google Cloud Agent Builder + Gemini 2.5 Pro + MongoDB Atlas*
*Hackathon submission: May 5 – June 11, 2026*
