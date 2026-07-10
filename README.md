# JWS Public Portfolio

Public portfolio of AI work by **John W. Spangler** — frameworks, applications, deployment artifacts, and AI product methodology case studies.

**See live AI solutions in action:**
**<a href="https://ai-bootcamp-jws-portfolio.streamlit.app/" target="_blank">ai-bootcamp-jws-portfolio.streamlit.app</a>**

*(Source code is proprietary; detailed public documentation is included where available.)*

---

## Contents

### APDLC: AI Product Development Life Cycle

[APDLC AI Product Development Life Cycle Summary - Public.md](APDLC%20AI%20Product%20Development%20Life%20Cycle%20Summary%20-%20Public.md)

A public overview of a proprietary framework for taking AI-enabled products from idea to durable operation. It presents APDLC as a lightweight, risk-proportionate product lifecycle paired with an AI-native agentic reference architecture.

**What it covers:**

- A six-phase, explicitly non-linear lifecycle for AI-enabled products
- Human-in-the-loop strategy, evaluation, governance, and cost as first-class design concerns
- An AI-native agentic reference architecture for multi-agent systems
- Risk-proportionate governance aligned to NIST AI RMF, ISO/IEC 42001, and OWASP Top 10 for LLM Applications
- The companion **AI Autonomous Development Guide (AADG)**, which distinguishes runtime autonomy from development-process autonomy

---

### Knowledge Curator: APDLC Engagement Case Study

[Knowledge Curator - APDLC Engagement Case Study - Public.md](Knowledge%20Curator%20-%20APDLC%20Engagement%20Case%20Study%20-%20Public.md)

A public case study showing APDLC applied to a real agentic knowledge-management system. The engagement covers the completed Frame and Design phases for **Knowledge Curator**, an automated Gmail → Claude → Obsidian pipeline for curated research capture.

**What it covers:**

- A four-agent architecture: Orchestrator, Ingest, Summarization, and Filing
- Human-escalated control mode with deterministic sender validation and explicit exception routing
- Design artifacts spanning architecture, state, risk, escalation, cost, governance, and build handoff
- Five defined build workstreams and a build-ready implementation plan
- A concrete example of framework validation through production-oriented design work

---

### Presentation PDF Summarizer

[Presentation-PDF-summarizer/](Presentation-PDF-summarizer/)

A production-ready Streamlit web application that uses Claude (Anthropic) to generate structured summaries of presentation slide-deck PDFs. Supports single-deck analysis and cross-deck thematic synthesis.

**Key capabilities:**

- Extracts 13 structured fields per deck (thesis, problem, approach, evidence, risks, implications, etc.)
- Cross-deck thematic synthesis across 10 sections with citation tracking
- Three Claude model options (Haiku / Sonnet / Opus) with real-time token and cost tracking
- Vision-based fallback for PDFs where text extraction fails
- 32 unit tests; deployed on Streamlit Community Cloud

**Stack:** Python 3.11+, Anthropic Claude API, PyMuPDF, Streamlit

---

### AWS EC2 + Docker Deployment

[AWS-Deploy/](AWS-Deploy/)

Deployment artifacts and evidence for a **Time Series Forecast AI Copilot** running on AWS EC2 via Docker. A live solution is not currently available to avoid hosting costs.

**Architecture:** EC2 t3.micro → nginx (SSL termination) → Streamlit app container

**Stack:** AWS EC2, Docker + Docker Compose, Nginx, Let's Encrypt, Streamlit, LangGraph (async), XGBoost, streamlit-authenticator

**Features deployed:** username/password + Google/Microsoft OAuth, CSV/Excel upload to SQLite, demo Walmart dataset, async LangGraph forecast agent with XGBoost and 95% conformal confidence intervals

---

## About

**John W. Spangler** — AI practice architect with a background spanning enterprise IT transformation (ITSM, Lean/Agile, DevOps) across individual-contributor, management, and coaching roles.

[linkedin.com/in/johnwspangler](https://www.linkedin.com/in/johnwspangler/)
