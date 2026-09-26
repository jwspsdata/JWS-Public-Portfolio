# Technologies Reference Sheet
## Presentation PDF Summarizer

Quick reference for the technologies, frameworks, and tools used in this project.

---

## Core Technologies

### Anthropic Claude API

**Models Available**:
- `claude-opus-4-6`: most capable; best for complex reasoning ($15 in / $75 out per M tokens)
- `claude-sonnet-4-6`: balanced; default model ($3 in / $15 out per M tokens)
- `claude-haiku-4-5-20251001`: fastest; cheapest ($0.80 in / $4 out per M tokens)

**Features Used**:
- Messages API: structured text summarization
- Vision API: title extraction from slide images (PNG)
- JSON mode: enforced structure for 13-field output
- Prompt caching: 90% cost savings on repeated content

**Version**: `anthropic>=0.94.1`

**Integration Pattern**:
```python
import anthropic
client = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
response = client.messages.create(
    model=selected_model,
    max_tokens=1500,
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": prompt},
            # or {"type": "image", "source": {...}} for vision
        ]
    }]
)
```

---

## PDF Processing

### PyMuPDF (fitz)

**Purpose**: extract text, images, metadata from PDF slides

**Key Methods**:
- `fitz.open(path)`: open PDF
- `doc.load_page(i)`: get page object
- `page.get_text('text')`: extract text
- `page.get_images(full=True)`: list images
- `page.get_pixmap(matrix=...)`: render as PNG (for vision API)
- `doc.is_encrypted`: check if password-protected
- `len(doc)`: page count

**Use Cases**:
- PDF validation (encryption, empty pages)
- Slide-by-slide text extraction
- Title extraction via vision (fallback)
- Metadata (slide count, content type detection)

**Version**: latest

---

## Web Framework

### Streamlit

**Purpose**: interactive web UI for file upload, processing, and download

**Components Used**:
- `st.set_page_config()`: page setup (title, layout="centered")
- `st.file_uploader()`: multi-file PDF upload
- `st.button()`: Summarize and Analyze Themes buttons
- `st.progress()`: progress bar for batch processing
- `st.empty()`: placeholder for status text
- `st.selectbox()`: model selection dropdown
- `st.session_state`: persistent state across reruns
- `st.download_button()`: download markdown files
- `st.success()`, `st.error()`, `st.warning()`, `st.info()`: status messages
- `st.spinner()`: loading indicator
- `st.sidebar`, `st.expander()`, `st.tabs()`: layout

**Session State Keys Tracked**:
```python
summaries_md          # Generated markdown
themes_md             # Cross-deck themes report
summary_filename      # Filename for download
pdf_count             # Number of PDFs processed
uploaded_names        # Sorted list of uploaded file names
preflight_done        # Validation phase completed
preflight_issues      # List of validation errors
valid_file_names      # Files that passed validation
user_confirmed        # User confirmed after issues
selected_model        # User's model choice
summary_usage         # Token counts from summarization
themes_usage          # Token counts from theme analysis
```

**Deployment Options**:
- Local: `streamlit run app.py`
- Cloud: Streamlit Community Cloud (free)
- Custom: Docker container on any server

---

## Python Libraries

### Core Python
- **Version**: 3.11+
- **Features Used**: type hints, match statements, f-strings, walrus operator (`:=`)
- **Modules**: 
  - `os`, `sys`: environment, paths
  - `pathlib.Path`: modern file paths
  - `tempfile`: temporary directories
  - `json`: JSON parsing/validation
  - `re`: regex for text extraction
  - `tomllib`: TOML config parsing (Python 3.11+)
  - `argparse`: CLI argument parsing
  - `collections.abc.Callable`: type hints for callbacks

### Testing
- **pytest**: 86 unit tests
- **pytest-cov** (optional): coverage reporting

---

## Data Structures

### Core Data Models

**Slide Data**:
```python
slide_data = [
    {
        'idx': 1,                          # Slide number
        'title': 'Introduction',           # Extracted title
        'lines': ['Introduction', ...],    # Cleaned text lines
        'chart_table': False,              # Has numeric data?
        'pure_image': False,               # No text; image only?
    },
    ...
]
```

