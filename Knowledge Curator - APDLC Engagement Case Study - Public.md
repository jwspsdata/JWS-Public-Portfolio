# Knowledge Curator

## An APDLC Engagement Case Study — Public Overview

**Status**: Design Phase Complete | Build Phase Pending  
**Tier**: Internal-Production  
**Type**: Agentic Knowledge Management System  
**Framework**: AI Product Development Life Cycle (APDLC) v7  
**Scope**: Capabilities overview — facilitation guides, artifact templates, and reference architecture specifics are proprietary and not included here

---

## Executive Summary

**Knowledge Curator** is an agentic knowledge management system built to solve a problem that anyone working at the edge of AI practice knows well: the field moves faster than any individual can track. New models, patterns, architectures, and tooling emerge continuously — and the volume of credible, high-signal material is relentless. Without a deliberate system for capturing and organizing it, knowledge accumulates as unread links, forgotten tabs, and email backlogs that never get reviewed.

Knowledge Curator automates that capture loop. It monitors personal Gmail inboxes, fetches and summarizes linked content via Claude, and files curated notes into a structured, isolated Obsidian vault — kept separate from 15+ years of accumulated career knowledge so that emerging AI practice has its own dedicated, searchable home. The longer-term goal: connect that vault to an LLM as a design and challenge partner. The immediate goal: stop losing good material to inbox entropy.

There is also a deliberate meta-point here. APDLC — the framework being validated through this engagement — exists in part because the pace of AI practice creates real delivery risk: teams are torn between building and keeping up. Knowledge Curator is a tool for staying current with the very problem APDLC addresses. The framework's first production engagement is itself a tool for staying current.

This document covers the Frame and Design phases for Knowledge Curator, completed in full under APDLC. It demonstrates what the framework produces, how it handled a real agentic system design, and what it revealed about its own gaps.

**At a glance**:

- **Problem**: Manually curating content across multiple inboxes — time-consuming, inconsistent, error-prone
- **Solution**: Automated async pipeline (Gmail → Claude → Obsidian vault)
- **Scale**: Single user; 3 configurable Gmail inboxes; personal tool
- **Design artifacts produced**: Complete artifact set, review-complete, build-ready

---

## The Problem Being Solved

The owner maintains three Gmail inboxes organized by knowledge domain (AI, Data, Quantitative Science). When discovering valuable content online, they forward themselves curated emails containing a URL, notes, or both.

**Pain points without automation**:

- Email backlogs accumulate without structured review
- Content is lost or forgotten if not manually curated into the knowledge vault
- No consistent summarization format
- Manual filing into Obsidian is time-consuming and irregular

**What "solved" looks like**: A background process that runs twice daily, processes the backlog, files clean markdown notes into the right vault folders, routes anything unusual for owner review, and sends a summary email with everything that was done — including token cost.

---

## Solution Architecture

### System Overview

```text
Gmail Inboxes (3)
    ↓ [Periodic trigger — 9am / 5pm daily]
Orchestrator Agent
    ├→ Ingest Agent        (email fetch, sender validation)
    ├→ Summarization Agent (URL fetch + Claude summarize)
    ├→ Filing Agent        (folder selection, dedup, vault write)
    └→ Notification Agent  (summary email dispatch)
    ↓
Obsidian Vault (curated-vault/)
    ├── AI/
    ├── Data/
    ├── QuantScience/
    ├── uncategorized/
    └── FLAGGED/
```

### Four Agents, Bounded Responsibilities

**Orchestrator** — coordinates the overall workflow; reads persisted state (latest Sent Date per inbox); dispatches ingest per configured inbox; aggregates results and triggers notification.

**Ingest** — fetches emails from Gmail API; validates sender against an exact-match allowlist; extracts URL and body text; routes to Summarization or Filing.

**Summarization** — fetches URL content via `httpx` + `trafilatura`; calls Claude Sonnet for a grounded summary (source-only; no editorializing); tags output with limit indicators (`content-truncated`, `limited-access`).

**Filing** — scans vault for duplicate detection; selects or creates topic folder; falls back to `uncategorized` when topic is unclear; routes sensitive/NSFW content to `FLAGGED`; writes markdown; handles near-duplicate merging (append-only).

### Control Mode & Human-in-the-Loop

**Overall Posture**: Human-Escalated — the system acts autonomously within defined boundaries; exceptions are routed to review folders; the owner reviews at their own pace; no blocking pre-approvals.

