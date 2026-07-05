# Data Quality Assessment
## Online Retail II Dataset — Sheet "Year 2010-2011"

**Project:** retail-customer-insights
**Phase:** 1 — Excel Exploration
**Date:** July 2026
**Author:** Heba

---

## 1. Dataset Structure

- Total columns: **8**
- Column names: `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `Price`, `CustomerID`, `Country`

---

## 2. Missing Values

| Location | Count | Notes |
|---|---|---|
| Total blank cells (whole sheet) | 136,534 | Found via blank Find & Replace search |
| `CustomerID` blanks | 135,080 | ~25% of all records — will be excluded from RFM analysis, since RFM requires a valid customer identifier |
| `Description` blanks | 1,454 | Remaining blank cells after subtracting `CustomerID` blanks (136,534 − 135,080 = 1,454), confirmed via filter |

**Decision:** Rows with missing `CustomerID` must be excluded before RFM segmentation. Rows with missing `Description` need individual review (see Section 4).

---

## 3. Negative Quantity Values

- Sorting `Quantity` ascending reveals negative values.
- **Root cause identified:** All negative `Quantity` values occur exclusively on invoices where `InvoiceNo` starts with the letter `C` (e.g. `C536379`).
- **Business meaning:** `C` prefix = **Cancellation / Return**. Negative quantity + positive price = a product being returned by a customer (damage or dissatisfaction), representing a loss for the business.
- **Decision:** These rows must be excluded (or handled separately) before calculating `Monetary` value in RFM — otherwise returns would be miscounted as additional purchases.

---

## 4. Non-Product Stock Codes (`StockCode`)

Identified via unique values in the `StockCode` filter (not by guessing keywords in `Description`, which is unreliable):

| StockCode | Description | Row Count | Quantity sign pattern |
|---|---|---|---|
| `POST` | POSTAGE | 1,257 | Mixed — negative only on `C`-prefixed invoices (returns), positive on regular invoices (incl. large wholesale values up to 1,000) |
| `D` | Discount | 77 | Mixed — negative only on `C`-prefixed invoices |
| `M` | Manual | 572 | Mixed — negative only on `C`-prefixed invoices |
| `DOT` | DOTCOM POSTAGE | 710 | **No negative values at all** — breaks the pattern seen in other codes (see note below) |
| `C2` | Carriage | 2 | Mixed — negative only on `C`-prefixed invoices |
| `BANK CHARGES` | Bank Charges | 37 | Mixed — negative only on `C`-prefixed invoices |
| `CRUK` | CRUK Commission | 16 | Not fully verified yet |
| `S` | Samples | 63 | Not fully verified yet |
| `AMAZONFEE` | Amazon Fee | 34 | Not fully verified yet |

**Open question (documented for later investigation):** Why does `DOT` (Dotcom Postage) never appear with negative quantity, unlike `POST`? Hypothesis: dotcom/online channel sales may follow a different return-handling process than regular sales. To be investigated further, not yet confirmed.

**Decision:** These are administrative/fee codes, not real products. They must be excluded from product-level analysis (e.g. best-selling items) but may need separate handling in revenue reconciliation.

---

## 5. `Description` vs `StockCode` Discrepancy (Case Study in Verification)

- Filtering `Description = "POSTAGE"` returned **1,253** rows.
- Filtering `StockCode = "POST"` returned **1,257** rows.
- Initial hypothesis: text mismatch (extra space, typo) in `Description`.
- **Verified root cause:** The 4 extra rows have a **blank `Description`**, `Price = 0`, and unusually large `Quantity` values (750, 800, 800, 1,000), all under `Country = United Kingdom`. These appear to be internal adjustments / stock corrections, not real sales.
- **Decision:** These 4 rows have zero monetary impact (`Price = 0`) and can be safely excluded from RFM `Monetary` calculations, but should be flagged as a documented data quality note.
- **Methodology note:** Verified using `EXACT()` formula cross-referenced with the `StockCode` filter — a lesson in not trusting assumptions until validated with a formula-based check.

---

## 6. Key Takeaways for Cleaning Phase

1. Exclude rows with missing `CustomerID` (135,080 rows) before RFM analysis.
2. Exclude or separately flag all `C`-prefixed invoices (returns/cancellations) before calculating `Monetary`.
3. Exclude non-product `StockCode` values (`POST`, `D`, `M`, `DOT`, `C2`, `BANK CHARGES`, `CRUK`, `S`, `AMAZONFEE`) from product-level analysis.
4. Exclude the 4 rows with blank `Description` and `Price = 0` identified as internal adjustments.
5. Document the `DOT` anomaly (no negative values) as an open question for further investigation in the Python phase.

---

*This document was produced through manual Excel exploration as part of Phase 1 of the retail-customer-insights project, prior to any data cleaning or transformation.*
