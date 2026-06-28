# Transaction Classification Rules
### Nonprofit Treasury System – Financial Management Project

This document describes the classification system used to categorize financial transactions in this project: its architecture, lookup logic, coverage, limitations, and maintenance process.

---

## Overview

Every transaction imported from the bank statement is assigned two attributes:

- **Category** — the high-level classification (e.g., Fixed Expenses, Food, Social Action)
- **Subcategory** — the granular classification within the category (e.g., Rent + Water, Lunch, Financial Aid)

Classification is handled through a combination of **automated lookup** and **manual review**. All entries are manually verified after automated classification runs.

---

## Architecture

The classification system is built on a dedicated sheet tab called `REGRAS_CLASSIFICACAO` (Classification Rules), which functions as a lookup table:

| Column | Content |
|---|---|
| A | Exact text as it appears in the bank statement (payee name or transaction description) |
| B | Category |
| C | Subcategory |

This table is the single source of truth for all classification rules. When a new recurring pattern is identified, a new row is added to this table.

---

## Classification Formula

Each transaction row in the `MOVIMENTACAO` (Transactions) tab uses the following formula to assign a category:

```
=IFERROR(VLOOKUP(H183, REGRAS_CLASSIFICACAO!$A:$C, 2, FALSE),
  IFERROR(VLOOKUP(G183, REGRAS_CLASSIFICACAO!$A:$C, 2, FALSE), ""))
```

**How it works:**

1. First, it attempts to match the value in column H (primary description field from the bank export) against column A of the classification rules table
2. If no match is found, it attempts to match the value in column G (secondary description field)
3. If neither field produces a match, the formula returns an empty string — flagging the transaction for manual classification

The same logic applies to subcategory, using column index 3 instead of 2.

**Why two lookup fields?** Bank statements sometimes export the same transaction with different description formats depending on the transaction type (e.g., a PIX transfer may populate different fields than a card payment). Using two lookup fields increases automatic coverage without requiring duplicate entries in the rules table.

---

## Classification Rules — Current Table

The table below reflects the classification rules currently in the system. Entries are exact matches against bank statement text.

### RECEITAS (Revenue)

| Bank Statement Text | Category | Subcategory |
|---|---|---|
|  CONTRIBUINTE 1 | RECEITAS | CONTRIBUIÇÃO ALUGUEL |
|  CONTRIBUINTE 2 | RECEITAS | OUTRAS RECEITAS |
|  CONTRIBUINTE 3 | RECEITAS | OFERTA |

### DESPESAS FIXAS (Fixed Expenses)

| Bank Statement Text | Category | Subcategory |
|---|---|---|
| FORNECEDOR 1 | DESPESAS FIXAS | ALUGUEL + ÁGUA |
| FORNECEDOR 2 | DESPESAS FIXAS | LUZ |
| FORNECEDOR 3 | DESPESAS FIXAS | LUZ |
| FORNECEDOR 4 | DESPESAS FIXAS | CONTADOR |

### AÇÃO SOCIAL (Social Action)

| Bank Statement Text | Category | Subcategory |
|---|---|---|
| BENEFICIARIO 1 | AÇÃO SOCIAL | TRANSPORTE |
| BENEFICIARIO 2 | AÇÃO SOCIAL | TRANSPORTE |
| BENEFICIARIO 3 | AÇÃO SOCIAL | CESTA BÁSICA |

### ALIMENTAÇÃO (Food)

| Bank Statement Text | Category | Subcategory |
|---|---|---|
| FORNECEDOR 1 | ALIMENTAÇÃO | CAFÉ DA MANHÃ |
| FORNECEDOR 2 | ALIMENTAÇÃO | CAFÉ DA MANHÃ |
| FORNECEDOR 3 | ALIMENTAÇÃO | CAFÉ DA MANHÃ |
| FORNECEDOR 4 | ALIMENTAÇÃO | CAFÉ DA MANHÃ |
| FORNECEDOR 5 | ALIMENTAÇÃO | ALMOÇO |
| FORNECEDOR 6 | ALIMENTAÇÃO | ALMOÇO |
| FORNECEDOR 7 | ALIMENTAÇÃO | ALMOÇO |
| FORNECEDOR 8 | ALIMENTAÇÃO | ALMOÇO |
| FORNECEDOR 9 | ALIMENTAÇÃO | CESTA BÁSICA |

### MATERIAIS (Materials)

| Bank Statement Text | Category | Subcategory |
|---|---|---|
| FORNECEDOR A | MATERIAIS | COMPRAS DE MERCADO |
| FORNECEDOR B | MATERIAIS | COMPRAS DE MERCADO |
| FORNECEDOR C | MATERIAIS | LIMPEZA E DESCARTÁVEIS |
| FORNECEDOR D | MATERIAIS | MINISTÉRIO INFANTIL |

### INFRAESTRUTURA (Infrastructure)

| Bank Statement Text | Category | Subcategory |
|---|---|---|
| EMPRESA A | INFRAESTRUTURA | MOBILIÁRIO |
| EMPRESA   | INFRAESTRUTURA | MANUTENÇÃO |

### EVENTOS (Events)

| Bank Statement Text | Category | Subcategory |
|---|---|---|
| FORNECEDOR E | EVENTOS | ANIVERSARIANTES |

---

## Coverage and Limitations

**What the system classifies automatically:**
Transactions from recurring suppliers and service providers with consistent bank statement descriptions. These represent the majority of fixed and operational expenses.

**What requires manual classification:**
- PIX transfers from individuals — the payee name changes with each new person, making pattern-based lookup impractical
- One-off transactions with no prior history
- Transactions where the bank statement description is ambiguous or truncated

**Design decision:** The system deliberately does not attempt fuzzy matching or partial-text lookup. Exact matching was chosen because the cost of a false positive (a transaction classified incorrectly without the user noticing) is higher than the cost of a manual review. Every unmatched transaction surfaces as an empty field, which triggers a manual check.

**Manual review process:** After automated classification runs, all transactions are reviewed manually. When an unclassified transaction belongs to a recurring supplier, a new rule is added to the `REGRAS_CLASSIFICACAO` table so future transactions from that supplier are classified automatically.

---

## Maintenance

The classification table is a living document. It grows as new suppliers and transaction patterns are identified.

**When to add a new rule:**
- A supplier appears more than once with the same bank statement description
- A transaction type is recurring and predictable

**When NOT to add a rule:**
- One-off transactions (adding them creates noise in the rules table without improving future coverage)
- Transactions where the description varies between occurrences

---

## Planned Improvements

- [ ] Add a `REVISAR` (Review) flag column to explicitly mark transactions that need manual attention, rather than relying on empty fields as the signal
- [ ] Build a summary view showing classification coverage rate — percentage of transactions classified automatically vs. manually each month
- [ ] Migrate classification logic to SQL for scalability and auditability

---

*Last updated: June 2026*
