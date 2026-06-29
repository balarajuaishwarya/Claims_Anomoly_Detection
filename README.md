# 
**Clinical Decision Making and Pattern Recognition in Health Care**
**Agentic Generative AI for Treatment, Payment, and Operations (TPO)**

This repository contains all four required deliverables, built around a single proposed system: the
**Cotiviti Agentic Audit Copilot** — a two-node, human-in-the-loop (HITL)
pipeline for payment-integrity claims review.

## Contents

| File | Deliverable | Description |
|---|---|---|
| `Report.docx` | Written Report | 2-page report + bibliography (MLA), defining the topic, trends, opportunities/threats, and strategic options for Cotiviti |
| `Presentation.pptx` | Slide Presentation | summarizing the report and the POC architecture |
| `Claims_Anomaly_Detector.ipynb` | POC — Node 1 | Statistical trigger engine (Python/pandas): rolling z-score anomaly detection over synthetic claims data |
| `Agentic_Claims_Investigator.ipynb` | POC — Node 2 | Generative AI reasoning layer: a real, multi-step Claude API chain that investigates each flagged claim and produces an auditable recommendation |
| `Claims_Anomaly_Detector_POC.html` | Hackathon POC — interactive demo | Standalone browser demo of Node 1, no installation required |
| `requirements.txt` | — | Python dependencies for both notebooks |

## Architecture: how the two notebooks fit together

```
        Claim data
            │
            ▼
 ┌─────────────────────────┐
 │  NODE 1 — Trigger Engine │   Claims_Anomaly_Detector.ipynb
 │  rolling z-score         │   • no API key required
 │  anomaly detection       │   • runs instantly, fully deterministic
 └────────────┬─────────────┘
              │  flagged claims (|z| ≥ 2.0)
              ▼
 ┌──────────────────────────────────┐
 │  NODE 2 — Generative AI Reasoner  │   Agentic_Claims_Investigator.ipynb
 │  triage → evidence review →       │   • requires ANTHROPIC_API_KEY
 │  structured recommendation        │   • real Claude API calls
 └────────────────────────────────────┘
              │
              ▼
   Human investigator reviews the
   recommendation and makes the
   final payment decision
```

Node 1 answers **"is this claim statistically unusual?"** Node 2 answers
**"why, and what should an investigator do about it?"** Node 1's output
(`handoff_records`, built in its final section) is in the exact shape Node
2 expects to receive.

Neither node issues an autonomous payment decision. Both are designed to
hand a human analyst an audit-ready recommendation, consistent with the
report's Human-in-the-Loop (HITL) design principle.

## Running the POC

### Option A — interactive browser demo (no setup)

Open `Claims_Anomaly_Detector_POC.html` directly in any browser. Adjust the
provider and sensitivity threshold; the chart, summary stats, and
reason-code table recompute live. This is Node 1 only.

### Option B — Node 1 notebook (Python, no API key needed)

```bash
pip install -r requirements.txt
jupyter notebook Claims_Anomaly_Detector.ipynb
```

Run all cells top to bottom. Generates synthetic claims, computes rolling
statistics, flags anomalies, plots results, and prints the handoff payload
for Node 2.

### Option C — Node 2 notebook (requires a Claude API key)

Node 2 makes real calls to the Claude API, so it needs an
[Anthropic API key](https://console.anthropic.com):

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
pip install -r requirements.txt
jupyter notebook Agentic_Claims_Investigator.ipynb
```

Run all cells top to bottom. If no key is set, the notebook prints clear
setup instructions and stops rather than fabricating output — every
response in a successful run is a genuine model call, so wording will vary
slightly each time it's run.

This notebook uses a fixed 3-step reasoning chain (triage → evidence
review → structured recommendation) rather than autonomous tool-calling.
This is a deliberate scope choice for a fast hackathon POC: it keeps the
demo cheap and easy to follow end-to-end while still showing genuine
multi-step LLM reasoning. A production system would more likely give
Claude real tool-calling to decide which evidence to retrieve.

## Notes on the data

All claims data in both notebooks and the HTML demo is **synthetic**,
generated with a seeded random process to produce realistic-looking
billing patterns with a known set of injected anomalies. No real patient,
provider, or claims data is used anywhere in this repository.

## Citation note

DRG upcoding, referenced in the written report and the Node 1 notebook, is
a well-documented administrative fraud pattern; see Luo & Gallagher (2010)
in the report's bibliography for a peer-reviewed source. The synthetic data
in this POC mimics the *statistical shape* of an upcoding pattern (a
provider's billed amount drifting above its historical norm) but does not
carry real DRG/CPT codes.
