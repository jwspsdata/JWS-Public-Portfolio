# JWS Public Portfolio

Public portfolio of AI work by **John W. Spangler** — frameworks, applications, and deployment artifacts.

Explore these solutions:
[ai-bootcamp-jws-portfolio.streamlit.app](https://ai-bootcamp-jws-portfolio.streamlit.app/)

---

## Contents

### APDLC: AI Product Development Life Cycle

[APDLC AI Product Development Life Cycle Summary - Public.md](APDLC%20AI%20Product%20Development%20Life%20Cycle%20Summary%20-%20Public.md)

An overview of a self-developed proprietary methodology framework for taking AI-enabled products from idea to durable production. See markdown file for complete details.

**What it covers:**

- A six-phase lifecycle that scales from internal prototype to regulated production system
- An AI-native agentic reference architecture for multi-agent systems 
- Risk-proportionate governance mapped to NIST AI RMF, ISO/IEC 42001, and OWASP LLM Top 10

---

### Presentation PDF Summarizer

[Presentation-PDF-summarizer/](Presentation-PDF-summarizer/)

A production-ready Streamlit web application that uses Claude (Anthropic) to generate structured summaries of presentation slide-deck PDFs. Supports single-deck analysis and cross-deck thematic synthesis. This folder contains a project summary, skills overview, example outputs (deck_summaries_first3.md), and a summary deck I presented to my employer on the inaugural IT Revolution Enterprise AI Summit (EAIS26_AI_deck_v7.pptx).

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

Deployment artifacts and evidence for a **Time Series Forecast AI Copilot** running on AWS EC2 via Docker.

**Architecture:** EC2 t3.micro → nginx (SSL termination) → Streamlit app container

**Stack:** AWS EC2, Docker + Docker Compose, Nginx, Let's Encrypt, Streamlit, LangGraph (async), XGBoost, streamlit-authenticator

**Features deployed:** username/password + Google/Microsoft OAuth, CSV/Excel upload to SQLite, demo Walmart dataset, async LangGraph forecast agent with XGBoost and 95% conformal confidence intervals

---

## About

**John W. Spangler** — AI practice architect with a background spanning enterprise IT transformation (ITSM, Lean/Agile, DevOps) across individual-contributor, management, and coaching roles.

[linkedin.com/in/johnwspangler](https://www.linkedin.com/in/johnwspangler/)
