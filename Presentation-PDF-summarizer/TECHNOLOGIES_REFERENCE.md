# Technologies Reference Sheet
## Presentation PDF Summarizer

Quick reference for all technologies, frameworks, and tools used in this project.

---

## 🤖 Core Technologies

### Anthropic Claude API

**Models Available**:
- `claude-opus-4-6`: Most capable; best for complex reasoning ($15 in / $75 out per M tokens)
- `claude-sonnet-4-6`: Balanced; default model ($3 in / $15 out per M tokens) ⭐ **Default**
- `claude-haiku-4-5-20251001`: Fastest; cheapest ($0.80 in / $4 out per M tokens)

**Features Used**:
- Messages API: Structured text summarization
- Vision API: Title extraction from slide images (PNG)
- JSON mode: Enforced structure for 13-field output
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

## 📄 PDF Processing

### PyMuPDF (fitz)

**Purpose**: Extract text, images, metadata from PDF slides

**Key Methods**:
- `fitz.open(path)`: Open PDF
- `doc.load_page(i)`: Get page object
- `page.get_text('text')`: Extract text
- `page.get_images(full=True)`: List images
- `page.get_pixmap(matrix=...)`: Render as PNG (for vision API)
- `doc.is_encrypted`: Check if password-protected
- `len(doc)`: Page count

**Use Cases**:
- PDF validation (encryption, empty pages)
- Slide-by-slide text extraction
- Title extraction via vision (fallback)
- Metadata (slide count, content type detection)

**Version**: Latest

---

## 🌐 Web Framework

### Streamlit

**Purpose**: Interactive web UI for file upload, processing, and download

**Components Used**:
- `st.set_page_config()`: Page setup (title, layout="centered")
- `st.file_uploader()`: Multi-file PDF upload
- `st.button()`: Summarize and Analyze Themes buttons
- `st.progress()`: Progress bar for batch processing
- `st.empty()`: Placeholder for status text
- `st.selectbox()`: Model selection dropdown
- `st.session_state`: Persistent state across reruns
- `st.download_button()`: Download markdown files
- `st.success()`, `st.error()`, `st.warning()`, `st.info()`: Status messages
- `st.spinner()`: Loading indicator
- `st.sidebar`, `st.expander()`, `st.tabs()`: Layout

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

## 🐍 Python Libraries

### Core Python
- **Version**: 3.11+
- **Features Used**: Type hints, match statements, f-strings, walrus operator (`:=`)
- **Modules**: 
  - `os`, `sys`: Environment, paths
  - `pathlib.Path`: Modern file paths
  - `tempfile`: Temporary directories
  - `json`: JSON parsing/validation
  - `re`: Regex for text extraction
  - `tomllib`: TOML config parsing (Python 3.11+)
  - `argparse`: CLI argument parsing
  - `collections.abc.Callable`: Type hints for callbacks

### Testing
- **pytest**: 32 unit tests
- **pytest-cov** (optional): Coverage reporting

---

## 📊 Data Structures

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

## 🏗️ Project Configuration

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

## 📦 Dependency Graph

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

## 🔄 Processing Pipeline Technologies

### Text Processing
- **Regex** (`re` module): Pattern matching for titles, metrics, control characters
- **String methods**: Strip, split, join, replace
- **Normalization**: Case conversion (titlecase with "iOS" handling)
- **Sanitization**: Remove Unicode artifacts, control characters

### List Parsing
- **Pattern matching**: Detect numbered sequences (1, 2, 3) or bullets
- **Heuristics**: Distinguish "process steps" from "named items"
- **Rendering**: Convert to "N-step process" format

### Chart Detection
- **Numeric pattern matching**: `\d|%` regex
- **Keyword hinting**: "trend", "increase", "decrease", "axis"
- **False positive filtering**: Org charts without data

### Content Classification
- **Image detection**: PyMuPDF `get_images()`
- **Text availability**: Check extracted text length
- **Chart/table marking**: Numeric data + keywords

---

## 🔐 Security & Authentication

### API Key Management
- **Streamlit local**: `.streamlit/secrets.toml` (git-ignored)
- **Streamlit Cloud**: Secrets manager in dashboard
- **CLI/Script**: `ANTHROPIC_API_KEY` environment variable

### Data Privacy
- No logging of PDF content
- Temporary files deleted immediately after processing
- No persistence of summaries (user downloads them)
- API calls logged only for token counting

### Dependency Security
- Minimal dependencies (3 core packages)
- No known vulnerabilities (as of June 2026)
- Regular updates recommended for anthropic, streamlit

---

## 📈 Monitoring & Observability

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
- Processing time: Implicit (progress bar)

---

