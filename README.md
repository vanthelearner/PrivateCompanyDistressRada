# Private Company Distress Radar v1 (UK Companies House)

This project is a small research / tooling sandbox to spot UK private companies that might be:

- **at risk of financial distress**, or  
- **boring but reliably profitable**

using only public data from the **UK Companies House** API.

The workflow is built in **Jupyter notebooks** and uses:

- the official **Companies House API**
- some lightweight **feature engineering**
- a few simple, configurable **heuristics** for scoring and ranking companies

> **Disclaimer:** This is a personal research project and not investment advice.  
> The signals are heuristic and incomplete and should not be used as the sole basis for any financial decision.

---

## High-Level Workflow (3 Stages)

### Stage 1 – Foundations (API + data collection)

- Connect to **Companies House API** using a personal API key (`.env`)
- Use `/advanced-search/companies` to build a **seed universe**:
  - Filter by **SIC codes** (e.g. B2B services cluster: `70229`, `70221`, `73110`)
  - Filter by **status** (e.g. only `"active"` companies)
- Save the seed list to `data/companies_seed.csv`
- For each seed company, fetch:
  - Company **profile** (`/company/{number}`)
  - **Filing history** (`/company/{number}/filing-history`)
  - **Charges** / security over assets (`/company/{number}/charges`)
- Cache raw JSON responses under `data/raw/` for inspection.

---

### Stage 2 – Data model & feature engineering (signals)

- Define a simple **feature schema per company**, including:

  - Basic:  
    `company_number`, `name`, `age_years`, `status`, `sic_codes`

  - Filing behaviour:  
    `num_accounts_last_10y`, `num_late_accounts`,  
    `last_accounts_date`, `overdue_accounts_flag`

  - Capital structure (charges):  
    `num_charges_total`, `num_charges_outstanding`,  
    `num_charges_satisfied`, `has_floating_charge_flag`, `last_charge_date`

- Parse raw JSON from Companies House into a **tabular feature table**:
  - Saved as `data/company_features_week2.csv`

- Add simple **heuristic signals**:

  - `signal_potential_distress` – patterns like:
    - overdue accounts
    - repeated late filings
    - many outstanding charges
    - very young company with multiple charges

  - `signal_boring_profitable_like` – patterns like:
    - older companies
    - consistent accounts filing over ~10 years
    - no late filings
    - not currently overdue
    - low/moderate outstanding charges

- Compute initial **raw scores** (0–3) for both dimensions:
  - `distress_score_v1`
  - `boring_profitable_score_v1`

- Export enriched features and watchlists:
  - `data/processed/company_features_with_signals_v1.csv`
  - `data/processed/distress_watchlist_v1.csv`
  - `data/processed/boring_profitable_watchlist_v1.csv`

All thresholds (e.g. “young = <5 years”, “boring if ≥10 years + ≥7 accounts in last 10y”) are set as **global variables at the top of the notebook**, so they can be tuned easily.

---

### Stage 3 – End-to-end distress radar workflow (v1 product)

Implemented in **`Stage 1 + Stage 2 + Stage 3.ipynb`**.

This notebook is a **from-scratch pipeline** that runs everything in one place:

1. **Build seed universe** via `/advanced-search/companies`
   - Controlled by global config:
     - `SEED_SIC_CODES` – list of SIC codes to filter on  
     - `SEED_COMPANY_STATUS` – typically `"active"`  
     - `SEED_SIZE` – number of companies to pull (e.g. `600`)  
     - `SEED_LOCATION` – optional location filter (e.g. `"london"`)

2. **Feature extraction directly from the API**
   - For each company in `companies_seed.csv` (optionally capped by `MAX_COMPANIES_FEATURES`):
     - Fetch profile, filings, and charges
     - Compute engineered features
   - Save per-company feature table to:
     - `data/company_features_week2.csv`

3. **Signals & raw scores**
   - Apply the same heuristic logic from Stage 2:
     - `signal_potential_distress`
     - `signal_boring_profitable_like`
   - Raw scores:
     - `distress_score_v1` (0–3)
     - `boring_profitable_score_v1` (0–3)
   - Save enriched table:
     - `data/processed/company_features_with_signals_v1.csv`

4. **Score scaling (0–100) & buckets**
   - Scale raw scores to **0–100** using simple min–max scaling:
     - `distress_score_0_100`
     - `boring_profitable_score_0_100`
   - Bucket into **Low / Medium / High** bands:
     - `distress_bucket`
     - `boring_profitable_bucket`
   - Save:
     - `data/processed/company_scores_v1.csv`

5. **Top lists & watchlists**
   - Build and save ranking views (configurable via `TOP_N_DISTRESSED` / `TOP_N_BORING`):
     - `data/processed/top_20_distressed_v1.csv`
     - `data/processed/top_20_boring_profitable_v1.csv`
   - Also save full watchlists for flagged companies:
     - `data/processed/distress_watchlist_v1.csv`
     - `data/processed/boring_profitable_watchlist_v1.csv`

6. **Basic visualisations**
   - Histograms:
     - Distribution of `distress_score_0_100`
     - Distribution of `boring_profitable_score_0_100`
   - Scatter plots:
     - `distress_score_0_100` vs `age_years`
     - `boring_profitable_score_0_100` vs `age_years`
   - These give a quick feel for:
     - how scores are distributed across the universe
     - whether age and the signals line up with intuition.

All main knobs (sample size, SIC codes, distress thresholds, boring thresholds, bucket cut-offs, top-N sizes) live in a **single “USER–TUNABLE CONFIG” block at the top** of `00_distress_radar_workflow.ipynb`.

---

## Repository Structure

```text
.
├─ README.md
├─ .gitignore
├─ .env                        # Companies House API key (NOT committed)
├─ data/
│  ├─ companies_seed.csv       # Seed list of company_numbers from advanced search
│  ├─ company_features_week2.csv
│  ├─ raw/
│  │  ├─ profiles/             # {company_number}.json from /company
│  │  ├─ filings/              # {company_number}.json from /filing-history
│  │  └─ charges/              # {company_number}.json from /charges
│  └─ processed/
│     ├─ company_features_with_signals_v1.csv
│     ├─ company_scores_v1.csv
│     ├─ distress_watchlist_v1.csv
│     ├─ boring_profitable_watchlist_v1.csv
│     ├─ top_20_distressed_v1.csv
│     └─ top_20_boring_profitable_v1.csv
├─ Week 1_Foundations.ipynb    # Stage 1 – API basics + manual exploration
├─ Stage 1 + Stage 2.ipynb     # Early combined pipeline (dev / scratchpad)
└─ Stage 1 + Stage 2 + Stage 3.ipynb
                               # Final from-scratch Stage 1–3 pipeline
