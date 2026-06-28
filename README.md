# Church Financial Management System
### End-to-end financial reporting and analysis for a nonprofit organization

---

## Problem Statement

The treasury team of a Brazilian nonprofit organization managed finances through a fully manual process: bank statements were copied by hand, transactions were typed one by one, and monthly reports were assembled from scratch each cycle.

The result was a process that took **~4 hours per month**, was prone to data entry errors (wrong signs on debit/credit, decimal mismatches), and produced no actionable financial indicators — only a raw list of transactions that required interpretation every time it was consulted.

There was no visibility into cost structure, no way to track financial reserves, and no indicator of how long the organization could sustain operations without new income.

---

## Objectives

- Reduce monthly closing time from ~4 hours to under 30 minutes
- Eliminate manual transcription errors through structured data entry
- Build a reusable classification system for all transactions
- Define and track key financial indicators aligned with organizational needs
- Produce a monthly executive dashboard accessible to non-technical stakeholders

---

## Results

| Metric | Before | After |
|---|---|---|
| Monthly closing time | ~4 hours | ~20 minutes |
| Data entry errors | Frequent | Near zero |
| Financial indicators tracked | 0 | 8+ |
| Reporting format | Manual table | Interactive dashboard |

---

## Data Architecture

```
Bank Statement (PDF/export)
        │
        ▼
[Raw Data Tab — Google Sheets]
  Copy-paste from bank export
        │
        ▼
[Classification Layer]
  Keyword-based auto-classification
  → Category + Subcategory
  → Transaction type (income / expense)
  → Project tag (e.g., Annual Retreat 2026)
        │
        ▼
[Financial Reserve — Manual Input]
  External to transactions (see modeling decisions)
        │
        ▼
[KPI Calculation Layer]
  Aggregations, ratios, period comparisons
        │
        ▼
[Looker Studio Dashboard]
  Connected via Google Sheets data source
  Auto-refreshes on sheet update
```

**Current data coverage:** January – June 2026 (6 months)

---

## Data Modeling Decisions

### 1. Two-level expense classification (Category → Subcategory)

When I assumed the treasury role, expenses were tracked at the category level only (e.g., "Food"). This made it impossible to understand *what kind* of food spending was occurring or whether patterns made sense.

I introduced a subcategory level to increase transparency and enable drill-down analysis:

- `Food` → `Lunch`, `Breakfast`, `Groceries`
- `Social Action` → `Financial Aid`, `Basic Food Basket`, `Transport`
- `Fixed Expenses` → `Rent + Water`, `Electricity`, `Accountant`

**Why this matters:** The subcategory layer is what enabled the organization to see, for example, that 41% of Social Action spending went to direct financial aid vs. 35% to food baskets. This level of visibility supports board-level decisions about resource allocation.

### 2. Financial reserve as an external input (not derived from transactions)

The organization does not hold its reserve in a dedicated savings account. Initially, the reserve was tracked through a prepaid credit card (funds loaded = reserve), but the card reached its maximum limit of R$ 5,000. From that point forward, the reserve is tracked as the month-end checking account balance.

This means the reserve **cannot be reliably derived from the transaction ledger**. It is entered manually at month-end close and stored as a separate data point.

**Trade-off acknowledged:** Manual input introduces a potential for error and requires discipline at close. The alternative — deriving reserve from transactions — would produce an incorrect figure given the current banking structure. A future improvement would be a dedicated reserve account that makes this fully automatable.

### 3. Operational cost definition (custom, not generic)

Standard financial frameworks define "operating costs" broadly. For this organization, I defined operational cost as the minimum spending required to keep operations running:

```
Operational Cost = Fixed Expenses + Food + Materials
```

Social Action and Events were explicitly excluded because they are discretionary — the organization can temporarily reduce or suspend them without ceasing to operate.

This definition then drives the **Financial Autonomy** indicator:

```
Financial Autonomy (months) = Total Reserve ÷ Monthly Operational Cost
```

As of June 2026: R$ 9,016 ÷ R$ 4,415 = **2.0 months**

This means the organization can sustain core operations for 2 months without any new income — a metric directly usable by leadership for risk assessment.

---

## Key Financial Indicators

| Indicator | Definition | Current Value |
|---|---|---|
| Total Revenue | Sum of all income entries in the period | R$ 43,551 |
| Total Expenses | Sum of all expense entries in the period | R$ 43,494 |
| Period Result | Revenue − Expenses | R$ +56.73 |
| Checking Account Balance | Month-end balance | R$ 3,862 |
| Card Reserve | Prepaid card balance | R$ 5,154 |
| Total Reserve | Checking + Card | R$ 9,016 |
| Operational Cost | Fixed + Food + Materials (monthly avg) | R$ 4,415 |
| Financial Autonomy | Reserve ÷ Operational Cost | 2.0 months |

---

## Dashboard Structure

The dashboard is built in **Looker Studio**, connected to Google Sheets as the data source.

**Page 1 — Executive Summary**
- KPI scorecards: Revenue, Expenses, Period Result, Reserve totals
- Expense breakdown by category (bar chart + donut)
- Monthly financial result trend (line chart)
- Operational metrics: Operational Cost, Financial Autonomy

**Page 2 — Financial Analysis**
- Revenue vs. Expenses trend (dual-line)
- Reserve evolution over time
- Top 10 expenses by subcategory
- Social Action and Food expense distribution

**Page 3 — Social Action Deep Dive**
- Monthly trend per subcategory (Aid, Food Basket, Transport)
- Beneficiary breakdown table with individual totals

**Page 4 — Food Expense Detail**
- Monthly trend: Lunch, Breakfast, Groceries
- Total by subcategory

**Page 5 — Annual Retreat 2026 Project**
- Participant registration and confirmation status
- Payment tracking: paid, outstanding, total expected
- Occupancy rate and budget gap analysis

---

## Tech Stack

| Tool | Role |
|---|---|
| Google Sheets | Data storage, classification logic, KPI calculation |
| Looker Studio | Dashboard and visualization layer |
| GitHub | Version control and portfolio documentation |

---

## Current Limitations & Roadmap

**Known limitations:**
- Bank statement import is still manual (copy-paste from bank export)
- Reserve input requires manual entry at month-end
- No automated data validation on import
- System covers a single fiscal year; historical comparison requires manual setup

**Planned improvements:**
- [ ] SQL version of core queries (for portfolio demonstration)
- [ ] Automated import pipeline using Python or n8n
- [ ] Data validation layer to catch sign errors and duplicates on import
- [ ] Dedicated reserve tracking separated from operational balance

---

## How to Explore This Project

1. Review the dashboard screenshots in `/dashboard/`
2. See the data model diagram in `/docs/data-model.png`
3. Read the KPI definitions in `/docs/kpi-definitions.md`
4. The classification logic (keyword rules) is documented in `/docs/classification-rules.md`

---

## About This Project

This system was built to solve a real operational problem for the treasury of a Brazilian nonprofit organization, starting from zero infrastructure. The process of building it — identifying what data was needed, how to structure it, and what indicators would actually support decisions — is documented throughout this repository.

It is part of a broader portfolio focused on **Data & Finance**, combining financial analysis, data modeling, and business intelligence.

> **Author:** Yuli Rodrigues
> Data & Finance professional focused on turning financial data into actionable decisions.
> Focus: Financial Analysis · Business Intelligence · FP&A · Data Analytics

---

*Last updated: June 2026*

