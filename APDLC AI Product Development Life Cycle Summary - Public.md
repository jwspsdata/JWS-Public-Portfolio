# APDLC: AI Product Development Life Cycle
## A Framework for AI-Enabled Products — Public Overview

**Current Version**: v9  
**Status**: Actively developed; first production engagement underway — Frame and Design complete, Build next  
**Type**: Proprietary methodology framework — this document is a capabilities overview only

---

## Executive Summary

- **Why it matters** — most AI investment still stalls before real ROI; pilots don't reach durable production. AI practices are evolving so rapidly, teams are torn between delivery and keeping up. APDLC exists to close these gaps.
- **What it is** — a lightweight, risk-proportionate lifecycle and agentic reference architecture for taking AI-enabled products from idea to durable operation.
- **Where it sits** — the product "outer loop" above the increasingly AI-automated SDLC; it ties together existing standards, patterns, and tools rather than replacing them.
- **How it governs** — governance scales with assessed risk (enough for a regulated system, not theater for a prototype) and maps to NIST AI RMF, ISO/IEC 42001, and the OWASP LLM Top 10.
- **Where it stands** — v9, refined through dry runs; first real engagement (Knowledge Curator) has completed Frame and Design.
- **Who built it** — an architect who has worked through every major tech transformation of the past two decades, where the human and organizational dimension is the recurring blind spot.

---

## What Is This?

APDLC is a framework I developed around a problem that's now widely reported: companies are struggling to turn AI investment into real ROI. Pilots stall, and systems that do ship often don't hold up in production. In addition, the staggering pace and volume of leading AI practices overwhelms even the most talented delivery teams. The model is rarely the hard part; the work around it is: governance, evaluation, human oversight, operational cost, and the fact that AI systems drift after launch. That work is what APDLC organizes.

APDLC gives that shape a name and a structure. It has been developed across nine iterations and pressure-tested through structured dry-run engagements — realistic scenarios run end-to-end to find where it breaks before a real build depends on it. Its first production engagement has completed the Frame and Design phases, with real artifacts, and is now heading into build.

This framework is shaped by a particular vantage point. I have worked through the major enterprise technology transformations of the past two decades — the web, virtualization, cloud, Agile, Lean IT, and DevOps — as an individual contributor, a manager, and a coach. Every one of those waves under-invested in the human and organizational dimension and over-invested in the technology itself. AI is repeating the pattern. APDLC is built by someone who has lived and learned through these major Technology transformations.

---

## The Core Problem I Was Solving

Building AI-enabled products is not a software engineering problem dressed up with different nouns. It requires a different lifecycle model because:

- **AI systems are non-deterministic** — their behavior must be continuously evaluated, not just tested at ship time
- **Risk evolves** — what was safe to automate at launch may not be safe six months later as usage patterns change
- **Human authority is a design decision, not a default** — teams that don't think about this explicitly end up with systems that either automate too much (unsafe) or too little (no value)
- **The "build" decision is multi-dimensional** — buy vs. build, which model, which orchestration form, what evaluation harness — and these choices interact
- **Agentic systems are not just "more AI"** — they introduce coordination problems, failure cascade risks, and audit requirements that don't exist in simpler LLM integrations
- **The field is evolving faster than most teams can track** — new models, patterns, and tooling emerge continuously. Without a structured foundation, teams burn energy on "what are we missing?" and high-stakes concerns get addressed inconsistently or not at all — not from negligence, but because nothing made the right path the easy one to take.
- **Overweight governance makes things worse, not better** — process disproportionate to actual risk gets ignored or worked around. Teams quickly spot which steps are theater and route around them, leaving the overhead without the protection.

A structured framework removes that friction: when the standing concerns are already encoded — risk, governance, evaluation, human control, cost — teams can put their full attention on the problem they're actually solving. What was missing was never good practice; it was something to tie it together into one coherent path from idea to operation. That integration is what APDLC is.

---

## How It Is Structured

APDLC has two layers that address different audiences and altitudes.

**Layer 1: The AI Product Lifecycle**

