# DEET Instant Registration System

> Upload a resume. Get a complete DEET profile in seconds — plus a live board of verified job vacancies scraped directly from employer career pages.

Built for the **DEET Hackathon · February 2026**

---

## What it does

Most job portals make applicants re-type everything from their resume. This fixes that.

- **Resume → Profile**: Upload a PDF, DOCX, or image. The app extracts your name, contact details, education, skills, work experience, and projects using OCR + NLP, then shows you a confidence-scored editable preview before saving.
- **Job Discovery**: A background scheduler scrapes verified employer career pages every 6 hours, deduplicates listings, classifies them by category, and surfaces them on a filterable job board.

---

## Demo

<!-- Drop a GIF here once you have one -->
> 🎥 *Demo GIF coming soon*

---

## Tech stack

| Layer | Tools |
|---|---|
| Backend | Flask, SQLite, APScheduler |
| OCR | pdfplumber (native PDF), pytesseract (scanned/image) |
| NLP | spaCy (NER), regex, python-dateutil |
| ML | scikit-learn — TF-IDF + Logistic Regression |
| Scraping | BeautifulSoup4, requests |
| Verification | python-whois, DNS resolution |
| Testing | pytest |

---

## Getting started

**Requirements:** Python 3.10+ — [Download here](https://www.python.org/downloads/) *(check "Add to PATH" during install)*

### Windows

```bash
# First-time setup
setup.bat

# Start the server
run.bat
```

### Mac / Linux

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m spacy download en_core_web_sm
python app.py
```

Then open **http://localhost:5000** in your browser.

---

## How the pipelines work

### Resume → DEET profile

```
Upload (PDF / DOCX / Image)
        │
        ▼
   OCR Engine
   ├── Native PDF  → pdfplumber
   ├── Scanned PDF → pytesseract
   └── Image       → PIL + pytesseract
        │
        ▼
   NLP Extractor
   ├── Section detection  (regex headers)
   ├── Named Entity Recognition  (spaCy)
   ├── Pattern matching  (email, phone, LinkedIn, GitHub)
   ├── Skills extraction  (keyword dictionary)
   └── Date normalization  (dateutil)
        │
        ▼
   Confidence Scorer  →  HIGH / MEDIUM / LOW badge per field
        │
        ▼
   Editable preview  →  User confirms  →  Saved to DB
```

**Extracted fields:** Name · Email · Phone · Address · LinkedIn · GitHub · Summary · Education · Skills · Certifications · Experience · Projects

---

### Job vacancy discovery

```
APScheduler (every 6h) or manual trigger
        │
        ▼
   Scraper  (BeautifulSoup per career page)
        │
        ▼
   Deduplicator
   ├── Exact  →  SHA-256 hash of title + company + location
   └── Near-duplicate  →  TF-IDF cosine similarity ≥ 0.85
        │
        ▼
   Classifier  →  TF-IDF + Logistic Regression  (keyword fallback)
        │
        ▼
   Employer Verifier
   ├── DNS resolution   +0.25
   ├── HTTP reachable   +0.25
   ├── HTTPS            +0.20
   └── Domain age ≥ 1yr +0.30
        │
        ▼
   SQLite  →  Job board
```

---

## API reference

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Landing page |
| `GET` | `/jobs` | Job discovery board |
| `GET` | `/preview` | DEET profile editor |
| `POST` | `/api/resume/upload` | Upload resume → extracted profile JSON |
| `POST` | `/api/resume/submit` | Save confirmed profile |
| `GET` | `/api/resume/profiles` | List all saved profiles |
| `GET` | `/api/jobs/list` | Paginated jobs (filter by category, location, search) |
| `POST` | `/api/jobs/crawl` | Trigger immediate crawl |
| `POST` | `/api/jobs/add` | Manually add a job posting |
| `GET` | `/api/dashboard/stats` | Platform statistics |
| `GET` | `/api/health` | Health check |

---

## Configuration

Edit `config.py`:

| Setting | Default | Description |
|---|---|---|
| `CRAWL_INTERVAL_HOURS` | `6` | Auto-crawl frequency |
| `DEDUP_COSINE_THRESHOLD` | `0.85` | Near-duplicate sensitivity |
| `MIN_DOMAIN_AGE_DAYS` | `180` | Employer trust threshold |
| `TARGET_CAREER_PAGES` | `[]` | Career page URLs to scrape |

### Adding a career page

```python
# config.py
TARGET_CAREER_PAGES = [
    {
        "company":           "Acme Corp",
        "url":               "https://acme.com/careers",
        "job_selector":      ".job-listing",
        "title_selector":    ".job-title",
        "location_selector": ".job-location",
        "link_selector":     "a",
    },
]
```

---

## Running tests

```bash
# Run all tests
python -m pytest tests/ -v

# Run a specific module
python -m pytest tests/test_nlp_extractor.py -v
python -m pytest tests/test_api.py -v
```

---

## Project structure

```
├── app.py                     # Flask entry point
├── config.py                  # Configuration
├── database.py                # SQLite helpers
├── requirements.txt
├── modules/
│   ├── ocr_engine.py          # PDF + image text extraction
│   ├── nlp_extractor.py       # NLP pipeline
│   ├── confidence_scorer.py   # Per-field confidence (0.0–1.0)
│   ├── job_scraper.py         # Career page scraper
│   ├── job_classifier.py      # ML job category classifier
│   ├── deduplicator.py        # Duplicate detection
│   ├── employer_verifier.py   # Domain trust scoring
│   └── scheduler.py           # Background crawl scheduler
├── templates/
│   ├── index.html             # Upload UI
│   ├── preview.html           # Profile editor
│   └── jobs.html              # Job board
├── static/
│   ├── css/style.css
│   └── js/
│       ├── main.js
│       ├── preview.js
│       └── jobs.js
└── tests/
    ├── test_nlp_extractor.py
    ├── test_confidence_scorer.py
    ├── test_deduplicator.py
    └── test_api.py
```

---

## Target metrics

| Metric | Target |
|---|---|
| Field extraction F1 | ≥ 85% |
| OCR character error rate | ≤ 5% |
| Job discovery precision | ≥ 90% |
| Duplicate detection rate | ≥ 95% |
| Resume API latency (p95) | ≤ 5s |

---

## License

MIT
