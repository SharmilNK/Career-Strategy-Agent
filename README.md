# Career-Strategy-Agent
Upload your resume · Set your target role· Get missing skills and a rewritten resume

A multi-tool LLM agent that analyses your resume against a target role,
identifies skill gaps, rewrites your resume, and produces a Word document
of missing skills.

---

## What it does

1. **Parses your resume** — extracts skills, roles, and education from a PDF
2. **Fetches job requirements** — from a real job posting URL, or O*NET as fallback
3. **Analyses the gap** — semantically matches your skills to the role requirements
4. **Rewrites your resume** — aligns language and emphasis to the target role
5. **Generates a Word doc** — lists every missing skill with a suggested action

---

## Architecture

```
User (Streamlit UI)
       │
       ▼
  agent.py  ←── hand-written plan → execute → observe loop
       │
       ├── Tool 1: parse_resume()          (PyMuPDF + LLM)
       ├── Tool 2: fetch_job_requirements() (BeautifulSoup + O*NET + LLM)
       ├── Tool 3: analyze_gap()           (LLM semantic matching)
       ├── Tool 4: rewrite_resume()        (LLM)
       └── Tool 5: generate_docx()         (python-docx)
```

**LLM** — `tinyllama` running locally via Ollama.

---

## Project structure

```
consensus/
├── setup_colab.ipynb   # Colab notebook: installs Ollama, writes files, launches UI
├── agent.py            # Agent loop (plan → execute → observe)
├── tools.py            # All 5 tool functions
├── app.py              # Streamlit UI
├── eval.py             # Quantitative evaluation script
├── eval_data/          # Place test PDFs here for evaluation
│   ├── resume_01.pdf
│   ├── resume_02.pdf
│   └── resume_03.pdf
└── README.md
```

---

## Running on Google Colab (recommended)

1. Open `setup_colab.ipynb` in Google Colab
2. Run all cells in order — this will:
   - Install Ollama and pull `tinyllama`
   - Install Python dependencies
   - Write source files to `/content/consensus/`
   - Launch Streamlit and expose it via ngrok
3. Paste your ngrok token when prompted (free at https://ngrok.com)
4. Open the ngrok URL in your browser

---

## Running locally

```bash
# 1. Install Ollama (https://ollama.com)
curl -fsSL https://ollama.com/install.sh | sh
ollama pull tinyllama

# 2. Install Python dependencies
pip install streamlit pymupdf python-docx beautifulsoup4 requests

# 3. Run the UI
streamlit run app.py
```
---
Webpage: NgrokTunnel: "https://odis-polyprotic-bethel.ngrok-free.dev" -> "http://localhost:8501" Open that URL in your browser.
---

## Running the evaluation

```bash
# Add test PDFs to eval_data/
mkdir eval_data
# copy resume_01.pdf, resume_02.pdf, resume_03.pdf into eval_data/

python eval.py
# Results printed to console and saved to eval_results.json
```

### Evaluation metrics

| Tool | Metric | Method |
|------|--------|--------|
| Resume parser | Precision / Recall / F1 | Compare extracted skills to manually labelled ground truth |
| Job requirements | Precision / Recall / F1 | Compare fetched skills to manually labelled required skills |
| Gap analysis (missing) | F1 | Compare identified gaps to ground truth gaps |
| Gap analysis (matched) | F1 | Compare matched skills to ground truth matches |
| Resume rewriter | ATS keyword match rate | % of target role keywords present in rewritten resume |

---

## Tools detail

### Tool 1 — Resume parser (`parse_resume`)
- Input: PDF bytes
- Process: PyMuPDF extracts raw text → LLM returns structured JSON
- Output: `{name, skills, roles, education, summary}`

### Tool 2 — Job requirements fetcher (`fetch_job_requirements`)
- Input: job posting URL (optional) + role title
- Process: BeautifulSoup scrapes posting → LLM extracts skills; O*NET fallback if URL fails
- Output: `{role, required_skills, preferred_skills, source}`

### Tool 3 — Gap analyser (`analyze_gap`)
- Input: resume_data + job_data
- Process: LLM performs semantic skill matching (not string equality)
- Output: `{matched_skills, missing_skills, overlap_score, recommendation}`

### Tool 4 — Resume rewriter (`rewrite_resume`)
- Input: resume_data + gap_data + job_data
- Process: LLM rewrites resume using job-aligned language without fabricating experience
- Output: plain text rewritten resume

### Tool 5 — Document generator (`generate_docx`)
- Input: missing_skills + preferred_gaps + target_role
- Process: python-docx builds formatted Word document; LLM generates per-skill action suggestions
- Output: `.docx` file bytes

---

## Limitations

- `tinyllama` is optimised for speed on GPU; larger models will improve output quality
- LinkedIn and some job boards block scraping; paste the URL and it will fall back to O*NET if needed
- Evaluation requires manually labelled test resumes in `eval_data/`


---

## Dependencies

```
streamlit
pymupdf
python-docx
beautifulsoup4
requests
```


