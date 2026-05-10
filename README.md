[README_DealStructuring.md](https://github.com/user-attachments/files/27426419/README_DealStructuring.md)
# 🤝 M&A Deal Structuring AI Assistant

An agentic AI system that automates the end-to-end analysis of mergers and acquisitions (M&A) deals. Built with **LangGraph** and powered by **GPT-4o** and **Flan-T5**, the system runs a sequential pipeline of six specialized agents — each handling a distinct layer of deal analysis — to produce a full structured deal recommendation from raw financial inputs.

---

## Overview

M&A deal structuring normally requires teams of investment bankers and financial analysts to work through financial modeling, capital structure decisions, stakeholder mapping, regulatory risk, and negotiation strategy in sequence. This project automates that workflow using an agentic graph architecture where each node represents a specialized reasoning agent operating on a shared deal state.

Given the financials of both companies and a proposed deal price, the system produces a complete deal summary covering accretion/dilution, optimal financing mix, management retention risk, regulatory approvals required, and recommended negotiation posture.

---

## Agent Pipeline

```
Financial Model → Accretion/Dilution → Capital Structure → Stakeholders → Regulatory → Negotiation
```

| Agent | Function | Output |
|---|---|---|
| **Financial Model Agent** | Combines revenue and net income; computes combined and acquirer EPS | `combined_revenue`, `combined_eps`, `acquirer_eps` |
| **Accretion/Dilution Agent** | Compares combined EPS to acquirer standalone EPS | `accretion_dilution_result` (Accretive / Dilutive) |
| **Capital Structure Agent** | Recommends cash vs. stock financing mix based on A/D result | `optimal_mix`, `deal_timeline_months` |
| **Stakeholders Analyst Agent** | Identifies payout preference conflicts, flags retention risk for key management | `key_management_to_retain`, `incentive_risk`, `potential_conflicts` |
| **Regulatory Compliance Agent** | Assesses antitrust exposure and determines required regulatory approvals | `antitrust_risk`, `approvals_required` |
| **Negotiation Strategy Agent** | Sets negotiation posture and fallback based on acquirer priorities and financial flexibility | `negotiation_strategy`, `fallback_option` |

---

## Sample Deal Summary Output

```
--- DEAL SUMMARY ---
Accretion/Dilution Result:     Dilutive
Optimal Capital Mix:           70% stock / 30% cash
Deal Timeline (Months):        6
Key Management to Retain:      CEO, CFO, CTO
Incentive Risk:                Low
Potential Stakeholder Conflicts: Mixed payout preferences
Antitrust Risk:                High
Regulatory Approvals Required: FTC, SEC
Negotiation Strategy:          Collaborative
Fallback Option:               Use stock instead of cash
```

---

## Deal State Schema

The pipeline operates on a shared `DealState` Pydantic model that carries all inputs and agent outputs through the graph:

| Field | Type | Description |
|---|---|---|
| `target_financials` | Dict | Target company revenue, net income, EPS |
| `acquirer_financials` | Dict | Acquirer company revenue, net income, EPS |
| `deal_price` | Float | Proposed acquisition price |
| `management_list` | List | Key executives at the target |
| `shareholder_data` | Dict | Shareholder payout preferences (cash/stock) |
| `employee_incentive_programs` | Dict | RSUs, options, or other retention programs |
| `industries_involved` | List | Industries of both parties (used for antitrust analysis) |
| `countries_involved` | List | Jurisdictions involved (drives regulatory routing) |
| `party_priorities` | Dict | Acquirer negotiation priorities (e.g., speed, control) |
| `financial_flexibility` | Float | Acquirer's available cash for deal financing |
| `regulatory_constraints` | List | Known regulatory risks going into the deal |

---

## Capital Structure Logic

| Condition | Recommended Mix |
|---|---|
| Deal is **Dilutive** | 70% stock / 30% cash — preserve cash, use equity to close |
| Deal is **Accretive** | 50% stock / 50% cash — balanced structure |

## Regulatory Routing Logic

| Condition | Output |
|---|---|
| Technology sector involved | Antitrust risk = **High** |
| USA in countries involved | Approvals required: **FTC, SEC** |
| Non-US deal | Approvals required: **EU Commission** |

## Negotiation Logic

| Condition | Output |
|---|---|
| Acquirer prioritizes speed | Strategy = **Collaborative** |
| Acquirer does not prioritize speed | Strategy = **Aggressive** |
| Financial flexibility < $100M | Fallback = Use stock instead of cash |
| Financial flexibility ≥ $100M | Fallback = Offer early close premium |

---

## Getting Started

**1. Clone the repo**
```bash
git clone https://github.com/your-username/deal-structuring-ai.git
cd deal-structuring-ai
```

**2. Install dependencies**
```bash
pip install langgraph langchain openai pydantic huggingface_hub transformers
```

**3. Set API keys**
```bash
export OPENAI_API_KEY=your_openai_key
export HUGGINGFACEHUB_API_TOKEN=your_hf_token
```

**4. Run the notebook**
```bash
jupyter notebook Deal_Structuring_AI_Assistant__2_.ipynb
```

The final cell will prompt you for both companies' financials and deal price, then run the full pipeline and print the deal summary.

---

## Input Prompts

```
Enter target company revenue:
Enter target company net income:
Enter target company EPS:

Enter acquirer company revenue:
Enter acquirer company net income:
Enter acquirer company EPS:

Enter deal price:
```

All remaining deal parameters (stakeholder data, industries, countries, priorities, financial flexibility) are configurable directly in the `dummy_input` dictionary in the notebook.

---

## Tech Stack

| Library | Role |
|---|---|
| `LangGraph` | Stateful agent graph — manages node execution and shared state |
| `LangChain` | LLM abstraction layer |
| `OpenAI GPT-4o` | Primary LLM |
| `Flan-T5-Large` (HuggingFace) | Secondary LLM (text-to-text generation) |
| `Pydantic` | `DealState` schema validation and type enforcement |

---

## Project Structure

```
deal-structuring-ai/
├── Deal_Structuring_AI_Assistant__2_.ipynb   # Full agentic pipeline
└── README.md
```

---

## Potential Extensions

- **Dynamic conditional branching** — add LangGraph conditional edges so the pipeline routes differently based on intermediate outputs (e.g., skip regulatory agent for small domestic deals)
- **LLM-generated narrative** — use GPT-4o to produce a plain-English deal memo summarizing all agent outputs, formatted like a banker's recommendation letter
- **Sensitivity analysis** — sweep deal price or EPS assumptions to show accretion/dilution breakeven thresholds
- **Comparable transactions** — integrate a database or web search agent to pull comparable M&A deal multiples for valuation benchmarking
---

## Context

This project was developed as part of an honors thesis exploring the integration of AI and M&A deal structuring. The system demonstrates how agentic architectures can compress multi-analyst workflows into a single automated pipeline, with each agent mirroring a specialized function typically performed by a member of an investment banking deal team.

---

## License

MIT License. See `LICENSE` for details.
