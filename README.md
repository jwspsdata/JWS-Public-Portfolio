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
- Companion thinking on AI-assisted, AI-directed, and AI-autonomous development through the **AADG**

### 2) Knowledge Curator: APDLC Engagement Case Study

[Knowledge Curator - APDLC Engagement Case Study - Public.md](Knowledge%20Curator%20-%20APDLC%20Engagement%20Case%20Study%20-%20Public.md)

A public case study showing the APDLC framework applied to a real agentic knowledge-management system.

**Highlights:**
- Automated Gmail → Claude → Obsidian workflow
- Four-agent design: Orchestrator, Ingest, Summarization, Filing
- Deterministic controls for sender validation and exception handling
- Full design artifact set across architecture, state, risk, governance, and build handoff
- Build-ready workstream plan for implementation

### 3) Presentation PDF Summarizer

[Presentation-PDF-summarizer/](Presentation-PDF-summarizer/)

A production-ready Streamlit application that uses Claude to generate structured summaries of presentation PDFs, including both single-deck analysis and cross-deck synthesis.

**Highlights:**
- 13 structured output fields per deck
- Cross-deck thematic synthesis with citation tracking
- Multiple Claude model options with token and cost visibility
- Vision fallback for difficult PDFs
- 32 unit tests and Streamlit Community Cloud deployment

**Stack:** Python 3.11+, Anthropic Claude API, PyMuPDF, Streamlit

### 4) AWS EC2 + Docker Deployment

[AWS-Deploy/](AWS-Deploy/)

Deployment artifacts and evidence for a **Time Series Forecast AI Copilot** deployed on AWS EC2 with Docker.

**Highlights:**
- EC2 + nginx + Streamlit container deployment
- Username/password plus Google/Microsoft OAuth
- CSV/Excel ingest into SQLite
- Async LangGraph forecasting workflow with XGBoost
- 95% conformal confidence intervals

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