| Action | Mode | Notes |
| -------- | ------ | ------- |
| Sender validation | Autonomous | Deterministic allowlist — no AI judgment |
| Email parsing | Autonomous | No execution of content |
| URL fetch & summarize | Autonomous | Grounded; source-only |
| Duplicate detection | Autonomous | Vault-based; silent discard on true duplicate |
| Folder selection / creation | Autonomous | For existing and new topics |
| Sensitive / NSFW routing | Human-Escalated | Routes to FLAGGED; owner reviews |
| Uncategorized content | Human-Escalated | Routes to uncategorized; owner reviews |

---

## APDLC in Practice: Frame → Design → Build-Ready

### Frame Phase

**Input**: Problem description, constraints (personal tool, security-sensitive), stakeholder context (single user/operator)

**Output**: A structured framing document covering problem statement, success criteria, data posture, risk assumptions, HITL expectations, and autonomy posture — complete enough that the Design phase could begin without clarifying questions.

**Seven measurable success criteria established**:

1. Curated emails processed into searchable Obsidian markdown with key takeaways
2. Non-allowlist emails deterministically ignored — no AI judgment in sender validation
3. Content filed consistently (topical folder, new folder, or uncategorized)
4. All output files tagged `ai-curated` for searchability
5. One summary email per inbox per run (one bullet per file)
6. True duplicates silently ignored; near-duplicates merged in-place
7. Sensitive/NSFW content routed to FLAGGED — no silent discard

### Design Phase: Artifact Set Produced

The Design phase produced a complete, review-ready artifact set — each artifact with a defined owner, scope, and acceptance criteria. The set spans six categories:

- **Problem framing** — problem statement, stakeholders, success criteria, data posture, risk assumptions, and autonomy posture
- **System architecture** — agent contracts, tool contracts, capability map, and orchestration graph
- **Data and state** — runtime data structures, state schema, and config schema
- **Risk and escalation** — failure mode catalog with detection and handling for every identified failure, escalation policy, and resume/recovery policy
- **Cost and governance** — cost model with per-item token budgets and monthly estimates, permission matrix (tool × agent × scope), trigger map, and human approval policy
- **Build handoff** — workstream map with parallelization order, artifact review matrix, and project README

**All artifacts are complete, reviewed, and committed to the repository.** Build can begin immediately.

---

## Key Design Decisions

### 1. Deterministic Sender Allowlist

**Decision**: Exact-match sender address validation — no AI judgment.

**Rationale**: Email injection and prompt injection are real threats. A deterministic allowlist is auditable, testable, and catches spoofed or lookalike addresses without LLM latency or cost overhead. Security posture is never traded for convenience here.

### 2. Grounded Summarization

**Decision**: Claude Sonnet summarizes only what is in the source — no inference, no editorializing.

**Rationale**: This is a curation tool, not an interpretation tool. Hallucination risk is mitigated by constraining the model to the extracted text. Owner values fidelity to source over augmentation.

### 3. Config-Driven Model Selection

**Decision**: Model identifier and pricing rates live in config — model rotation requires a config change, not a code change.

**Rationale**: The Claude model landscape evolves. Hardcoding model IDs and pricing constants into application code makes routine updates a code-change event. Externalizing them reduces friction and prevents stale pricing calculations.

### 4. In-Product Cost Surfacing

**Decision**: Token usage and estimated cost are included in the per-inbox summary email — not dashboards or external APM.

**Rationale**: This is a personal tool without cloud monitoring. The summary email is the primary operator-facing output. Cost transparency there means the owner always knows what the system consumed, supports future optimization decisions, and requires no additional tooling.

### 5. No Silent Discard

**Decision**: Anomalous content routes to FLAGGED. Nothing is silently discarded except true duplicates and paywalled URLs with no extractable content.

**Rationale**: Owner needs visibility into what the system is doing. Routing to FLAGGED is explicit — owner can review anytime. Trust is built by making the system's behavior observable, not opaque.

### 6. Append-Only Near-Duplicate Merge

**Decision**: Near-duplicate content (same topic, different source) appends to an existing file rather than creating a new one, tagged `[merged]`.

**Rationale**: Creating a new file for every related piece of content fragments the vault. Merging preserves multiple perspectives on the same topic in one place. Append-only ensures no prior content is ever overwritten. The `[merged]` tag makes origin transparent.

### 7. Deterministic Resumption via Sent Date

**Decision**: State is persisted as the latest Sent Date per mailbox — manually editable for controlled re-testing.

**Rationale**: The simplest resumption mechanism sufficient for this use case. Human-readable for debugging, manually editable for re-running against specific date ranges, and deterministic — no email is ever processed twice under normal operation, and recovery after a crash requires only re-reading state from the file.

---

## Technology Stack