A six-phase lifecycle — from problem framing through continuous improvement — designed to be domain-agnostic and tier-scalable. The same framework applies to an internal prototype and a regulated, customer-facing production system; what changes is the depth of rigor at each phase. Domain-agnostic frameworks risk being too abstract to act on; APDLC avoids that by keeping the *structure* general while forcing each engagement to commit concrete, domain-specific decisions into its artifacts.

Key design decisions I made here:
- The lifecycle is **explicitly non-linear**. Evaluation findings should be able to trigger redesign; operational learnings should be able to trigger re-framing. Frameworks that force linearity cause teams to ignore evidence.
- **Human-in-the-loop strategy is a first-class phase concern**, not an afterthought. The framework forces teams to define their autonomy posture and escalation logic before they build, not after.
- **Cost is a continuous concern**, modeled in design and tracked in operations — not discovered after the invoice arrives.

**Layer 2: The AI-Native Agentic Reference Architecture**

A concrete architecture pattern for multi-agent systems. Designed to be implementable in any tech stack while capturing what makes agentic systems controllable, debuggable, and safe to operate.

Key design decisions I made here:
- **Orchestration logic lives in code, not prompts.** Prompt-embedded control flow is untestable, unversionable, and opaque. Explicit orchestration is none of those things.
- **Agents are designed as bounded reasoning units** with defined input/output contracts and explicit responsibility boundaries. This is a direct response to the "everything agent" pattern that produces systems nobody understands.
- **Tools follow an intent-execution separation.** The model expresses intent; the harness decides whether and how to execute. This decouples reasoning from authorization and makes safety controls composable.
- **State is unified and durable.** Long-running agentic workflows need to survive failures, support human review loops, and be auditable. The architecture addresses this structurally.

---

## How APDLC Relates to What Already Exists

The field does not lack good practice. Governance standards (NIST AI RMF, ISO/IEC 42001), agent patterns (12-Factor Agents, and guidance from Anthropic and Google), and orchestration tooling (LangGraph and its peers) are each strong — they simply live at different altitudes, and none alone carries an AI *product* from idea to operation. APDLC's job is to tie them together, not replace them.

The **inner-loop / outer-loop** distinction is well established in software delivery, and recent work like Kim & Yegge's *Vibe Coding* extends it to AI-assisted development. APDLC applies it at the product level. The SDLC — writing, testing, shipping code — is the inner loop, increasingly the domain of AI development agents (a practice now emerging as the **Agentic Development Lifecycle, ADLC**). This elevates experienced engineers rather than displacing them: their leverage moves up from producing code to directing and governing the systems that produce it.

APDLC operates at the outer loop — the *product* life cycle that surrounds and directs the inner one: framing the problem, designing the system and its autonomy posture, deciding what is worth building and at what risk, evaluating it, operating it, improving it. This loop is inherently human — it is where intent, judgment, and accountability live — and the inner loop cannot operate coherently without it: an automated ADLC still needs something to tell it what is worth building and why. That is APDLC's differentiation: ADLC and agent-build methods make the *building* better; APDLC supplies the coherence that makes the building worth doing — what gets built, for whom, under what controls, and whether it earned its place.

---

## Risk-Proportionate Governance

In APDLC, **the amount of governance produced scales with assessed risk, not with effort or team size**.

Every engagement begins with a risk assessment that establishes the tier: internal prototype, production system, customer-facing system, or regulated system. That assessment then drives everything downstream — which artifacts are required, how deeply each phase is executed, which stakeholders must review, and what the approval chain looks like before moving forward.

Lightweight is not the primary goal; honesty about risk is. Too much governance for the risk slows teams down and gets worked around; too little fails when it matters. The right answer is neither "always minimal" nor "always rigorous," but proportionate.

In practice this means:

- A low-risk internal prototype may produce a handful of focused artifacts and move through phases in days
- A customer-facing system with sensitive data handling produces a full artifact set with structured review cycles at each phase transition
- A regulated system adds compliance-specific evidence requirements on top of the full set

The risk posture established at the start is not static. It is revisited at each major phase transition and updated when operational evidence reveals that actual risk differs from assumed risk.