**LLM Response** (per-deck):
```python
llm_result = {
    'deck_title': str,
    'summary': str,
    'problem_context': str,
    'approach_shared': str,
    'approach_practices': list[str],
    'supporting_evidence': str | None,
    'value_add': str | None,
    'team_implications': str | None,
    'management_implications': str | None,
    'skills_role_evolution': str | None,
    'chart_table_insights': str | None,
    'risks_failure_modes': str | None,
    'presenter_orgs': str | None,
}
```

**Usage Tracking**:
```python
usage = {
    'input_tokens': int,
    'output_tokens': int,
    'cache_creation_input_tokens': int,    # One-time cost
    'cache_read_input_tokens': int,         # Cheap repeat access
}
```

---

## Project Configuration

### Streamlit Config (`.streamlit/config.toml`)
```toml
[client]
showErrorDetails = true

[theme]
primaryColor = "#FF0000"
```

### Streamlit Secrets (`.streamlit/secrets.toml`)
```toml
ANTHROPIC_API_KEY = "sk-ant-..."
```

### Pipeline Config (`configs/summary_paths.toml`)
```toml
[summary]
input_dir = "decks/"
output_file = "deck_summaries_first3.md"
pdf_limit = 3
```

---

## Dependency Graph

```
Presentation PDF Summarizer
├── anthropic (>=0.94.1)
│   ├── httpx (HTTP client)
│   ├── pydantic (JSON parsing)
│   └── typing-extensions (Type hints)
├── pymupdf (PDF extraction)
│   └── (C library dependencies)
└── streamlit (Latest)
    ├── click (CLI framework)
    ├── watchdog (File monitoring)
    ├── altair (Charting)
    ├── pandas (Data processing)
    ├── numpy (Numerical computing)
    ├── pillow (Image handling)
    └── ...20+ transitive deps
```

**Total Package Size**: ~50MB  
**Installation Time**: ~30-60 seconds

---

## Processing Pipeline Technologies

### Text Processing
- **Regex** (`re` module): pattern matching for titles, metrics, control characters
- **String methods**: strip, split, join, replace
- **Normalization**: case conversion (titlecase with "iOS" handling)
- **Sanitization**: remove Unicode artifacts, control characters

### List Parsing
- **Pattern matching**: detect numbered sequences (1, 2, 3) or bullets
- **Heuristics**: distinguish "process steps" from "named items"
- **Rendering**: convert to "N-step process" format

### Chart Detection
- **Numeric pattern matching**: `\d|%` regex
- **Keyword hinting**: "trend", "increase", "decrease", "axis"
- **False positive filtering**: org charts without data

### Content Classification
- **Image detection**: PyMuPDF `get_images()`
- **Text availability**: check extracted text length
- **Chart/table marking**: numeric data + keywords

---

## Security & Authentication

### API Key Management
- **Streamlit local**: `.streamlit/secrets.toml` (git-ignored)
- **Streamlit Cloud**: secrets manager in dashboard
- **CLI/Script**: `ANTHROPIC_API_KEY` environment variable

### Data Privacy
- No logging of PDF content
- Temporary files deleted immediately after processing
- No persistence of summaries (user downloads them)
- API calls logged only for token counting

### Dependency Security
- Three core packages, kept deliberately minimal to limit the dependency surface
- No automated vulnerability scanning or update bot wired up yet — updates are manual

---

## Monitoring & Observability

### Token Tracking
- Input tokens (prompt + content)
- Output tokens (generated summary)
- Cache creation tokens (first write; 5x normal cost)
- Cache read tokens (subsequent access; 1/5 normal cost)

### Cost Calculation
```python
cost = (
    input_tokens * input_price +
    output_tokens * output_price +
    cache_creation_tokens * cache_write_price +
    cache_read_tokens * cache_read_price
) / 1_000_000
```

### Metrics Exposed in UI
- Tokens: `234,567 in / 12,345 out / 567 cached`
- Estimated USD: `$0.0234`
- Processing time: implicit (progress bar)

---

## Testing Technologies

### Test Framework: pytest

**Test Modules**:
- `test_content_builders.py` (6 tests): title/author extraction, chart detection
- `test_list_parsing.py` (2 tests): sequence detection, rendering
- `test_llm_summarizer.py` (20 tests): prompt building, response validation
- `test_model_config.py` (18 tests): model registry, pricing, costs
- `test_text_processing.py` (3 tests): cleaning, normalization, sanitization
- `test_theme_analyzer.py` (17 tests): deck parsing, indexing
- `test_summarizer_contract.py` (19 tests): integration test (optional)
- `test_placeholder.py` (1 test): placeholder (no-op)

