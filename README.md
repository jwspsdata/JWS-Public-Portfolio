# JWS Public Portfolio

Public portfolio of AI work by **John W. Spangler** — methodology, applied systems design, production applications, and deployment artifacts.

**Live demo:**  
**[jws-ai-portfolio.streamlit.app](https://jws-ai-portfolio.streamlit.app/)**

*Source code is proprietary. Public writeups and deployment evidence are included where appropriate.*

---

## Portfolio Contents

### 1) APDLC: AI Product Development Life Cycle

[APDLC AI Product Development Life Cycle Summary - Public.md](APDLC%20AI%20Product%20Development%20Life%20Cycle%20Summary%20-%20Public.md)

A public overview of a proprietary framework for taking AI-enabled products from idea to durable operation.

**Highlights:**
- Six-phase lifecycle for AI-enabled products
- Explicit treatment of evaluation, governance, HITL, and cost
- AI-native reference architecture for agentic systems
- Risk-proportionate governance aligned to NIST AI RMF, ISO/IEC 42001, and OWASP Top 10 for LLM Applications
- Evaluation as a designed discipline: methods matched to how knowable the right answer is, evaluators checked rather than trusted, and evaluation carried into operation
- Cross-checked against comparable lifecycle standards and methods, including ISO/IEC 5338
- Companion thinking on AI-assisted, AI-directed, and AI-autonomous development through the **AADG**

### 2) Healthcare Payment Reconciliation Program

[Healthcare-Payment-Reconciliation-Program/](Healthcare-Payment-Reconciliation-Program/)

A program that matches synthetic insurer deposits to their remittances, flags what doesn't match, and tests where AI actually earns its place against free deterministic baselines. All data is synthetic.

**Live demo (replay only, synthetic data):** [hcpayreconcile.streamlit.app](https://hcpayreconcile.streamlit.app)

#### Program Summary

[PUBLIC_SUMMARY.md](Healthcare-Payment-Reconciliation-Program/PUBLIC_SUMMARY.md)

**Highlights:**
- Coded rules (exact, near-miss, grouping) do most of the matching at no per-item cost; people review everything the rules aren't trusted to decide
- Three narrow AI steps tested against free baselines, under an evaluation plan fixed before the test data existed
- AI verifier kept to spot-check trusted rules; AI match proposer and variance explainer switched off when they didn't beat the free baselines
- Analogue Cost Model gives each step a break-even point instead of an arbitrary accuracy target
- Integer-cent money, quarantine instead of dropping bad records, append-only audit log, and resistance to prompt injection planted in memo text
- Replays recorded model outputs, so the demo and CI need no API key and cost nothing to run

**Stack:** Python (uv), Pydantic, SQLite, Streamlit, Anthropic Python SDK (Message Batches), pytest, GitHub Actions, Synthea

#### AWS Migration Mock

[AWS_PUBLIC_SUMMARY.md](Healthcare-Payment-Reconciliation-Program/AWS_PUBLIC_SUMMARY.md)

What it would take to move the app onto an enterprise-style AWS platform. The infrastructure is written as code and checked, but not deployed.

**Highlights:**
- Eight AWS CDK stacks in Python, checked under the cdk-nag AwsSolutions and HIPAA Security rule packs, with 30 template tests and negative controls
- Claude on Amazon Bedrock, with guardrails that mask patient data, hold items with prompt injections, and require grounded explanations
- Investigation agent on Bedrock AgentCore that only proposes; people decide every item
- Aurora PostgreSQL with an append-only audit log plus an S3 Object Lock copy
- Agent accuracy raised from 14/20 to 17/20 (40/41 on held-out items) by changing what the agent could see, not by rewording the prompt

**Stack:** AWS CDK, Amazon Bedrock + AgentCore, Aurora PostgreSQL, S3, Docker, cdk-nag

### 3) Knowledge Curator: APDLC Engagement Case Study

[Knowledge Curator - APDLC Engagement Case Study - Public.md](Knowledge%20Curator%20-%20APDLC%20Engagement%20Case%20Study%20-%20Public.md)

A public case study showing the APDLC framework applied to a real agentic knowledge-management system.

**Highlights:**
- Automated Gmail → Claude → Obsidian workflow
- Four-agent design: Orchestrator, Ingest, Summarization, Filing
- Deterministic controls for sender validation and exception handling
- Full design artifact set across architecture, state, risk, governance, and build handoff
- Built and running in production; its Build findings reshaped APDLC's own Build phase

### 4) Presentation PDF Summarizer

[Presentation-PDF-summarizer/](Presentation-PDF-summarizer/)

A production-ready Streamlit application that uses Claude to generate structured summaries of presentation PDFs, including both single-deck analysis and cross-deck synthesis.

**Highlights:**
- 13 structured output fields per deck
- Cross-deck thematic synthesis with citation tracking
- Multiple Claude model options with token and cost visibility
- Vision fallback for difficult PDFs
- 86 unit tests and Streamlit Community Cloud deployment

**Stack:** Python 3.11+, Anthropic Claude API, PyMuPDF, Streamlit

### 5) AWS EC2 + Docker Deployment

[AWS-Deploy/](AWS-Deploy/)

Deployment artifacts and evidence for a **Time Series Forecast AI Copilot** deployed on AWS EC2 with Docker.

**Highlights:**
- EC2 + nginx + Streamlit container deployment
- Username/password authentication (bcrypt-hashed, role-based access)
- CSV/Excel ingest into SQLite
- Async LangGraph forecasting workflow with XGBoost
- 95% Gaussian prediction intervals (fixed z-score, not calibrated conformal prediction)

**Stack:** AWS EC2, Docker, Docker Compose, Nginx, Let's Encrypt, Streamlit, LangGraph, XGBoost, streamlit-authenticator

---

## What This Portfolio Demonstrates

- **AI product methodology** — framing, governance, evaluation, and lifecycle design
- **Agentic system architecture** — bounded agents, orchestration, controls, and HITL strategy
- **Applied AI delivery** — production-style applications and design-to-build readiness
- **Deployment capability** — cloud hosting, containerization, authentication, and runtime operations

---

## About

**John W. Spangler** — AI practice architect with a background spanning enterprise technology transformation across ITSM, Lean/Agile, and DevOps.

[linkedin.com/in/johnwspangler](https://www.linkedin.com/in/johnwspangler/)
