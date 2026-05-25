# Technical Test: AI Engineer

**Sun Valley Investment. Position: AI Engineer, Development and Implementation Specialist**

Welcome. This document describes the technical case you need to solve. Read it in full before starting.

---

## 1. Business context

Sun Valley Investment is a fund that invests in mining assets. Every opportunity requires reviewing dozens of quarterly and annual reports to build a consolidated view of the company's financial and operational KPIs. Today an analyst does this manually, copying figures from PDFs into Excel. It is a slow, error-prone process.

Your mission: build **MineSight DD**, an end-to-end system that automates the extraction, consolidation, and visualization of key indicators from a mining company's public reports.

The target company is **Mineros S.A.** (Colombia), chosen for its public report availability and regional relevance. Reports are published at:

> https://www.mineros.com.co/es-co/inversionistas/informes-financieros

The page organizes reports by year (2021-2026) and by quarter (Q1-Q4), plus annual information forms.

> **Note**: Mineros' reports are in **Spanish**. Your extraction tooling and any LLM you use must handle Spanish properly.

---

## 2. The system to build (3 blocks)

### Block 1: Automated acquisition pipeline

A script that **downloads all available PDFs** from the Mineros page, **without filtering by year** (all visible years). It must be:
- **Idempotent**: do not re-download what is already on disk
- **Robust**: retries with backoff, identifiable User-Agent, respects `robots.txt`
- **Well organized**: clear on-disk structure (e.g., `data/raw/2025/Q1/...`)
- **Traceable**: logs of what was downloaded, from which URL, and file size

**If you hit site blockers** (anti-bot, dynamic JS, captcha) that you cannot reasonably resolve: document the block in your README, show the code you tried, and download manually. **You will not be penalized** if the block is real and properly documented.

### Block 2: KPI extraction and consolidation

On the downloaded PDFs, you must:

1. **Detect and extract all tables** in the reports. We expect you to use AI where it adds value (LLMs for parsing complex tables, vision models for image-based tables, etc.).
2. **Propose a set of financial and operational KPIs** you consider relevant to evaluate Mineros as an investment. You have **full freedom** to decide which ones. We expect you to justify your choices.

   **Why we give you this freedom**: what we want to evaluate is your ability to **face a new business problem and structure the solution yourself**: which indicators matter, how to research them, how to defend them. Day to day at Sun Valley you will solve similar problems for different business units across the organization; each time you will need to understand the context, propose the approach, and justify it. Here we want to see that **proactivity, creativity, and research**.

   **On the use of LLMs (Claude, Gemini, ChatGPT) for ideation**: it is allowed and welcome to use them to understand the mining investment problem, explore which KPIs are industry-standard, or validate your reasoning. **If you do use them for that purpose**, include the prompts you used in a section of your report (`docs/REPORT.md`, "Use of LLMs in ideation"). This does not penalize you, on the contrary: we are interested in seeing **how you use these tools as an amplifier, not as a replacement for judgment**.

3. **Consolidate those KPIs into a single normalized table** (CSV, Parquet, SQLite. Your choice of format).

**Processing scope**: run block 2 **only on the four quarters of 2025** (Q1, Q2, Q3, Q4). This keeps the work manageable.

**IMPORTANT on reproducibility**: your code **must NOT** be hardcoded to 2025. It must be designed to process **any year** (or multiple years) by changing a parameter. In your README, explain how to run it for, for example, the four quarters of 2024.

> This evaluates engineering judgment: we don't want a one-off script, we want a reusable pipeline.

### Block 3: Visualization for an executive audience

Build a visual interface on top of the consolidated table. **Free format**: Streamlit, Gradio, web dashboard, notebook with plots, FastAPI app with a minimal frontend, whatever you prefer.

The audience is **executive / investment decision-makers**, not technical. The visualizations should:
- Allow seeing the quarterly evolution of KPIs
- Allow comparisons (e.g., Q3 2025 vs Q3 2024 if you have both)
- Highlight trends or alerts relevant to an investment analyst

**Justify your format choice** in the report (why Streamlit over Gradio, why include a given chart, etc.).

---

## 3. Non-functional requirements

- **Logging**: structured, with traceability of every downloaded PDF and every extracted table
- **Error handling**: specific (no generic `except Exception`); retries with backoff on HTTP downloads
- **Modularity**: reasonable separation (`acquisition/`, `extraction/`, `consolidation/`, `dashboard/`)
- **Tests**: at least **3 meaningful unit tests**
- **Typing**: type hints on public functions
- **Reproducibility**: pinned dependencies (`requirements.txt` or `pyproject.toml`); fixed seeds where applicable
- **Parameter-driven configuration**: the year to process must be a parameter, not hardcoded

---

## 4. Optional stretch goals (bonus +15%)

These set top candidates apart. **They are not mandatory** given the tight timeline.

