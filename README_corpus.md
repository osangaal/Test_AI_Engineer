# Corpus: MineSight DD

Unlike a test that hands you a Drive folder with static PDFs, this test expects the **corpus to be built automatically** from the investor relations page of **Mineros S.A.** (Colombia).

---

## Single corpus source

**Mineros financial reports page**:
> https://www.mineros.com.co/es-co/inversionistas/informes-financieros

The page groups reports by **year (2021-2026)** and by **quarter (Q1, Q2, Q3, Q4)**, plus the **Annual Information Form** for each year.

The candidate must programmatically download **all available PDFs** from this URL. Full details in `CASE_BRIEF.md`, section **Block 1: Automated acquisition pipeline**.

---

## Processing scope

While the download covers all years, processing (extraction + consolidation + dashboard) runs **only on the four quarters of 2025**. The code must be designed to process any year by changing a parameter. This evaluates engineering judgment (no hardcoding).

---

## Language

Mineros' reports are in **Spanish**. The candidate must configure extraction tooling and, if using LLMs, models that properly handle Spanish.

---

## Notes for the internal evaluator

### Before the first evaluation

1. **Manually visit the URL** and verify the page is live, accessible, and that the PDF links work. If the page was redesigned, update `CASE_BRIEF.md` accordingly.
2. **Manually download the four 2025 quarterly PDFs** (Q1-Q4) and read them to form your own view on which KPIs are reasonable to expect.
3. **Identify 5 specific table figures** (e.g., gold production in Q1 2025, AISC in Q3 2025) that will be used to verify the precision of the candidate's extraction. See `_internal/CALIBRATION_QUERIES.md`.
4. **Document the version of the site** (visit date) in this file.

### If the page has anti-bot or dynamic JS

- If **you** could not easily download the PDFs with a simple script (`requests` plus `BeautifulSoup`), then **do not disqualify** a candidate who documents the same block and downloads manually.
- What should be penalized: a candidate who attempts nothing (no code, no documentation).

### Corpus verification log

| Visit date | Verified by | Years visible on the page | Notes / site changes |
|---|---|---|---|
| ____ | ____ | 2021-2026 (as of 2026-05-25) | fill before first use |

---

## Why Mineros and not an international company?

- **Regional realism**: aligned with the market Sun Valley operates in.
- **Language**: Sun Valley analysts work in Spanish; the system should function in Spanish.
- **Manageable size**: the Mineros corpus is tractable in 4 days, unlike majors with thousands of pages.
- **Real public data**: verifiable, non-invented figures.
- **Clear structure**: the page organizes reports by year and quarter, ideal for evaluating a structured download pipeline.