### Alignment With Established Standards

APDLC is not a parallel invention that ignores the governance work the field has already done. Its risk tiers, control planes, and evaluation practices are mapped onto recognized standards rather than reinventing them:

- **NIST AI Risk Management Framework (AI 100-1)** and the **Generative AI Profile (AI 600-1)** — the Govern, Map, Measure, and Manage functions align with APDLC's lifecycle phases and cross-cutting control planes
- **ISO/IEC 42001:2023** — informs the governance and accountability framing for AI management systems
- **OWASP Top 10 for LLM Applications (2025)** — anchors the agentic threat model, including prompt injection, indirect injection, and excessive agency

This means a team using APDLC is not choosing between a practical framework and standards compliance — the framework is a path toward both at once, calibrated to the assessed risk tier.

## What the Framework Produces

Each phase of APDLC produces structured artifacts. The artifact set is designed so that:

- Decisions are **traceable** — you can see why a design choice was made, not just what was chosen
- Handoffs are **explicit** — the output of one phase is the named input to the next
- Reviews are **structured** — stakeholder review expectations are defined per artifact type and risk tier, not ad hoc

Together these make an engagement auditable: the reasoning behind the system is recoverable, not locked in someone's head.

---

## Evaluation as a First-Class Discipline

Evaluating a non-deterministic system is not the same as testing deterministic software, and "we evaluate continuously" is not an answer. APDLC treats evaluation as a designed discipline with a stance on the problems that actually make it hard:

- **Non-determinism** — quality is judged across multiple runs against thresholds, not a single pass/fail assertion.
- **Evaluation-first** — success criteria and quality thresholds are set in the Frame phase, before anything is built — not reverse-engineered just before release.
- **Multi-modal and layered** — evaluation runs pre-deployment (offline benchmarks, harness runs, red-teaming), at runtime (live metrics, drift detection, A/B comparison), and post-incident (root-cause analysis feeding new regression tests), across unit, system, and production levels.
- **AI-assisted, but not self-certifying** — where AI is used to evaluate AI, the evaluator's own reliability is something to be checked, not assumed.

The detailed mechanics — harness design, scoring methods, rubrics — live in the reference architecture and supporting artifacts, not here. The point of this overview is the posture, not the recipe.

---

## Evolution and Validation

APDLC did not arrive fully formed. The version history reflects real learning:

- **Early versions** focused on getting the lifecycle phases right — what questions does each phase answer, what does it produce?
- **Middle versions** added the architecture layer and formalized evaluation patterns, context engineering, and state management
- **Recent versions** incorporated influence from external methodologies and leading agentic practice — most notably 12-Factor Agents, alongside published guidance from Anthropic and Google — added durable suspension and resume patterns, and formalized multi-channel trigger handling
- **v9** named the delivery path concern for the first time and introduced the AADG (AI Autonomous Development Guide) acknowledges the future will have autonomous Agentic development teams and the unique requirements it brings

Each iteration came from either a structured dry-run that exposed a gap, or external thinking that I assessed, adapted, and integrated — not wholesale adopted. A continious improvment feedback loop embedded explicitly in the APDLC. The first production engagement now underway will be the framework's first test against a live build, and findings from it will feed the next iteration.

---

## In Practice: Knowledge Curator

The framework's first production engagement is **Knowledge Curator** — an agentic system that monitors email, summarizes linked content with Claude, and files curated notes into a structured knowledge vault.

Knowledge Curator exists to keep pace with the very thing this framework wrestles with: the pace of change in AI practice. It summarizes personally currated emerging techniques into a dedicated, isolated vault — kept separate from 15+ years of personal career knowledge — with the eventual goal of connecting that vault to an LLM as a design and challenge partner. APDLC's first real test is itself a tool for staying current.

Its Frame and Design phases are complete. Design alone produced a full set of structured artifacts — problem framing, agent and tool contracts, an orchestration graph, a failure-mode catalog, a cost model, escalation and human-approval policies, and a permission matrix — each reviewed against a defined role matrix before the build was authorized.

