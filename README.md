# BSA PO Box Address Remediation — Automation Framework

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![BSA Compliant Architecture](https://img.shields.io/badge/BSA-Compliant%20Architecture-green)]()
[![Azure AI Foundry](https://img.shields.io/badge/Azure-AI%20Foundry-0078d4)]()

A BSA-compliant workflow automation framework for remediating PO Box and missing address records surfaced in Verafin alerts. Built for community banks operating under Bank Secrecy Act (BSA) Customer Identification Program (CIP) requirements.

---

## The Problem

When a Verafin alert surfaces a customer with only a PO Box or no address on file, the bank has a CIP gap under 31 CFR 1020.220. The traditional manual process requires an analyst to:

- Search ImageCenter for customer documents with a physical address
- Run actDataScout (county-specific public records)
- Search Arkansas Court Connect by customer name and DOB
- Write a standardized Verafin compliance note
- Update the core banking system (Bankway CIF)

This process is time-consuming, inconsistent, and creates an incomplete audit trail.

---

## The Solution

An automated three-tier workflow that resolves high-confidence records without analyst involvement, routes exceptions for human review, and generates a daily summary for the BSA Officer.

| Tier | Who handles it | What happens |
|------|---------------|-------------|
| 1 — Auto-resolve | No one (fully automated) | High-confidence matches: Verafin note submitted, CIF updated, audit log written |
| 2 — Exception queue | BSA Analyst | Low-confidence matches routed for analyst confirmation |
| 3 — Officer oversight | BSA Officer | No-result records escalated; daily summary reviewed (~5 min) |

**Estimated outcome:** ~80% reduction in analyst time with a stronger, more consistent audit trail.

---

## Architecture

```
Verafin CSV Export
       │
       ▼
┌─────────────────┐
│  Ingest + Filter │  (Power Automate / Azure Logic Apps)
│  PO Box records  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Bankway CIF    │  Check if physical address already exists
│  Lookup         │
└────────┬────────┘
         │
    ┌────┴────┐
    │ Found?  │
    └────┬────┘
   YES   │   NO
    ▼         ▼
Auto-     ┌──────────────────┐
resolve   │ Multi-source     │  actDataScout + ARCourts
          │ Search           │  (by customer name + DOB)
          └────────┬─────────┘
                   │
                   ▼
          ┌────────────────┐
          │ Azure AI       │  Model Catalog: Claude, GPT-4o,
          │ Foundry        │  Llama, Phi-3 — swap without
          │ Evaluation     │  changing workflow code
          └────────┬───────┘
                   │
         ┌─────────┼──────────┐
       HIGH       LOW        NONE
         │         │          │
    Auto-      Analyst    BSA Officer
    resolve    queue      escalation
         │         │          │
         └─────────┴──────────┘
                   │
                   ▼
         ┌─────────────────┐
         │  Daily Summary  │  Email to BSA Officer
         │  Report         │  resolved / exceptions / no-result
         └─────────────────┘
```

---

## Tech Stack

| Component | Power Automate Option | Azure Logic Apps Option |
|-----------|----------------------|------------------------|
| Orchestration | Microsoft Power Automate | Azure Logic Apps (consumption plan) |
| File storage | SharePoint | Azure Blob Storage |
| Record parsing | Power Automate expressions | Azure Functions |
| CIF lookup | HTTP connector → Bankway API | HTTP action + Azure Key Vault |
| Public records search | HTTP connector | HTTP action |
| AI evaluation | Azure AI Foundry (HTTP) | Azure AI Foundry (native) |
| Audit log | SharePoint list | Azure SQL / Table Storage |
| Notifications | Teams + Outlook | Teams + Office 365 |
| Secret management | Environment variables | Azure Key Vault |

### AI Model Options (Azure AI Foundry Model Catalog)

Models can be swapped without changing workflow code:

- **Claude Sonnet** (Anthropic via Azure Marketplace) — recommended for structured evaluation tasks
- **GPT-4o mini** (Azure OpenAI) — cost-effective, fast classification
- **Phi-3.5** (Microsoft) — runs fully in-tenant, lowest cost
- **Llama 3.1** (Meta) — open-source option

---

## Repository Structure

```
bsa-po-box-remediation/
│
├── README.md                          # This file
│
├── diagrams/
│   ├── power-automate-workflow.svg    # PA workflow mockup
│   ├── azure-logic-apps-workflow.svg  # Azure workflow mockup
│   └── automation-architecture.svg   # Three-tier architecture diagram
│
├── templates/
│   ├── verafin-note-found.md          # Verafin note template — address found
│   ├── verafin-note-not-found.md      # Verafin note template — no address found
│   └── audit-log-schema.md            # Azure SQL / SharePoint audit log fields
│
├── workflow/
│   ├── power-automate/
│   │   └── README.md                  # PA setup and connector guide
│   └── azure-logic-apps/
│       └── README.md                  # Azure Logic Apps setup guide
│
├── compliance/
│   ├── regulatory-basis.md            # BSA/CIP regulatory citations
│   └── confidence-threshold-guide.md  # How to set and document the AI threshold
│
└── LICENSE
```

---

## BSA Compliance Notes

This framework is designed to satisfy the following regulatory requirements:

- **31 CFR 1020.220 (CIP)** — Obtains and documents a verifiable physical address for PO Box records
- **31 CFR 1020.210 (CDD/AML Program)** — Maintains accurate customer information for ongoing transaction monitoring
- **31 CFR 1020.210(b) (Internal Controls)** — Written, consistently applied automated procedure demonstrates operating internal controls
- **31 CFR 1010.430 (Record Retention)** — All decisions logged with 5-year retention

**Critical compliance requirements for implementation:**
1. BSA Officer must approve and sign the updated SOP before go-live
2. Every automated decision must be logged (timestamp, source, confidence score, action)
3. Negative searches must be documented — not just positive results
4. Unresolved records must escalate to officer, not silently close
5. BSA Officer must conduct periodic spot-checks of auto-resolved records

> **Note:** This repository contains sanitized documentation and templates only. No customer data, proprietary system credentials, or institution-specific configurations are included.

---

## Getting Started

### Prerequisites

- Access to Verafin (CSV export capability)
- Bankway API access (request from core vendor)
- Verafin API access (request from Verafin support)
- actDataScout subscription with API access confirmed
- Azure subscription (for Logic Apps option) or M365 license (for Power Automate option)
- Azure AI Foundry endpoint deployed

### Setup Overview

1. **Request API access** from Bankway and Verafin vendors
2. **Deploy Azure AI Foundry** endpoint and select model from Model Catalog
3. **Configure workflow** using the appropriate platform guide in `/workflow/`
4. **Set confidence threshold** using guidance in `/compliance/confidence-threshold-guide.md`
5. **Update SOP** and obtain BSA Officer sign-off before go-live
6. **Run in shadow mode** for one cycle alongside manual process to validate output
7. **Go live** and monitor via daily summary report

---

## Author

**Lauren Bolin** — BSA Compliance Specialist  
[LinkedIn](https://linkedin.com/in/) · [GitHub](https://github.com/)

---

## License

MIT License — see [LICENSE](LICENSE) for details.

This project is provided for educational and reference purposes. Nothing in this repository constitutes legal or regulatory advice. Always consult your compliance team and legal counsel before implementing automated BSA processes.