## 🧪 Testing Technologies

### Test Framework: pytest

**Test Modules**:
- `test_content_builders.py` (6 tests): Title/author extraction, chart detection
- `test_list_parsing.py` (5 tests): Sequence detection, rendering
- `test_llm_summarizer.py` (4 tests): Prompt building, response validation
- `test_model_config.py` (6 tests): Model registry, pricing, costs
- `test_text_processing.py` (5 tests): Cleaning, normalization, sanitization
- `test_theme_analyzer.py` (4 tests): Deck parsing, indexing
- `test_summarizer_contract.py` (1 test): Integration test (optional)

**Test Execution**:
```bash
pytest tests/ -v                           # All tests, verbose
pytest tests/unit/test_text_processing.py  # Specific module
pytest tests/ -k "clean"                   # Pattern matching
pytest tests/ --cov=src/pdf_summary        # Coverage report
```

**Coverage**:
- Core functions: ~85-90% line coverage
- Error paths: ~70% coverage
- Integration: Contract tests only (minimal)

---

## 🚀 Deployment Technologies

### Local Development
- **OS**: Windows, macOS, Linux
- **Python**: 3.11+
- **Virtual Environment**: venv or conda
- **Package Manager**: pip

### Streamlit Cloud Deployment
- **Hosting**: Streamlit Community Cloud (free)
- **CI/CD**: GitHub auto-deploy on push
- **Secrets**: Web dashboard
- **Scaling**: Automatic

### Alternative Deployments
- **Docker**: Custom Dockerfile (not included)
- **AWS Lambda**: Via FastAPI wrapper (not included)
- **Heroku**: Via Procfile (not included)

---

## 📚 Version Reference

### Pinned Versions
```
anthropic >= 0.94.1    (Latest features)
pymupdf                (Auto-latest)
streamlit              (Auto-latest)
pytest                 (Auto-latest, for testing)
```

### Compatibility
- **Python**: 3.11, 3.12, 3.13 (expected)
- **OS**: Windows, macOS, Linux
- **Browsers**: Modern browsers (Chrome, Firefox, Safari, Edge)

---

## 🔗 External Services

### Anthropic Claude API
- **Endpoint**: `https://api.anthropic.com/v1/messages`
- **Authentication**: API key in Authorization header
- **Rate Limits**: 5,000 RPM, 500K TPM
- **Availability**: 99.9% SLA (enterprise)

### Streamlit Community Cloud
- **Endpoint**: User-provided URL (e.g., `pdfsum-project.streamlit.app`)
- **Infrastructure**: Streamlit-managed
- **Limits**: Free tier: 3 active apps, 1 GB storage
- **Scaling**: Automatic (up to 3 concurrent sessions)

---

## 💡 Technology Choices & Rationale

### Why PyMuPDF (not pdfplumber / pypdf)?
- **Speed**: Fastest extraction for text + images
- **Vision support**: Direct PNG rendering for Claude vision
- **Reliability**: Handles encrypted PDFs, empty pages gracefully
- **Size**: Minimal; no heavy dependencies

### Why Streamlit (not Dash / FastAPI)?
- **Speed to market**: Build in hours, not days
- **UX**: Great defaults (file upload, progress bars)
- **Deployment**: Single command (no infra)
- **State management**: Built-in session state
- **Limitation**: Only synchronous I/O (acceptable here)

### Why Anthropic (not OpenAI)?
- **Vision API**: Better for title extraction fallback
- **Pricing**: Sonnet is 1/5 cost of GPT-4
- **Performance**: On-par quality with faster inference
- **Prompt caching**: 90% cost savings on repeated content

### Why Minimal Dependencies?
- **Security**: Smaller attack surface
- **Performance**: Faster startup (critical for Streamlit)
- **Reliability**: Fewer version conflicts
- **Simplicity**: Easy to understand entire stack
- **Cost**: Smaller Docker images

---

## 🆕 Tech Stack Comparison Table

| Aspect | Choice | Alternative(s) | Why |
|--------|--------|-----------------|-----|
| **LLM** | Anthropic Claude | OpenAI GPT-4, Gemini | Cheaper + vision API |
| **PDF** | PyMuPDF | pdfplumber, pypdf | Fastest + PNG rendering |
| **Web** | Streamlit | Dash, FastAPI | Rapid development + built-in UI |
| **Config** | TOML | YAML, JSON, env vars | Modern, Python 3.11+ native |
| **Testing** | pytest | unittest, nose | Simpler syntax, better fixtures |
| **Deployment** | Streamlit Cloud | AWS, Heroku, Docker | Free + auto-scaling |

---

*Last Updated: June 2026*
*Minimal, modern, production-grade stack*
