# Data Model
### Nonprofit Treasury System – Financial Management Project

This document describes the data architecture of the system: the spreadsheet structure, how data flows between tabs, the role of each sheet, and the relationships between them.

---

## Overview

The system is organized across two separate workbooks:

- **Base Mestre** — core treasury system covering all organizational transactions, classification, reserves, and financial health indicators
- **Retiro 2026** — standalone event management system for the Annual Retreat, covering participant registration, payment tracking, and budget control

---

## Workbook 1 — Base Mestre

### Sheet Structure

```
BASE MESTRE
│
├── RAW_BANK
│     Raw bank statement data (copy-paste from bank export)
│
├── MOVIMENTACOES
│     Classified transaction ledger
│     ← reads from RAW_BANK (manual copy)
│     ← reads from REGRAS_CLASSIFICACAO (VLOOKUP)
│     ← reads from PLANO_CONTAS (validation)
│
├── PLANO_CONTAS
│     Master list of valid categories and subcategories
│     → used as validation source in MOVIMENTACOES
│
├── REGRAS_CLASSIFICACAO
│     Keyword-to-category mapping table
│     → used as lookup source in MOVIMENTACOES
│
├── RESERVA_HISTORICA
│     Month-end reserve snapshot (manual input)
│     → feeds SAUDE_FINANCEIRA and Looker Studio
│
└── SAUDE_FINANCEIRA
      Consolidated KPI calculation layer
      ← reads from MOVIMENTACOES (aggregations)
      ← reads from RESERVA_HISTORICA (reserve values)
      → feeds Looker Studio dashboard
```

---

### Sheet Descriptions

#### RAW_BANK
**Role:** Landing zone for raw bank statement data.

**Columns:** Date, Transaction type, Transaction description, Payee/Description, Debit value, Credit value.

**How data enters:** Manual copy-paste from the bank's exported statement. No transformations applied at this stage — data is kept exactly as exported.

**Design decision:** Keeping raw data untouched in a separate tab creates a clear audit trail. If a classification error is found, the original source data is always available for reference.

---

#### MOVIMENTACOES
**Role:** Core transaction ledger. Every financial movement of the organization lives here, fully classified.

**Columns:** Row ID, Date, Year, Month, Year-Month, Day of week, Bank transaction description, Payee, Value, Category, Subcategory, Project tag, Notes.

**How data enters:** Rows are populated from RAW_BANK. Classification fields (Category, Subcategory) are populated automatically via VLOOKUP against REGRAS_CLASSIFICACAO, with manual review and override for unmatched transactions.

**Key formula:**
```
Category =
  IFERROR(
    VLOOKUP(payee_field, REGRAS_CLASSIFICACAO!$A:$C, 2, FALSE),
    IFERROR(
      VLOOKUP(description_field, REGRAS_CLASSIFICACAO!$A:$C, 2, FALSE),
    "")
  )
```

**Project tag:** Optional field used to link transactions to specific initiatives (e.g., Annual Retreat 2026). Enables project-level financial tracking within the main ledger.

---

#### PLANO_CONTAS
**Role:** Master reference table defining all valid categories and subcategories in the system.

**Columns:** Category, Subcategory.

**Current structure:**

| Category | Subcategories |
|---|---|
| Receitas | Oferta, Contribuição Aluguel, Outras Receitas |
| Despesas Fixas | Aluguel + Água, Luz, Contador |
| Alimentação | Café da Manhã, Almoço, Lanche |
| Ação Social | Cesta Básica, Transporte, Auxílio Financeiro |
| Materiais | Limpeza e Copa, Ministério Infantil |
| Infraestrutura | Manutenção, Equipamento e Mobiliário |
| Eventos | Retiro, Casais, Mulheres, Homens, Jovens, Aniversariantes |
| Reserva Financeira | Aporte Reserva, Resgate Reserva |

**Role in the system:** Acts as the single source of truth for valid classification values. Prevents inconsistent category names from entering the ledger (e.g., "Alimentação" vs "ALIMENTAÇÃO" vs "alimentacao").

---

#### REGRAS_CLASSIFICACAO
**Role:** Lookup table that maps bank statement text to category and subcategory. Powers the automatic classification of recurring transactions.

**Columns:** Bank statement text (exact match), Category, Subcategory.

**How it works:** See `classification-rules.md` for full documentation.

---

#### RESERVA_HISTORICA
**Role:** Historical record of the organization's financial reserve at month-end close.

**Columns:** Year-Month, Card Balance, Checking Account Balance, Total Reserve.

**How data enters:** Manual input at the end of each month. Values are not derived from the transaction ledger (see KPI definitions for rationale).

**Current data:**

| Year-Month | Card Balance | Checking Account | Total Reserve |
|---|---|---|---|
| 2026-01 | R$ 5,153.66 | R$ 3,057.18 | R$ 8,210.84 |
| 2026-02 | R$ 5,153.66 | R$ 3,102.58 | R$ 8,256.24 |
| 2026-03 | R$ 5,153.66 | R$ 5,193.35 | R$ 10,347.01 |
| 2026-04 | R$ 5,153.66 | R$ 7,916.15 | R$ 13,069.81 |
| 2026-05 | R$ 5,153.66 | R$ 5,360.11 | R$ 10,513.77 |
| 2026-06 | R$ 5,153.66 | R$ 3,862.29 | R$ 9,015.95 |

