# KPI Definitions
### Nonprofit Treasury System – Financial Management Project

This document defines all financial indicators used in this project: their formulas, calculation scope, data sources, and the business rationale behind each design decision.

---

## 1. Total Revenue

**Definition:** Sum of all income entries recorded in the transaction ledger for the selected period.

**Formula:**
```
Total Revenue = SUM(all transactions where type = "income")
```

**Scope:** Period-sensitive. With no filter applied, covers the full dataset (January–June 2026). With a monthly filter, covers only the selected month.

**Data source:** Transaction ledger (Raw Data tab — Google Sheets).

**Business rationale:** Tracks all inflows to the organization. Used as the baseline for period result and budget health assessment.

---

## 2. Total Expenses

**Definition:** Sum of all expense entries recorded in the transaction ledger for the selected period.

**Formula:**
```
Total Expenses = SUM(all transactions where type = "expense")
```

**Scope:** Period-sensitive. Mirrors the same filter logic as Total Revenue.

**Data source:** Transaction ledger (Raw Data tab — Google Sheets).

**Business rationale:** Full view of organizational spending. Feeds into period result, expense distribution, and category-level analysis.

---

## 3. Period Result

**Definition:** Net financial result for the selected period — the difference between total revenue and total expenses.

**Formula:**
```
Period Result = Total Revenue − Total Expenses
```

**Scope:** Period-sensitive. No adjustments applied — reserve movements are tracked separately and do not affect this calculation.

**Data source:** Derived from Total Revenue and Total Expenses.

**Business rationale:** Indicates whether the organization ended the period with a surplus or deficit. A positive result does not necessarily mean the organization is financially healthy — reserve level and operational autonomy must be assessed alongside this indicator.

**Design decision:** Reserve movements were deliberately excluded from this calculation. Including them would conflate operational performance with treasury management, making it harder to assess whether the organization's core activities are self-sustaining.

---

## 4. Checking Account Balance

**Definition:** Month-end balance of the organization's checking account, recorded manually at the close of each month.

**Formula:**
```
Checking Account Balance = manually entered month-end value
```

**Scope:** Point-in-time. Reflects the balance at the last day of the reference month.

**Data source:** Manual input — not derived from the transaction ledger.

**Business rationale:** Represents the liquid cash available in the organization's primary operating account. Together with the card reserve, it forms the total reserve.

**Known limitation:** Because this value is entered manually, it is not automatically reconciled against the transaction ledger. A future improvement would be to derive this value from the running transaction balance, which would enable automatic reconciliation and error detection.

---

## 5. Card Reserve

**Definition:** Available balance on the organization's prepaid credit card, recorded manually at the close of each month.

**Formula:**
```
Card Reserve = manually entered month-end card balance
```

**Scope:** Point-in-time.

**Data source:** Manual input.

**Business rationale:** The organization uses a prepaid credit card as a secondary reserve mechanism. Funds are loaded onto the card and held there until needed. This balance is tracked separately from the checking account because it represents a distinct liquidity layer — less immediately accessible than the checking account but earmarked as reserve.

**Context:** This structure emerged from an operational constraint. The card has a maximum load limit of R$ 5,000. Once that limit was reached, additional reserves began accumulating in the checking account instead. Both balances are now tracked and summed as the total reserve.

---

## 6. Total Reserve

**Definition:** Combined financial reserve available to the organization at month-end.

**Formula:**
```
Total Reserve = Checking Account Balance + Card Reserve
```

**Scope:** Point-in-time. Reflects the reserve position at the last day of the reference month.

**Data source:** Derived from Checking Account Balance and Card Reserve (both manually entered).

**Business rationale:** Represents the organization's total financial buffer — the funds available to absorb unexpected expenses or sustain operations during periods of reduced income. Used as the numerator in the Financial Autonomy calculation.

**Design decision:** The two components are tracked separately before being summed because they represent different liquidity layers. Keeping them distinct in the data model allows leadership to understand not just the total reserve, but its composition — which matters if, for example, the card has restrictions on how funds can be used.

---

## 7. Operational Cost

**Definition:** Average monthly cost of keeping the organization's core operations running, calculated over the last 6 months.

**Formula:**
```
Operational Cost = AVG over last 6 months of (Fixed Expenses + Food + Materials)
```

**Scope:** Rolling 6-month average. Recalculates as new months are added.

**Data source:** Transaction ledger, filtered to categories: Fixed Expenses, Food, Materials.

**Categories included:**

| Category | Rationale |
|---|---|
| Fixed Expenses | Non-negotiable recurring costs (rent, utilities, accountant) |
| Food | Regular operational cost — feeding the team and community |
| Materials | Supplies required for regular operations |

**Categories explicitly excluded:**

| Category | Rationale |
|---|---|
| Social Action | Discretionary — can be reduced without halting operations |
| Events | Discretionary — not required for core functioning |

**Business rationale:** A rolling 6-month average was chosen over a single-month or year-to-date average because it is more representative of current spending patterns. A single month can be distorted by one-off expenses; a full-year average may reflect conditions that no longer apply.

**Design decision:** The definition of "operational" was deliberately narrow. The goal was to isolate the minimum cost required to keep the organization functioning — the floor below which it cannot operate. This produces a more conservative and useful denominator for the Financial Autonomy indicator.

---

## 8. Financial Autonomy

**Definition:** Number of months the organization can sustain core operations using its current reserve, without any new income.

**Formula:**
```
Financial Autonomy (months) = Total Reserve ÷ Operational Cost
```

**Display:** Rounded to 1 decimal place.

**Current value (June 2026):** R$ 14,208 ÷ R$ 4,415 = **3.2 months**

**Scope:** Point-in-time numerator (Total Reserve at month-end) divided by rolling average denominator (Operational Cost over last 6 months).

**Data source:** Derived from Total Reserve and Operational Cost.

**Business rationale:** This is the primary risk indicator in the system. It answers the question leadership needs to ask: *"If all income stopped today, how long could we keep operating?"*

A value of 3.2 months means the organization has crossed the lower threshold of financial resilience — above the critical 3-month floor that nonprofit financial frameworks typically recommend as a minimum. The improvement from 2.0 months (January 2026) to 3.2 months reflects a period of revenue growth outpacing operational costs.

**Design decision:** Financial Autonomy was built from a custom definition of operational cost rather than total expenses. Using total expenses as the denominator would include discretionary spending (Social Action, Events), which overstates the true survival threshold. The narrower denominator produces a more accurate picture of resilience.

---

## Indicator Summary

| Indicator | Type | Scope | Source |
|---|---|---|---|
| Total Revenue | Flow | Period-sensitive | Transaction ledger |
| Total Expenses | Flow | Period-sensitive | Transaction ledger |
| Period Result | Flow | Period-sensitive | Derived |
| Checking Account Balance | Stock | Point-in-time | Manual input |
| Card Reserve | Stock | Point-in-time | Manual input |
| Total Reserve | Stock | Point-in-time | Derived |
| Operational Cost | Average | Rolling 6 months | Transaction ledger |
| Financial Autonomy | Ratio | Point-in-time / Rolling | Derived |

---

*Last updated: June 2026*