| Component | Technology | Rationale |
| ----------- | ----------- | ----------- |
| **Runtime** | Python 3.10+ | Locally executed; no cloud infrastructure |
| **LLM** | Claude Sonnet (config-driven) | Quality/cost balance; sufficient for blog and academic content |
| **Email** | Gmail API + google-auth-oauthlib | OAuth token management included |
| **URL Fetch** | httpx | Async-ready; timeout control; retry logic |
| **Content Extraction** | trafilatura | HTML cleanup; boilerplate removal |
| **Secrets** | python-dotenv | .env-based; excluded from git |
| **Knowledge Vault** | Obsidian (local) | Owner's existing knowledge base; Git sync external |
| **Orchestration** | Python (form TBD in Build) | Decision deferred: engineered code, durable-execution, or graph framework |

---

## Build Readiness

### 5 Workstreams with Defined Parallelization

**WS1: Infrastructure & State** *(critical path; must complete first)*  
Config schema, state persistence, secrets management, error logging, crash recovery.

**WS2: Gmail Ingest** *(parallel with WS3, WS4)*  
Gmail OAuth flow, email fetching per mailbox, sender validation, URL/body extraction.

**WS3: Content Summarization Pipeline** *(parallel with WS2, WS4)*  
URL fetching with cap/timeout/retries, HTML extraction, content quality checks, Claude API integration, tag generation.

**WS4: Vault Filing** *(parallel with WS2, WS3)*  
Vault structure scanning, duplicate/near-duplicate detection, folder selection, FLAGGED routing, markdown generation, merge-in-place.

**WS5: Orchestration & Notification** *(depends on WS1–4)*  
Orchestrator coordination, run loop, summary email generation (with token/cost), non-reprocessing flag, external trigger integration.

---

## What This Demonstrates

### Capability: Systematic Agentic System Design

The design decomposes a real agentic workflow into exactly four bounded agents, eleven tools, and five parallel build workstreams — no more, no less. Each agent has a single responsibility. Each tool has explicit input/output contracts, failure modes, and permission scope. Every path through the system has a defined destination.

This level of decomposition requires architectural judgment (knowing where to draw boundaries), completeness verification (ensuring nothing falls through), and pragmatism (stopping at four agents rather than over-fragmenting).

### Capability: Security-Conscious Design at Personal Scale

A personal knowledge management tool might seem low-stakes from a security perspective. This design disagrees: sender validation is deterministic and auditable; untrusted content is never executed; prompt injection is treated as a real risk; and the escalation policy ensures nothing is silently discarded.

The design demonstrates that security-first thinking doesn't require complexity — it requires explicit threat modeling and deterministic controls where they matter.

### Capability: HITL Strategy as First-Class Concern

Human-in-the-loop strategy is defined per action before anything is built. The system operates in human-escalated mode: autonomous for the normal path, explicit escalation for the exception path. The owner reviews at their own pace with no blocking gates. "Human-in-the-loop" does not mean "human blocks every step" — it means human authority is designed, not defaulted.

### Capability: Cost as a Design Concern

Cost is not post-hoc. Token budgets are defined per item, monthly estimates are modeled, pricing constants are externalized to config, and cost signals flow to the operator through the product's own output. For a personal tool with no external APM, this is the right approach — and it required a design decision, not just implementation.

### Capability: Framework Improvement via Real Engagement

The framework's first production engagement surfaced seven improvements. A framework that enables work is good. A framework that enables work and reveals its own gaps through real use is better — because those gaps are systematic, not incidental, and the improvements belong in the methodology, not in project-specific workarounds.

This is the kind of feedback loop that makes methodology authorship credible: structured dry-runs find what breaks; real engagements find what was missing.

---

## What Is Not in This Document

The operational details of the APDLC framework — facilitation guides, artifact templates, decision logic, and reference architecture specifics — are proprietary and not published here. This case study is intended to demonstrate what the framework produces and how it handled a real engagement, not to reproduce how it works internally.

For discussion of the framework, engagements, or licensing, contact the author directly.

---

## About the Author

**John W. Spangler** is an AI practice architect who developed the APDLC framework including structured dry-run engagements and now production use. His background spans the major enterprise technology transformations of the past two decades — enterprise IT service management, Lean/Agile delivery, and DevOps adoption — across individual-contributor, management, and coaching roles.

Full background: [linkedin.com/in/johnwspangler](https://www.linkedin.com/in/johnwspangler/)

---

*Author: John W. Spangler*  
*Version: 1.0 (June 2026)*  
*Engagement Status: Design Complete | Build Pending*  
*Framework: APDLC v7*
