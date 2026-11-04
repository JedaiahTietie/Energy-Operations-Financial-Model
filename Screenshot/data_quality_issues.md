# Known Data-Quality Issues in `data/raw/`

These issues are intentionally injected by `src/generate_synthetic_data.py` with seed `42`.
Use `data/expected_clean/` only to validate your work, not as a substitute for documenting
your own Power Query cleaning steps.

| File | Issue | Count | Recommended treatment |
|---|---|---:|---|
| `fact_ledger_raw.csv` | Exact duplicate rows | 120 | Remove duplicates on `transaction_id`; keep the first record. |
| `fact_ledger_raw.csv` | Case / trailing-space inconsistencies in `service_line` or `corridor` | 180 | Trim, clean, and map to the dimension tables. |
| `fact_ledger_raw.csv` | `amount_ngn` = `N/A` | 18 | Replace with null, then recover from `amount_local × fx_rate_to_ngn`; document any residual exclusions. |
| `fact_ledger_raw.csv` | Missing `fx_rate_to_ngn` | 25 | Merge with `macro_factors_clean.csv` on `month` and fill. |
| `fact_ledger_raw.csv` | Missing `customer_id` | 90 | Keep the row; label customer as `Unallocated` for revenue analysis. |
| `fact_ledger_raw.csv` | `currency` = `usd` | 40 | Uppercase and validate against `NGN` / `USD`. |
| `fact_ledger_raw.csv` | Invalid or mixed-format dates | 12 | Parse with `try ... otherwise null`; exclude or correct with a documented rule. |
| `fact_budget_raw.csv` | Case / trailing-space inconsistencies | 20 | Trim and map to `dim_service_line`. |
| `fact_budget_raw.csv` | `budget_amount_ngn` = `N/A` | 8 | Replace with null; recover from the neighbouring plan rows or exclude with a note. |
| `fact_budget_raw.csv` | Exact duplicate rows | 5 | Remove duplicates on the full row. |
| `fact_collections_raw.csv` | Exact duplicate invoice rows | 40 | Remove duplicates on `invoice_id`; keep the first record. |
| `fact_collections_raw.csv` | `payment_date` before `invoice_date` | 15 | Set to null and flag as `Invalid payment date`. |
| `fact_collections_raw.csv` | Missing `paid_amount_ngn` | 20 | Replace with `0` where status is `Open` / `Overdue`; otherwise investigate. |
| `fact_collections_raw.csv` | `invoice_amount_ngn` = `N/A` | 10 | Recover from `fact_ledger_clean.csv` by `invoice_id`. |
| `macro_factors_raw.csv` | Missing diesel price | 2 | Interpolate linearly between adjacent months and flag the row. |
| `macro_factors_raw.csv` | `inflation_index` = `N/A` | 1 | Interpolate and flag the row. |
| `macro_factors_raw.csv` | Inconsistent month format | 1 | Normalise to `YYYY-MM`. |

## Legitimate exceptions — do not delete

- `record_type = 'Credit Note'` rows have a **positive** signed amount on a cost account. They are
  supplier reversals, not errors. Keep them and explain the sign convention.
- Negative `amount_ngn` values are normal cost records. `Revenue` is positive; `Cost` is negative.
- Blank `customer_id` is expected on cost records.

## Expected clean row counts

- `fact_ledger_clean.csv`: 14,400 rows
- `fact_budget_clean.csv`: 3,840 rows
- `fact_collections_clean.csv`: 4,320 rows
- `macro_factors_clean.csv`: 48 rows
- `dim_date.csv`: 1,461 daily rows
