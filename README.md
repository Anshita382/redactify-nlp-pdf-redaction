# 🔒 PDF Redactor — NLP-Powered Sensitive Information Censoring

A command-line tool that automatically detects and redacts sensitive information from PDF documents using **spaCy NER**, **coreference resolution**, and **manual name matching**. Built as part of CIS6930 (Spring 2025) at the University of Florida.

## Overview

Sensitive documents often contain personally identifiable information (PII) — names, organizations, locations — that must be removed before sharing. This tool automates that process by combining three complementary redaction strategies:

| Strategy | Flag | What It Catches |
|----------|------|-----------------|
| **Named Entity Recognition** | `--entities` | People, organizations, and locations detected by spaCy's NER pipeline |
| **Coreference Resolution** | `--coref` | Pronouns and references that resolve back to sensitive entities (e.g., "he", "the CEO") |
| **Manual Name Matching** | `--names` | Specific names you want redacted regardless of NER output |

Redacted regions are replaced with solid black bars in the output PDF, and every redaction is logged to a stats file for auditability.

## Demo

📹 [Watch the walkthrough →](https://www.youtube.com/watch?v=T352t90wg1I)

## Installation

**Prerequisites:** Python 3.10+, [uv](https://github.com/astral-sh/uv)

```bash
# Clone the repository
git clone https://github.com/Anshita382/cis6930sp25-project2.git
cd cis6930sp25-project2/cis6930sp25-project2

# Install dependencies and download spaCy models
uv add pip
uv run -m spacy download en_core_web_sm
uv run -m spacy download en_core_web_trf
```

## Usage

```bash
uv run python main.py \
  --input "resources/*.pdf" \
  --output redacted/ \
  --names Bill Carter \
  --entities \
  --coref \
  --stats redacted/stats.tsv
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--input` | ✅ | Glob pattern(s) for input PDF files |
| `--output` | ✅ | Directory for redacted output PDFs |
| `--names` | | One or more names to redact by exact match |
| `--entities` | | Enable spaCy NER to detect PERSON, ORG, and GPE entities |
| `--coref` | | Enable coreference resolution via API |
| `--stats` | | Path to write a TSV log of all redactions |

### Example: Redact only specific names
```bash
uv run python main.py --input "docs/report.pdf" --output clean/ --names "Jane Doe"
```

### Example: Full NER + coreference pipeline
```bash
uv run python main.py --input "docs/*.pdf" --output clean/ --entities --coref --stats clean/audit.tsv
```

## How It Works

```
Input PDF(s)
    │
    ├─► Extract text per page (PyMuPDF)
    │
    ├─► Name matching ──────────► tokens to redact
    ├─► spaCy NER (PERSON/ORG/GPE) ──► tokens to redact
    ├─► Coreference API ────────► tokens to redact
    │
    ├─► Search & apply redaction annotations
    │
    └─► Save redacted PDF + append to stats log
```

### Stats Output

The stats file is a tab-separated log with one row per redaction:

```
File                    Page    Token           Length  Type
resources/test1in.pdf   0       Bill Carter     11      PERSON
resources/test2in.pdf   0       NASA            4       ORG
resources/test2in.pdf   0       Washington DC   13      GPE
```

## Project Structure

```
cis6930sp25-project2/
├── main.py              # Core redaction pipeline
├── pyproject.toml       # Project metadata and dependencies
├── resources/           # Sample input PDFs for testing
│   ├── test1in.pdf
│   ├── test2in.pdf
│   └── test3in.pdf
├── redacted/            # Output directory with redacted PDFs + stats
│   └── stats.tsv
└── tests/
    ├── tests_ner.py     # Validates spaCy entity extraction
    ├── tests_coref.py   # Validates coreference API connectivity
    └── tests_token.py   # Validates name-based redaction on sample PDFs
```

## Running Tests

```bash
uv run pytest tests/
```

> **Note:** `tests_coref.py` requires access to the UF internal GPU cluster API and will fail on external networks.

## Tech Stack

- **[PyMuPDF (fitz)](https://pymupdf.readthedocs.io/)** — PDF text extraction and redaction annotation
- **[spaCy](https://spacy.io/)** — Named Entity Recognition with `en_core_web_sm`
- **[Requests](https://docs.python-requests.org/)** — HTTP client for the coreference resolution API
- **[uv](https://github.com/astral-sh/uv)** — Fast Python package and project manager

## Known Limitations

- The `--coref` flag relies on a REST API hosted on UF's internal GPU cluster — it will not work outside the UF network.
- Only works with PDFs that contain selectable (non-scanned) text; image-based PDFs are not supported.
- spaCy models must be downloaded before using `--entities`.

## Author

**Anshita Rayalla** 

University of Florida · CIS6930 Spring 2025
