# Private Company Distress Radar (UK Companies House)

This project is a small research / tooling sandbox to spot **UK private companies** that might be (there will be 5 stages, more coming):

- at **risk of financial distress**, or  
- **boring but reliably profitable**

using only **public data from Companies House**.

The workflow is built in **Jupyter notebooks** and uses the official Companies House API plus some simple feature engineering and heuristics.

---

## High-Level Workflow

1. **Stage / Week 1 – Foundations (API + data collection)**  
   - Connect to Companies House API  
   - Fetch:
     - Company **profile**
     - **Filing history**
     - **Charges** (security over assets)  
   - Cache raw JSON responses to disk so you don’t hammer the API every time.

2. **Stage / Week 2 – Data model & feature engineering (signals)**  
   - Design a **feature schema** for each company  
   - Parse raw JSON → **tabular features**  
   - Create first-pass **distress** and **boring-profitable** signals  
   - Build and export **ranked watchlists**.

---

## Repository Structure

Roughly:

```text
.
├─ README.md
├─ .gitignore
├─ .env                      # Companies House API key (NOT committed)
├─ data/
│  ├─ companies_seed.csv     # Seed list of company_numbers
│  ├─ raw/
│  │  ├─ profiles/           # {company_number}.json from /company endpoint
│  │  ├─ filings/            # {company_number}.json from /filing-history
│  │  └─ charges/            # {company_number}.json from /charges
│  └─ processed/
│     ├─ company_features_week2.csv
│     ├─ company_features_with_signals_v1.csv
│     ├─ distress_watchlist_v1.csv
│     └─ boring_profitable_watchlist_v1.csv
├─ Week 1_Foundations.ipynb  # Stage/Week 1 – API + caching
└─ Stage 1 + Stage 2.ipynb   # Consolidated Week 2 pipeline