- **S-1. Conversational agent over the data**: an agent (LangChain/LangGraph) with a SQL/lookup tool over the consolidated table that answers natural-language questions ("in which quarter was AISC highest?"). Include citations back to the source PDF or table.
- **S-2. Normalized relational DB**: persist the table in PostgreSQL/SQLite with a defined schema plus a migration script.
- **S-3. Real multi-year processing**: in addition to 2025, process at least one other full year (2024 or 2023) and show a year-over-year comparison in the dashboard.
- **S-4. Real deployment** on Azure / Render / Railway / Streamlit Cloud with a public URL.
- **S-5. Extraction quality evaluation**: golden set of 10+ figures manually verified against what your system extracted, with a precision report.

---

## 5. Deliverables

### 5.1 Private Git repository

Invite as collaborator `{{recruiting-user}}`. Suggested structure:

```
minesight-dd/
├── README.md                    # How to run in <5 min, assumptions, real hours
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml
├── .env.example
├── src/
│   ├── acquisition/             # report downloader
│   ├── extraction/              # parsing + table detection
│   ├── consolidation/           # normalization and KPI table construction
│   ├── dashboard/               # visualization
│   └── config/
├── tests/
├── data/
│   ├── raw/                     # downloaded PDFs (gitignored)
│   └── processed/               # consolidated table
└── docs/
    ├── REPORT.md                # see 5.2
    └── slides.pdf               # see 5.3
```

Your README must include: run instructions, **real hours invested**, assumptions made, **limitations encountered while scraping** (if any), and **how to run the pipeline for another year**.

### 5.2 Report (`docs/REPORT.md`)

Required sections (2-4 pages):

1. **Executive summary** (1 paragraph): is it production-ready? Yes/No and why.
2. **Acquisition pipeline**: how many PDFs were downloaded, problems encountered, how they were resolved.
3. **Proposed KPIs**: list of the indicators you consolidated, **each one justified from the perspective of a mining investment analyst**.
4. **Extraction quality**: how you verified that the extracted figures are correct. If you did a quantitative evaluation (golden set), include the results.
5. **Technical decisions and trade-offs**: extraction stack, LLM model choice if applicable, normalization across PDFs (column names may vary between quarters).
6. **Known limitations**: what does NOT work yet.
7. **Roadmap**: what you would prioritize with one additional week.
8. **Use of LLMs in ideation** (if applicable): if you used Claude, Gemini, ChatGPT or similar to research the problem or shape your KPI choices, include the prompts here.

### 5.3 Presentation (PDF slides)

Between **6 and 10 slides**. Minimum structure:

1. Problem framed in **investment language**, not technical
2. High-level architecture (diagram: download → extraction → consolidation → dashboard)
3. Proposed KPIs with justification (1 slide)
4. Dashboard demo (screenshots of the most important visualizations)
5. Technical decisions and trade-offs (1 slide)
6. Limitations and roadmap
7. **Estimated operating cost** if the system ran monthly

---

## 6. Evaluation rubric (100 pts + 15 bonus)

We share the weights so you can prioritize your effort:

| Competency | Weight |
|---|---|
| A. Acquisition pipeline (robust download and organization) | **15 pts** |
| B. Table extraction (precision and intelligent use of AI) | **25 pts** |
| C. Proposed KPIs, business judgment, and normalization | **25 pts** |
| D. Dashboard / visualization (usefulness for executive audience) | **15 pts** |
| E. MLOps (Docker, packaging, multi-year reproducibility) | **10 pts** |
| F. Code quality (modularity, tests, logs) | **5 pts** |
| G. Communication (report and slides) | **5 pts** |
| Stretch goals (S-1 to S-5) | **+15 pts** |

---

## 7. Rules

- **AI-assisted coding allowed** (Copilot, Cursor, ChatGPT). **However**, you must be able to defend every technical decision in the synchronous session.
- **Document your assumptions**. If something is ambiguous, decide yourself and justify it. This includes **which KPIs you chose and why**.
- **Be honest about time**: report real hours in your README.
- **Free LLM stack**: OpenAI, Azure OpenAI, Anthropic, open-source models. API costs are on you.
- **Responsible scraping**: respect `robots.txt`, use an identifiable User-Agent, reasonable delays between requests.

---

## 8. Timeline and delivery

- **Estimated effort**: 8-12 hours of effective work
- **Deadline**: **3 calendar days** from receipt of this document
- **Delivery method**: email the recruiter with the repo link plus slides PDF attached (or a public link)

---

## 9. After delivery

We will schedule a **30-minute synchronous session**:
- **10 min**: you present the system (working dashboard and key decisions)
- **20 min**: technical Q&A on your KPI choices, code, and trade-offs

This session is the main filter against LLM abuse and to verify real depth.

---

## 10. Contact

**Operational** questions (problems with the Mineros link, delivery format): **ogaspar@oceloteminerals.com**

**Technical** questions: we don't answer them, they are part of the evaluation.

**Good luck.**

Oscar Gaspar Alvarez
Lead AI · Sun Valley Investment