**Observation:** The card balance has remained constant at R$ 5,153.66 across all months because the prepaid card reached its maximum load limit. Reserve fluctuations are driven entirely by the checking account balance.

---

#### SAUDE_FINANCEIRA
**Role:** KPI calculation layer. Aggregates data from MOVIMENTACOES and RESERVA_HISTORICA to produce the financial health indicators displayed in the dashboard.

**Structure:** Two sections in the same sheet.

**Section 1 — Current indicators (point-in-time):**

| Indicator | Value (June 2026) |
|---|---|
| Card Balance | R$ 5,153.66 |
| Checking Account Balance | R$ 3,862.29 |
| Current Reserve | R$ 9,015.95 |
| 6-Month Average Operational Cost | R$ 4,414.63 |
| Financial Autonomy | 2.0 months |
| Social Action % of Budget | 32.7% |

**Section 2 — Monthly operational cost history:**
Rolling calculation of operational cost (Fixed Expenses + Food + Materials) for each month, used to compute the 6-month average.

---

### Data Flow Diagram

```
Bank Export (PDF / online banking)
        │
        │ manual copy-paste
        ▼
┌─────────────────┐
│   RAW_BANK      │  ← source of truth for raw transactions
└────────┬────────┘
         │ manual transfer + VLOOKUP
         ▼
┌─────────────────────────────────────────────┐
│   MOVIMENTACOES                             │
│                                             │
│  Date | Payee | Value | Category | Subcat  │
│                                             │
│  Category ← VLOOKUP ← REGRAS_CLASSIFICACAO │
│  Subcat   ← VLOOKUP ← REGRAS_CLASSIFICACAO │
│  Validation ← PLANO_CONTAS                 │
└──────────────────┬──────────────────────────┘
                   │ aggregations (SUMIF, AVERAGEIF)
                   ▼
┌─────────────────────────────────────────────┐
│   SAUDE_FINANCEIRA                          │  ← KPI layer
│                                             │
│  also reads from RESERVA_HISTORICA ────────►│
└──────────────────┬──────────────────────────┘
                   │ connected as data source
                   ▼
         ┌─────────────────┐
         │  LOOKER STUDIO  │  ← dashboard layer
         │    Dashboard    │
         └─────────────────┘
```

---

## Workbook 2 — Retiro 2026

### Sheet Structure

```
RETIRO 2026
│
├── PARTICIPANTES
│     Participant registry with confirmation and payment tracking
│
├── DESPESAS
│     Event budget: estimated vs. realized costs
│
└── DASHBOARD
      Event KPI summary — feeds Looker Studio
```

---

### Sheet Descriptions

#### PARTICIPANTES
**Role:** Tracks every invited participant — confirmation status, adult/child classification, payment progress.

**Columns:** ID, Name, Confirmed, Adult/Child, Estimated Value, Amount Paid, Pending Amount, Status.

**Status values:** Não confirmado (Not confirmed), Em pagamento (Payment in progress), Quitado (Paid in full).

**Key calculations:**
- Pending Amount = Estimated Value − Amount Paid
- Occupancy Rate = Confirmed participants ÷ Total expected participants

---

#### DESPESAS
**Role:** Event budget control — tracks planned costs against actual spending.

**Columns:** Date, Category, Description, Estimated Value, Realized Value.

**Categories:** Hospedagem (Accommodation), Transporte (Transport), Alimentação (Food).

---

#### DASHBOARD
**Role:** KPI summary layer for the event, feeding the Looker Studio dashboard page.

**Current indicators:**

| Indicator | Value |
|---|---|
| Target participants | 38 |
| Confirmed | 4 |
| Confirmation rate | 11% |
| Estimated total revenue | R$ 10,200 |
| Collected to date | R$ 700 |
| Pending collection | R$ 9,500 |
| Estimated total cost | R$ 8,300 |
| Realized cost | R$ 300 |
| Projected deficit | -R$ 1,900 |
| Organization subsidy needed | R$ 0 |

---

## Design Decisions

**Two separate workbooks instead of one:** The Annual Retreat is a time-limited project with its own logic, participants, and budget. Keeping it in a separate workbook avoids polluting the main transaction ledger with event-specific data while still allowing the event to appear in the main dashboard via a project tag in MOVIMENTACOES.

**PLANO_CONTAS as a control layer:** Maintaining a master list of valid categories prevents data quality issues that would silently corrupt aggregations — a misspelled category name would cause transactions to disappear from reports without any error message.

**SAUDE_FINANCEIRA as a dedicated KPI layer:** Rather than calculating KPIs directly in the dashboard tool, all financial health indicators are computed in a dedicated sheet. This makes the logic transparent, auditable, and easy to modify without touching the dashboard configuration.

---

*Last updated: June 2026*
