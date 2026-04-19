# Swiss Voting Booklets Multilingual Alignment Pipeline

## Overview

This repository implements an end-to-end pipeline for constructing a multilingual parallel corpus from Swiss federal voting booklets. The system performs automated document acquisition, optical character recognition (OCR), text normalization, and multilingual paragraph alignment across Switzerland's national languages.

The pipeline is designed for reproducible large-scale processing and can be executed through a single entrypoint script.

---

## Pipeline Execution

The complete workflow can be executed with:

```bash
python monolithic_pipeline.py
```

This command runs all processing stages sequentially:

1. Download voting booklets
2. OCR extraction
3. OCR cleaning
4. Multilingual alignment

### Running Individual Stages

Specific stages may also be executed independently:

```bash
python monolithic_pipeline.py download
python monolithic_pipeline.py ocr
python monolithic_pipeline.py clean
python monolithic_pipeline.py align
```

Multiple stages can be chained:

```bash
python monolithic_pipeline.py clean align
```
---

## Requirements

### Python

Python 3.10 or newer is recommended.

### Installation

Clone the repository:

```bash
git clone <repository-url>
cd <repository>
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Additionally, `pdf2image` requires Poppler.

**Ubuntu**
```bash
sudo apt install poppler-utils
```

**macOS**
```bash
brew install poppler
```

### Environment Variables

Create a `.env` file in the project root:

```
OPENAI_API_KEY=your_api_key_here
```

Environment variables are loaded automatically.

---

## Processing Stages

### 1. Download

Voting booklets are automatically downloaded from the online collection of voting booklets of the federal government and stored in:

```
voting booklets/
```

### 2. OCR Extraction

The OCR stage performs:

- automatic PDF discovery
- page rotation correction
- PDF-to-image conversion
- vision-model text extraction
- paragraph reconstruction

Output: `gemini-ocr/`

> Existing OCR results are reused automatically.

### 3. OCR Cleaning

Cleaning removes typical OCR artifacts:

- invisible Unicode characters
- layout metadata
- bullet symbols
- hyphenation errors
- spacing inconsistencies

Output: `cleaned_ocr_txt/`

### 4. Multilingual Alignment

Paragraph alignment is performed using [Sentence SwissBERT](https://huggingface.co/jgrosjean-mathesis/sentence-swissbert) embeddings.

**Procedure:**

1. Paragraphs are embedded independently per language
2. Cosine similarity matrices are computed
3. Position-aware dynamic programming aligns segments
4. Many-to-many paragraph mappings are supported

**Supported languages:**

| Code | Language |
|------|----------|
| `de` | German   |
| `fr` | French   |
| `it` | Italian  |
| `rm` | Romansh  |

Output: `parallel_corpus/mono_aligned_<YEAR>.jsonl`

**Example entry:**

```json
{
  "de": "Bundesrat und Parlament empfehlen den Stimmberechtigten, am 11. März 2007 wie folgt zu stimmen:",
  "fr": "Le Conseil fédéral et le Parlement vous recommandent de voter, le 11 mars 2007:",
  "it": "Consiglio federale e Parlamento vi raccomandano di votare come segue l’11 marzo 2007:",
  "rm": "Il cussegl federal ed il parlament recumondan a las votantas ed als votants da votar ils 11 da mars 2007 sco suonda:"
}
```

---

## Incremental Processing

The pipeline supports safe re-execution:

- completed OCR steps are skipped
- intermediate artifacts are reused
- processing resumes automatically

---

## Configuration

### Voting Dates

Edit in `monolithic_pipeline.py`:

```python
DATES_TO_PROCESS = [
    "12.06.1977",
    "01.12.1985",
    "11.03.2007"
]
```

### Alignment Parameters

Key parameters in `align_with_ssb.py` include:

- maximum merge size
- positional weighting
- embedding batch size

---

## Output

Final aligned datasets are stored in:

```
parallel_corpus/
```

Each file contains aligned multilingual paragraph segments suitable for NLP training, corpus linguistics, or translation research.

---

## Results

The pipeline was evaluated on three voting dates (12.06.1977, 01.12.1985, 11.03.2007).

### Corpus Statistics

| Year  | DE Tokens | FR Tokens | IT Tokens | RM Tokens | Paragraphs |
|-------|-----------|-----------|-----------|-----------|------------|
| 1977  | 4,016     | 4,997     | 4,492     | –         | 209        |
| 1985  | 1,621     | 2,199     | 1,827     | 2,042     | 107        |
| 2007  | 2,017     | 2,612     | 2,431     | 2,785     | 179        |
| **Total** | **7,654** | **9,808** | **8,750** | **4,827** | **495** |

### OCR Quality (Gemini 2.5 Flash Lite)

| Year | DE WER | FR WER | IT WER | RM WER |
|------|--------|--------|--------|--------|
| 1977 | 0.046  | 0.045  | 0.032  | –      |
| 1985 | 0.012  | 0.018  | 0.016  | 0.017  |
| 2007 | 0.035  | 0.072  | 0.039  | 0.057  |

### Alignment Quality (Sentence SwissBERT)

German (`de`) served as the grounding language; all other languages were aligned against it.

| Year | Lang | F1     | CER    |
|------|------|--------|--------|
| 1977 | FR   | 0.921  | 0.076  |
| 1977 | IT   | 0.919  | 0.068  |
| 1985 | FR   | 0.953  | 0.021  |
| 1985 | IT   | 1.000  | 0.000  |
| 1985 | RM   | 0.981  | 0.080  |
| 2007 | FR   | 0.981  | 0.010  |
| 2007 | IT   | 0.986  | 0.004  |
| 2007 | RM   | 0.953  | 0.027  |


