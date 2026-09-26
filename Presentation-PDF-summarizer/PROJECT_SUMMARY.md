# Presentation PDF Summarizer - Project Summary

A Streamlit web application that uses Claude to generate structured summaries of presentation slide deck PDFs, supporting both single-deck analysis and cross-deck thematic synthesis.

**Live**: Deployed on Streamlit Community Cloud  
**Repository**: Private GitHub (jwspsdata)  
**Technologies**: Python 3.11+, Anthropic Claude API, PyMuPDF, Streamlit  
**Status**: Deployed, with an 86-test suite covering core logic

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Key Features](#key-features)
4. [Technology Stack](#technology-stack)
5. [Data Pipeline](#data-pipeline)
6. [Code Structure](#code-structure)
7. [Design Patterns](#design-patterns)
8. [Setup & Deployment](#setup--deployment)
9. [Testing](#testing)
10. [Production Considerations](#production-considerations)

---

## Project Overview

### Purpose
Generate structured, actionable summaries of presentation slides using LLM analysis, enabling quick knowledge extraction from conference decks, training materials, and business presentations.

### Who It's For
- **Research teams** analyzing multiple conference presentations for trends
- **Knowledge managers** documenting key insights from presentations
- **Professionals** quickly understanding unfamiliar presentation content
- **Organizations** synthesizing learnings across presentations

### What Makes It Different
- **Structured extraction**: not just text summaries — extracts specific fields (thesis, problem, approach, evidence, value, implications)
- **Multi-slide intelligence**: understands how information is structured across slides (lists, charts, images)
- **Cross-deck synthesis**: beyond single-PDF analysis, detects patterns across multiple presentations
- **Vision-enabled fallback**: uses Claude's vision capabilities for PDFs where text extraction fails
- **Cost-aware**: real-time token tracking and cost calculation; model selection affects final price

---

## Architecture

### System Architecture

```
User Browser (Streamlit UI)
         ↓
    Streamlit App (app.py)
         ├── Session State Management
         ├── File Upload Handler
         ├── Model Selector (Opus/Sonnet/Haiku)
         └── Progress Tracking
         ↓
    PDF Processing Pipeline
         ├── summarizer.py (orchestration)
         │   ├── PDF validation (encryption, empty pages)
         │   ├── Slide-by-slide parsing
         │   ├── Text extraction & cleaning
         │   ├── List detection & parsing
         │   └── Chart/image detection
         └── Claude API Calls
             ├── llm_summarizer.py (per-deck)
             └── theme_analyzer.py (cross-deck)
         ↓
    Markdown Output
         ├── deck_summaries.md (single PDF)
         ├── deck_summaries.md (multiple PDFs)
         └── themes_report.md (cross-deck analysis)
```

### Data Flow

```
Upload PDFs
    ↓
Step 1: Preflight Check (no API calls)
    ├── Detect duplicates
    ├── Validate readability
    └── User confirmation on issues
    ↓
Step 2: PDF Content Extraction (local)
    ├── Open with PyMuPDF
    ├── Extract text from each slide
    ├── Detect and categorize content
    │   ├── Charts/tables (has numeric data)
    │   ├── Pure images (no text)
    │   └── Text-heavy slides
    ├── Clean and normalize text
    └── Parse structured lists
    ↓
Step 3: LLM Summarization (Claude API)
    ├── Per-deck: Call LLM for structured fields
    │   ├── System prompt: expert summarizer role
    │   ├── User message: slide data + context
    │   └── Output: validated JSON with 13 fields
    └── Track tokens & cost
    ↓
Step 4: Optional - Cross-Deck Analysis
    ├── Input: All deck summaries
    ├── Call LLM for thematic synthesis
    └── Output: Markdown report with 10 themes
    ↓
Download Results
    ├── Single file: {stem}_summary.md
    ├── Multiple files: deck_summaries.md
    └── Cross-deck: themes_report.md
```

---

## Key Features

### 1. Per-Deck Summarization

Extracts 13 structured fields for each PDF:

| Field | Purpose | Example |
|-------|---------|---------|
| **deck_title** | Actual presentation title (from slide 1) | "Production > Prototype: Replacing Core Platforms" |
| **summary** | 2-sentence thesis + distinctive insight | "The presentation shows how small teams can replace legacy systems rapidly. Key insight: parallel systems reduce risk..." |
| **problem_context** | Challenge being addressed | "Legacy platform blocked feature velocity; required 6+ month migrations" |
| **approach_shared** | High-level solution framing | "Build new system in parallel; migrate users incrementally" |
| **approach_practices** | Specific named practices (2-5 items) | ["Canary deployments by region", "Feature flags for rollback", "Automated smoke tests"] |
| **supporting_evidence** | Metrics or case studies | "Reduced migration time from 6 months to 3 weeks; cut deployment risk by 90%" |
| **value_add** | Outcomes and benefits | "Teams ship features 2x faster; customer satisfaction increased 15%" |
| **team_implications** | IC/team-level people change | "Engineers now own deployment decisions; QA shifts to continuous testing" |
| **management_implications** | Manager/leader decisions | "Hiring shifted from backend specialists to platform generalists" |
| **skills_role_evolution** | New capabilities needed | "Teams must learn Go and Kubernetes; legacy COBOL skills deprecating" |
| **chart_table_insights** | Data-heavy slide findings | "Timeline visualization shows 40% reduction in migration phases" |
| **risks_failure_modes** | What went wrong or nearly did | "Early rollouts revealed network latency issues in new region" |
| **presenter_orgs** | Organization(s) represented | "Acme Corp, Delivery Partner Inc." |

### 2. Thematic Cross-Deck Analysis (Multiple PDFs)

Synthesizes patterns across decks into 10 thematic sections:

- **Executive Summary**: 5-7 headline findings
- **Common Challenges**: team/org obstacles
- **Risks & Failure Modes**: cautionary patterns
- **Common Practices**: how work is organized
- **Technology Patterns**: tools and architectures
- **Governance & Guardrails**: control mechanisms
- **Value & Outcomes**: measured results
- **Organizational Readiness**: cultural/structural factors
- **Adoption Maturity & Speed**: scaling patterns
- **Skills & Role Evolution**: capability changes

**Citation tracking**: each claim cites source decks [N] for traceability.

### 3. Content Handling

**Text Extraction & Cleaning**
- Multi-line text from PyMuPDF with proper formatting
- Normalization of whitespace, case, special characters
- Deduplication of repeated content
- Handling of slides with mixed languages

**List Detection**
- Identifies implied numerical sequences in slides
- Detects explicit bullet/number patterns
- Categorizes as "step sequence" (process) or "named items"
- Renders summaries like "4-step process: Design → Build → Test → Deploy"

**Chart & Table Recognition**
- Detects numeric data, percentages, trends
- Identifies org charts, timelines, matrices
- Filters false positives (e.g., org charts with no data)
- Extracts first relevant evidence line for reporting

**Vision Fallback**
- When text extraction yields no title: renders PDF slide as PNG
- Sends image to Claude for title extraction
- Falls back to heuristic extraction if vision fails

### 4. Model Selection & Cost Tracking

**Three Claude models available:**

| Model | Speed | Cost | Best For |
|-------|-------|------|----------|
| **Haiku 4.5** | Fastest | Cheapest | High-volume batch jobs, cost-sensitive |
| **Sonnet 4.6** | Fast | Moderate | Default; balanced speed/cost/quality |
| **Opus 4.6** | Slower | Most expensive | Complex decks, nuanced reasoning needed |

**Real-time cost calculation:**
- Input tokens (prompt + slide content)
- Output tokens (generated summary JSON)
- Cache creation tokens (10x cheaper on repeated reads)
- Cache read tokens (96% savings on cache hits)

**UI shows**:
- Tokens used: `234,567 in / 12,345 out / 567 cached`
- Estimated cost: `$0.0234`

### 5. Reliability

**Error Handling**
- Duplicate filename detection
- Encrypted PDF rejection
- Empty PDF handling
- Unreadable PDF graceful failure
- LLM timeout/retry logic
- Malformed JSON recovery

**State Management**
- Session state persistence across reruns
- File upload change detection (reset all processing)
- Model change handling (reset cached results)
- Progress tracking across multi-file batches

**Deployment**
- Works locally (environment variable) and cloud (Streamlit secrets)
- No file I/O side effects (uses tempfiles)
- Stateless API calls (Anthropic SDK)
- Configurable via TOML files

---

## Technology Stack

### Core Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| **anthropic** | ≥0.94.1 | Claude API client; models, vision, messages |
| **pymupdf** | Latest | PDF parsing; text/image extraction, slide inspection |
| **streamlit** | Latest | Web UI; file upload, progress bars, session state |

**Total package size**: ~50MB (slim; no heavy ML frameworks)

### Design Approach
- **Minimal dependencies**: only what's essential
- **Modern Python 3.11+**: type hints, match statements, structural pattern matching
- **Synchronous I/O**: straightforward flow; Anthropic client handles async internally
- **No caching framework**: leverages Anthropic's prompt caching API directly

---

## Data Pipeline

### Phase 1: Preflight (No API Calls)

**Input**: List of uploaded PDF files  
**Process**:
```python
for each file:
    - Check for duplicate filename
    - Try to open with PyMuPDF
    - Verify not encrypted
    - Verify has ≥1 page
    - Store valid paths
```
**Output**: `(valid_paths: list[Path], error_messages: list[str])`

**User decision**: proceed with valid files or fix and re-upload.

### Phase 2: Content Extraction (Local, No API)

**Input**: Valid PDF paths  
**Process** (per PDF):
```
For each slide:
    - Extract text via PyMuPDF
    - Clean/normalize lines
    - Detect if chart/table
    - Detect if pure image
    - Build slide_data struct

Build metadata:
    - Extract deck title from slide 1 (text or vision)
    - Infer author from filename or slide content
    - Parse structured lists
    - Identify chart evidence lines
```
**Output**: `slide_data: list[dict]` with 5-key entries per slide

**Cost**: none (local processing)

### Phase 3: LLM Summarization (Claude API)

**Input**: `slide_data` (extracted slide content)  
**Process**:
```
1. Build user prompt:
   - Deck title candidate
   - PDF filename
   - Slide-by-slide text, titles, types
   - Implied list summaries
   - Chart evidence lines

2. Call Claude:
   - System: expert summarizer role
   - Messages: user prompt + slide data
   - Model: user-selected (Haiku/Sonnet/Opus)
   - Max tokens: 1500
   - Temperature: 1 (creative but grounded)

3. Parse response:
   - Validate JSON structure
   - Extract 13 fields
   - Sanitize all strings (remove Unicode, control chars)
```

**Output**: `llm: dict[str, str]` + `usage: dict[str, int]`

**Cost**: ~0.15-0.50 USD per deck (Sonnet)

### Phase 4: Cross-Deck Analysis (Optional)

**Input**: Concatenated deck summaries (markdown)  
**Process**:
```
1. Parse all deck summaries:
   - Extract deck titles
   - Extract field values
   - Extract practices lists

2. Build deck index:
   - Numbered map: [1] → {title, author, all fields}

3. Call Claude:
   - System: thematic synthesis role
   - Messages: summaries + deck index
   - Model: same as user selection
   - Max tokens: 4000

4. Enforce structure:
   - Parse markdown into 10 sections
   - Validate citation format [N]
   - Ensure max 8 citations per section
```

**Output**: `themes_md: str` (markdown report)

**Cost**: ~0.05-0.20 USD per analysis (depends on deck count)

---

## Code Structure

### Directory Layout

```
Presentation_PDF_summarizer/
├── app.py                          # Streamlit entry point (254 lines)
├── requirements.txt                # Dependencies
├── pyproject.toml                  # Package metadata
│
├── src/pdf_summary/                # Core library
│   ├── __init__.py
│   ├── summarizer.py               # Main orchestration (230 lines)
│   │   ├── validate_pdfs()         # Check encryption, empty, readable
│   │   ├── generate_summary()      # Full pipeline: extract → LLM → format
│   │   └── main()                  # CLI entry point
│   │
│   ├── llm_summarizer.py           # Claude API for per-deck (330 lines)
│   │   ├── extract_title_via_vision()  # Fallback for vision
│   │   ├── call_llm_summarizer()       # Main API call
│   │   └── SYSTEM_PROMPT               # Expert summarizer role
│   │
│   ├── theme_analyzer.py           # Claude API for cross-deck (280 lines)
│   │   ├── parse_deck_summaries()  # Parse markdown → dicts
│   │   ├── build_deck_index()      # Numbering for citations
│   │   ├── generate_themes_report()    # Main API call
│   │   └── SYSTEM_PROMPT               # Thematic synthesis role
│   │
│   ├── model_config.py             # Model registry & pricing (78 lines)
│   │   ├── MODELS                  # Dict of {model_id: pricing}
│   │   ├── selectbox_options()     # For UI dropdown
│   │   ├── calculate_cost()        # USD estimation
│   │   └── usage_from_response()   # Extract token counts
│   │
│   ├── content_builders.py         # Metadata extraction (130 lines)
│   │   ├── extract_title_from_page()   # Heuristic title detection
│   │   ├── infer_deck_title()          # Fallback from filenames
│   │   ├── infer_author()              # From filename or content
│   │   └── is_chart_or_table()         # Numeric data detection
│   │
│   ├── list_parsing.py             # Structured list detection (110 lines)
│   │   ├── parse_structured_lists()    # Identify sequences/items
│   │   └── render_structured_lists()   # Format for display
│   │
│   └── text_processing.py          # Text cleaning (180 lines)
│       ├── clean_lines()           # Whitespace normalization
│       ├── normalize_title()       # Case & special char handling
│       ├── sanitize_summary_line() # Remove Unicode artifacts
│       └── append_bullet()         # Markdown formatting
│
├── .streamlit/
│   ├── config.toml                 # UI settings (centered layout)
│   └── secrets.toml.example        # Template for API key
│
├── configs/
│   └── summary_paths.toml          # CLI config: input dir, output, limits
│
├── tests/unit/
│   ├── test_content_builders.py
│   ├── test_list_parsing.py
│   ├── test_llm_summarizer.py
│   ├── test_model_config.py
│   ├── test_summarizer_contract.py
│   ├── test_text_processing.py
│   ├── test_theme_analyzer.py
│   └── test_placeholder.py
│
├── docs/                           # Documentation
├── outputs/                        # Sample outputs (git-ignored)
└── README.md                       # User-facing guide
```

### Key Modules

#### **app.py** (Streamlit UI) — 254 lines
**Responsibilities**:
- User interface entry point
- Session state management (9 state keys tracked)
- File upload handling & duplicate detection
- Progress tracking with progress bars
- Model selection dropdown
- Cost estimation display
- Preflight validation UI
- Download button generation

**State Flow**:
```
uploaded_files → preflight_done → user_confirmed → summaries_md → themes_md
```

#### **summarizer.py** (Orchestration) — 230 lines
**Responsibilities**:
- Validate PDFs (encryption, empty, readable)
- Orchestrate full pipeline
- Extract slide content (text, images, metadata)
- Call LLM for per-deck summary
- Format and write markdown output
- Progress callbacks for UI

**Key functions**:
- `validate_pdfs(paths)` → `(valid_paths, error_msgs)`
- `generate_summary(project_root, input_dir, output_path, limit, on_progress, model)` → `(pdfs, md, usage)`
- `main()` → CLI entry point with argparse

#### **llm_summarizer.py** (Per-Deck LLM) — 330 lines
**Responsibilities**:
- Build structured prompts from slide data
- Call Claude API with system prompt
- Extract and validate JSON response
- Handle vision-based title fallback
- Track token usage

**Key functions**:
- `extract_title_via_vision(png_bytes, model)` → `str`
- `call_llm_summarizer(deck_title, filename, slide_data, ..., model)` → `(llm_response_dict, usage)`

**System Prompt**: 38-line expert summarizer role with strict rules for 13 output fields

#### **theme_analyzer.py** (Cross-Deck LLM) — 280 lines
**Responsibilities**:
- Parse deck summaries into structured data
- Build indexed lookup (deck numbers)
- Call Claude API for synthesis
- Enforce thematic structure (10 sections)
- Validate citations [N] format

**Key functions**:
- `parse_deck_summaries(md_text)` → `list[dict]`
- `build_deck_index(decks)` → `dict[int, dict]`
- `generate_themes_report(summaries_md, model)` → `(markdown, usage)`

**System Prompt**: 45-line expert analyst role with 10-section structure and citation rules

#### **model_config.py** (Registry) — 78 lines
**Responsibilities**:
- Define available Claude models and pricing
- Provide UI dropdown options
- Calculate USD costs from token usage
- Normalize usage dicts
- Extract token counts from API responses

**Data**:
```python
MODELS = {
    "claude-opus-4-6": {input: 15.00, output: 75.00, cache_write: 18.75, cache_read: 1.50},
    "claude-sonnet-4-6": {input: 3.00, output: 15.00, cache_write: 3.75, cache_read: 0.30},
    "claude-haiku-4-5-20251001": {input: 0.80, output: 4.00, cache_write: 1.00, cache_read: 0.08},
}
```

#### **content_builders.py** (Metadata) — 130 lines
**Responsibilities**:
- Extract presentation title from first slide
- Infer author from filename or slide content
- Detect charts and tables (numeric data)
- Identify pure-image slides
- Handle edge cases (single-word titles, name patterns)

#### **list_parsing.py** (Sequences) — 110 lines
**Responsibilities**:
- Detect numbered/bulleted sequences in slides
- Distinguish process steps from named items
- Build summaries like "4-step process"
- Render for display in markdown

#### **text_processing.py** (Cleaning) — 180 lines
**Responsibilities**:
- Normalize whitespace and line breaks
- Title case conversion (handles "iOS", "APIs")
- Remove Unicode artifacts and control characters
- Sanitize summary lines
- Append bullets to markdown in proper format

---

## Design Patterns

### 1. Orchestration (summarizer.py)

The entry point coordinates sub-modules without embedding their low-level logic:

```python
def generate_summary(...):
    # Step 1: Load and validate
    pdfs = sorted([...])
    
    # Step 2: Per-PDF loop
    for pdf in pdfs:
        # Delegate extraction
        slide_data = extract_slides(pdf)
        
        # Delegate LLM call
        llm_result = call_llm_summarizer(slide_data, model)
        
        # Delegate formatting
        md = format_deck_summary(llm_result)
        
    return pdfs, md, total_usage
```

Keeping the coordinator thin makes the flow easy to follow and easy to extend with new processing steps without touching the orchestration logic itself.

### 2. Two-Phase Processing (app.py + summarizer.py)

**Phase 1 (no API calls)**: preflight duplicate detection and PDF validation, with a user decision point to confirm or fix issues before anything is sent to Claude.

**Phase 2 (with API)**: extract content locally, call Claude per deck, aggregate results, format output.

This ordering means a batch of bad files fails before any API cost is incurred, rather than partway through a paid run.

### 3. Session State as Single Source of Truth (app.py)

```python
_STATE_KEYS = (
    "summaries_md", "themes_md", "summary_filename", 
    "pdf_count", "uploaded_names", "preflight_done", ...
)

# Every rerun: check state to skip completed steps
if st.session_state.summaries_md is not None:
    # Display results; skip processing
    st.download_button(...)
else:
    # Still processing; show UI
    st.button("Summarize", ...)
```

Streamlit reruns the whole script on every interaction, so state has to live outside the function body to survive that. Tracking it explicitly in one place keeps the state transitions legible.

### 4. Prompt as Source of Truth (llm_summarizer.py, theme_analyzer.py)

System prompts define the exact output structure and rules directly, rather than leaving field definitions implicit in downstream parsing code:

```python
SYSTEM_PROMPT = """
You are an expert summarizer...
Produce a structured JSON with exactly these fields:
- summary: Two sentences. First: ...
- problem_context: ... One sentence.
- approach_practices: JSON array of short phrases (2-5 items).
- ...
"""
```

Keeping the schema in the prompt (rather than duplicated across prompt and parser) means the output structure only needs to change in one place.

### 5. Configurable Models (model_config.py)

A registry decouples model identifiers and pricing from the rest of the code:

```python
MODELS = {
    "claude-opus-4-6": {...},
    "claude-sonnet-4-6": {...},
    "claude-haiku-4-5-20251001": {...},
}

# Adding a new model is a dict entry -- the UI dropdown and pricing pick it up automatically
```

### 6. Pure Functions for Text/List Transformations (text_processing.py, list_parsing.py)

```python
# Avoided: class TextProcessor { def clean() { ... } }
# Used instead: def clean_lines(text: str) -> list[str] { ... }
```

Plain functions over stateful classes here, since these are pure transformations with no reason to carry state between calls — they're easier to test and compose as a result.

### 7. Error Handling as Validation (summarizer.py)

Errors are surfaced to the user rather than raised as unhandled exceptions:

```python
def validate_pdfs(paths):
    valid, errors = [], []
    for path in paths:
        if is_encrypted(path):
            errors.append(f"{path.name}: password-protected")
        else:
            valid.append(path)
    return valid, errors
```

This lets a batch with one bad file continue processing the rest, with a clear message about what to fix.

---

## Setup & Deployment

### Local Setup

```bash
# 1. Clone and install
git clone https://github.com/jwspsdata/Presentation_PDF_summarizer
cd Presentation_PDF_summarizer
pip install -r requirements.txt

# 2. Create secrets file
cp .streamlit/secrets.toml.example .streamlit/secrets.toml
# Edit: Add your ANTHROPIC_API_KEY

# 3. Run locally
streamlit run app.py
# Opens at http://localhost:8501

# 4. Test
python -m pytest tests/ -v
```

### Deploy to Streamlit Cloud

```bash
# 1. Push to GitHub (public or private)
git push origin main

# 2. Go to https://share.streamlit.io
# Sign in with GitHub account
# Click "New app"
# Select repo, branch, main file (app.py)

# 3. In app dashboard: Settings → Secrets
# Add: ANTHROPIC_API_KEY = "sk-ant-..."

# 4. App auto-deploys and auto-reruns on push
```

### CLI Usage

```bash
# Summarize first 3 PDFs (config default)
python -m pdf_summary.summarizer

# Summarize all PDFs in a directory
python -m pdf_summary.summarizer --all --input-dir ./decks

# Custom output path
python -m pdf_summary.summarizer --output ./my_summary.md

# Limit to N PDFs
python -m pdf_summary.summarizer --limit 5

# Use different model
python -m pdf_summary.summarizer --model claude-opus-4-6
# (Note: CLI env var ANTHROPIC_API_KEY required)
```

---

## Testing

### Test Coverage

86 unit tests across 8 test modules:

| Module | Tests | Focus |
|--------|-------|-------|
| `test_content_builders.py` | 6 | Title extraction, author inference, chart detection |
| `test_list_parsing.py` | 2 | Sequence detection, rendering |
| `test_llm_summarizer.py` | 20 | Prompt building, JSON validation |
| `test_model_config.py` | 18 | Model registry, pricing, cost calculation |
| `test_text_processing.py` | 3 | Cleaning, normalization, sanitization |
| `test_theme_analyzer.py` | 17 | Deck parsing, indexing, citation validation |
| `test_summarizer_contract.py` | 19 | End-to-end contract test (optional) |
| `test_placeholder.py` | 1 | Placeholder (no-op) |

### Running Tests

```bash
# Run all tests with verbose output
python -m pytest tests/ -v

# Run specific test module
python -m pytest tests/unit/test_text_processing.py -v

# Run tests matching a pattern
python -m pytest tests/ -k "clean" -v

# Show coverage report
pip install pytest-cov
python -m pytest tests/ --cov=src/pdf_summary --cov-report=html
# Open htmlcov/index.html
```

### Example Test

```python
# test_text_processing.py
def test_clean_lines():
    raw = "   Hello   \n  World  \n"
    result = clean_lines(raw)
    assert result == ["Hello", "World"]
    assert all(not line.startswith(" ") for line in result)
```

---

## Production Considerations

### 1. API Rate Limiting

Anthropic enforces per-minute request and token limits that scale with account tier; the specific ceiling depends on which tier the deployed key is on.

**Mitigation**:
- Batch processing (multiple PDFs in one app session)
- Model selection (Haiku for high-volume, Opus for complex)
- Token caching (repeat summaries cost a fraction as much)

### 2. Error Recovery

**Transient errors** (network timeouts, rate limits):
- Anthropic SDK auto-retries with exponential backoff
- User can click "Summarize" again on timeout

**Permanent errors** (invalid PDF, malformed response):
- Error message shown to user
- Non-blocking: other decks continue processing
- User can download partial results

### 3. Cost Management

**Per-deck cost** (Sonnet):
- Input tokens: ~5,000-8,000 (slide text + metadata)
- Output tokens: ~500-800 (structured JSON)
- Total: ~$0.02-0.05 per deck

**Cross-deck analysis** (10 decks):
- Input tokens: ~30,000 (all summaries)
- Output tokens: ~1,500-2,000 (thematic report)
- Total: ~$0.10-0.20 per analysis

**Cost levers**:
- Haiku: substantially cheaper than Sonnet, at a real but often acceptable quality tradeoff for simpler decks
- Batch processing: cache write on the first summary, cache read on the themes analysis
- Model selection: the UI lets users choose cost vs. quality directly

### 4. Privacy & Security

**Data handling**:
- PDFs uploaded to app; processed locally; sent to Anthropic API only
- No file storage; temp directories cleaned up immediately
- No logging of PDF content (only token counts logged)

**Secrets**:
- Streamlit secrets.toml (local) or Streamlit dashboard (cloud)
- API key never logged or printed
- Environment variable fallback for non-Streamlit deployments

### 5. Scalability

**Single instance** (current):
- Handles 1-50 PDFs per session
- Serialized API calls (no parallelism)
- Suitable for typical conference presentation analysis

**Multi-instance** (if needed):
- Streamlit Cloud auto-scales
- Anthropic API handles concurrent requests
- No shared state between instances

### 6. Monitoring & Observability

**What's tracked**:
- Tokens consumed per session (input, output, cached)
- Cost estimation per session
- Processing time per PDF
- Error counts and types

**What's not tracked**:
- PDF content (privacy)
- User identities (Streamlit Cloud doesn't auth)
- LLM response quality (no feedback mechanism yet)

**Gaps worth closing**:
- No tracing (LangSmith or similar) for detailed observability
- No user feedback loop (thumbs up/down on summaries)
- No cost alerting if a session exceeds an expected threshold

---

## Development Roadmap

### Implemented
- Per-deck summarization (13 fields)
- Cross-deck thematic analysis (10 themes)
- Multi-model selection (Haiku/Sonnet/Opus)
- Cost tracking and estimation
- Vision-based title extraction fallback
- Error handling across the pipeline
- 86 unit tests
- Streamlit Cloud deployment

### Not Yet Built
- **Batch processing**: drag-and-drop folder of PDFs
- **Template selection**: choose summary schema (executive, technical, business)
- **Custom prompts**: user-provided system prompts
- **Multi-language support**: translate summaries to other languages
- **Comparison mode**: side-by-side analysis of 2-3 decks
- **Search integration**: index and search across all summaries
- **Feedback loop**: user ratings to improve LLM prompts
- **Caching strategy**: store completed summaries to avoid re-processing
- **Async processing**: background jobs for large batches
- **Export formats**: JSON, CSV, HTML in addition to Markdown

---

## Summary

Presentation PDF Summarizer is a Python application that pairs multi-model Claude integration (cost tracking, caching, vision fallback) with a PyMuPDF-based extraction pipeline and a Streamlit UI built around explicit, testable session state. Structured outputs come from system prompts with strict field schemas, validated and sanitized on parse. The 86-test suite covers the core logic (prompt building, model pricing, text cleaning, theme parsing) rather than the Streamlit UI layer itself, and the app runs both locally and on Streamlit Cloud without code changes.

Design choices worth calling out on their own:
- Three dependencies total (anthropic, pymupdf, streamlit) — no heavier framework pulled in for what these three already cover
- Type hints throughout, targeting modern Python 3.11+
- Clear separation between UI, extraction, LLM calls, and formatting, so each is independently testable
- Errors surface to the user with enough detail to fix and retry, rather than failing silently
- Token and cost accounting is visible in the UI itself, not something the user has to check separately

---

*Last Updated: June 2026*
*Status: Live & Maintained*