A single-owner internal tool, yet specified rigorously enough that its build sequence, failure handling, and cost envelope were all known before a line of production code was written.

---

## The Companion: AI Autonomous Development Guide (AADG)

### The Evolution This Addresses

AI tooling in the development process has moved fast. The progression is real and accelerating:

1. **AI-Assisted** — developers use AI tools to accelerate discrete tasks (code completion, test generation, documentation). Human judgment governs every decision; AI is a productivity multiplier.
2. **AI-Directed** — a human defines the goal and acceptance criteria; an AI agent executes the work, proposes solutions, and iterates. The human reviews and approves, but is no longer doing the work step-by-step.
3. **AI-Autonomous** — an AI agent receives a goal, decomposes it, executes it, evaluates it, and delivers it with minimal or no human involvement in the loop. The human's role shifts to goal-setting and outcome review.

Most organizations are somewhere between stages 1 and 2 today — and the majority have not reasoned about the move toward stage 3 at all. The frameworks, governance patterns, and risk controls appropriate for stage 1 and 2 are not sufficient for stage 3, and practices for navigating that transition safely are still emerging. APDLC takes this problem up early, on purpose: better to frame the questions before organizations need the answers than after they have built something they cannot govern.

### What the AADG Addresses

The AADG is the companion guide for teams moving along this delivery path spectrum. It addresses a distinction that most frameworks collapse: **runtime autonomy** (how autonomous the *system* is when running in production) is a separate concern from **delivery path autonomy** (how autonomous the *development process* itself is).

These interact but they are not the same thing. A fully autonomous production system may have been built through a tightly human-reviewed development process. Conversely, a system built AI-autonomously may be designed to require human approval on every runtime action. Conflating the two leads to governance gaps in the development process, surprises in production, or both.

As the development process becomes more autonomous, new questions emerge that have no equivalent in traditional software work: How do you confirm an AI agent built what you intended when you cannot read every line it wrote? How do you define "done" precisely enough that the agent can judge its own work against it? Framing questions like these early is what the AADG is for.

It is an early, deliberate position rather than a finished manual. The core idea it rests on — separating how the system behaves in production from how the work to build it gets done — is settled and holds up well today. The detailed guidance is still being evaluated and written.

---

## What This Demonstrates

If you are reading this to assess capability rather than to use the framework:

- **Systems thinking at product and architecture altitude simultaneously** — APDLC holds lifecycle concerns and implementation concerns in the same framework without conflating them
- **Principled opinionation** — every design decision in the framework has a *reason*, and the reasons are grounded in actual failure modes, not preferences
- **Domain-agnostic pattern recognition** — the same core concerns (risk posture, HITL strategy, evaluation, cost, auditability) appear in every AI-enabled product regardless of domain; APDLC surfaces them without prescribing domain-specific answers
- **Iterative methodology development** — v9 is the result of treating the framework itself as a product with its own feedback loop

---

## What Is Not in This Document

The operational details of the framework — the facilitation guides, artifact templates, decision logic, and reference architecture specifics — are proprietary and not published here. This overview is intended to convey the thinking behind APDLC and the problems it addresses, not to reproduce it.

For discussion of the framework, engagements, or licensing, contact the John Spangler directly.

---

## About the Author

I'm **John W. Spangler**, an AI practice architect whose career spans the major enterprise technology transformations of the past two decades — enterprise IT service management, Lean/Agile, and DevOps adoption — across individual-contributor, management, and coaching roles. That background informs APDLC's core conviction: the discipline of adopting transformational technology, not the technology itself, is where most organizations succeed or fail.

Having lived through each of those prior waves, my interest now is not in bolting AI onto an existing operating model but in co-creating what comes next. The organizations that navigate this transition well will treat it as an organizational change as much as a technical one — and I am looking to partner with a team that wants to build that future together.

Full background: [linkedin.com/in/johnwspangler](https://www.linkedin.com/in/johnwspangler/)

---

*Author: John W. Spangler*  
*Version: 9 (June 2026)*  
*Status: Actively developed; first production engagement underway — Frame and Design complete*