**Test Execution**:
```bash
pytest tests/ -v                           # All tests, verbose
pytest tests/unit/test_text_processing.py  # Specific module
pytest tests/ -k "clean"                   # Pattern matching
pytest tests/ --cov=src/pdf_summary        # Coverage report
```

**Coverage**: concentrated on the core pipeline logic (prompt building, parsing, pricing) rather than the Streamlit UI layer, which isn't exercised by the test suite.

---

## Deployment Technologies

### Local Development
- **OS**: Windows, macOS, Linux
- **Python**: 3.11+
- **Virtual Environment**: venv or conda
- **Package Manager**: pip

### Streamlit Cloud Deployment
- **Hosting**: Streamlit Community Cloud (free)
- **CI/CD**: GitHub auto-deploy on push
- **Secrets**: web dashboard
- **Scaling**: automatic

### Alternative Deployments (not built)
- Docker: would need a Dockerfile, not currently in the repo
- AWS Lambda: would need a FastAPI wrapper
- Heroku: would need a Procfile

---

## Version Reference

### Dependency Pinning
```
anthropic >= 0.94.1    (floor pinned; otherwise auto-latest)
pymupdf                (auto-latest)
streamlit              (auto-latest)
pytest                 (auto-latest, for testing)
```

No lockfile — installs resolve to whatever's current at install time. Fine for a personal demo project; would need locking down for anything with more than one contributor or a CI pipeline.

### Compatibility
- **Python**: 3.11, 3.12, 3.13 (expected, not exhaustively tested)
- **OS**: Windows, macOS, Linux
- **Browsers**: modern browsers (Chrome, Firefox, Safari, Edge)

---

## External Services

### Anthropic Claude API
- **Endpoint**: `https://api.anthropic.com/v1/messages`
- **Authentication**: API key in Authorization header
- **Rate limits**: per-minute request and token limits that scale with account tier; not something this app manages or negotiates directly

### Streamlit Community Cloud
- **Endpoint**: user-provided URL (e.g., `pdfsum-project.streamlit.app`)
- **Infrastructure**: Streamlit-managed
- **Limits**: free tier caps active apps and storage per account
- **Scaling**: automatic within the free tier's session limits

---

## Technology Choices & Rationale

### Why PyMuPDF (not pdfplumber / pypdf)?
- Fast text and image extraction
- Direct PNG rendering, needed for the Claude vision fallback
- Handles encrypted PDFs and empty pages without extra wrapper code
- No heavy dependency chain

### Why Streamlit (not Dash / FastAPI)?
- Fastest path from idea to a working UI for this kind of upload-process-download flow
- File upload, progress bars, and session state are built in rather than assembled from parts
- Single-command deployment, no separate infrastructure to stand up
- Tradeoff: synchronous only — acceptable here since the app processes one batch at a time, would need reconsidering for concurrent multi-user load

### Why Anthropic (not OpenAI)?
- Vision API fit the title-extraction fallback need directly
- Sonnet pricing compares favorably to GPT-4-class models at similar quality
- Prompt caching cuts repeated-content cost substantially

### Why Minimal Dependencies?
- Smaller attack surface and fewer version conflicts to manage
- Faster Streamlit cold-start
- Easier for someone else (or future me) to read the entire stack in one sitting

---

## Tech Stack Comparison

| Aspect | Choice | Alternative(s) | Why |
|--------|--------|-----------------|-----|
| **LLM** | Anthropic Claude | OpenAI GPT-4, Gemini | Cheaper at comparable quality, plus a vision API this project actually uses |
| **PDF** | PyMuPDF | pdfplumber, pypdf | Fastest extraction, with PNG rendering built in |
| **Web** | Streamlit | Dash, FastAPI | Fastest path to a working UI for this shape of workflow |
| **Config** | TOML | YAML, JSON, env vars | Native `tomllib` support in Python 3.11+ |
| **Testing** | pytest | unittest, nose | Simpler fixtures and assertion syntax |
| **Deployment** | Streamlit Cloud | AWS, Heroku, Docker | Free and zero infrastructure to manage for a single-instance demo |

---

*Last Updated: June 2026*
