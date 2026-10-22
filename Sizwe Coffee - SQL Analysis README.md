# Nurah-Galiem-SQL-Assignment
## Swize Coffee - SQL Analysis

## Project Overview

This project answers eight business questions from the Sizwe Coffee year-end review using SQL (PostgreSQL dialect, executed in DuckDB).

Business context. Sizwe Coffee (Pty) Ltd is a coffee roastery and wholesale supplier based in Isando, Johannesburg, with a second distribution centre in Epping, Cape Town. It sells to about 1,400 business customers through two channels, Wholesale (about 70% of revenue) and Subscription (about 30%). The financial year runs 1 July to 30 June; the data covers FY2024 (1 Jul 2023 - 30 Jun 2024) and FY2025 (1 Jul 2024 - 30 Jun 2025). Revenue was R441.0m in FY2024 and R480.0m in FY2025.

The eight questions (asked by Nomsa Dlamini, Financial Director, and Bongani Twala, National Sales Manager):

1. Channel performance - FY2024 vs FY2025 revenue and growth by channel

2. Profit by product line - revenue, COGS, gross profit and margin

3. The collections problem - settlement speed and outstanding value by channel

4. The ten largest customers by FY2025 revenue

5. Top Wholesale customers who have never received a trade promotion

6. Sales representatives against monthly target

7. Monthly revenue with month-on-month change and a rolling 3-month average

8. Why gross margin fell from 34.7% to 33.0%

| Table | Rows | Used in |
|---|---|---|
| customers | 1,400 | 3, 4, 5 |
| products | 23 | 2, 8 |
| sales_orders | 48,144 | 1, 4, 5, 6, 7, 8 |
| sales_lines | 168,172 | 1, 2, 4, 5, 6, 7, 8 |
| ar_ledger | 48,144 | 3 |
| reps | 18 | 4, 6 |
| trade_promotions | 640 | 5 |
| rep_visits, vendors | - | not needed |

Relationships: `sales_orders.customer_id -> customers.customer_id`; `sales_orders.order_id -> sales_lines.order_id`; `sales_lines.sku -> products.sku`; `customers.rep_id` / `sales_orders.rep_id -> reps.rep_id`; `ar_ledger.customer_id -> customers.customer_id`; `trade_promotions.customer_id -> customers.customer_id`.

## Skills demonstrated

| Technique | Where |
|---|---|
| Inner joins across 3-4 tables | Tasks 1-5, 8 |
| Conditional aggregation (`SUM(CASE WHEN ...)`) to pivot FY columns | Task 1 |
| `GROUP BY` aggregation, margin calculations | Tasks 2, 3 |
| `LEFT JOIN` + `COALESCE` to keep rows with no match | Tasks 4, 6 |
| Filter in the `ON` clause to preserve outer-join rows | Task 6 |
| `NOT EXISTS` anti-join (NULL-safe) | Task 5 |
| CTEs | Tasks 1, 7, 8 |
| Window functions: `LAG`, rolling `AVG ... ROWS BETWEEN`, `ROW_NUMBER`, `SUM() OVER (PARTITION BY)` | Tasks 7, 8 |
| Date functions (`DATE_TRUNC`, `DATEDIFF`) | Tasks 3, 7 |
| `ORDER BY` / `LIMIT` ranking | Tasks 2, 4, 5, 6 |

## How to run

1. In Databricks, go to **Catalog > Create table > Upload file** and upload each `sizwe_*.csv` (customers, products, sales_orders, sales_lines, ar_ledger, reps, trade_promotions). Keep the table names identical to the file names without the `sizwe_` prefix, e.g. `sizwe_sales_lines.csv` -> `sales_lines`.

2. In the upload screen tick **First row contains header** and make sure column types are inferred (dates as DATE, amounts as DOUBLE/DECIMAL), not left as STRING.

3. Open the SQL Editor, select your catalog/schema (`USE schema_name;`) and run each query in `queries.sql` in order. Each query is preceded by its task number, with written answers in comments beneath it.
4. 
Databricks-specific note: Task 3 uses `DATEDIFF(end, start)` for days to settle, because subtracting two dates returns an interval in Databricks SQL.

## Key insights

**For Nomsa (Finance)**

- **Growth is even across channels.** Wholesale grew 8.85% (R308.2m to R335.4m) and Subscription 8.84% (R132.8m to R144.6m); total revenue rose R441.0m to R480.0m.
- **Collections are the real problem and it is all Wholesale.** Subscription pays in 3.8 days. Wholesale averages 68.7 days against 60-day terms and holds R87.8m of the R88.6m outstanding (3,454 open invoices). Average days to settle worsened from 56.9 (FY2024) to 84.1 (FY2025), and every open invoice is from FY2025. Independent Cafes carry the largest open balance (about R36.7m). The largest account, Halvane Retail Group, has about R10.7m outstanding against a R6m credit limit.
- **The margin decline is a mix effect, not a pricing or cost problem.** Each line's margin is flat year on year (Bulk Commercial 19.89% to 19.91%, House Blend 38.32% to 38.34%, Single Origin 44.24% to 44.31%). Bulk Commercial, the lowest-margin line, grew 46% (R110.3m to R161.5m) and its revenue share rose from 25.0% to 33.7%, while the other four lines lost R12.2m combined. Applying FY2024 line margins to the FY2025 mix reproduces 33.0%.
- **Profit is concentrated differently from revenue.** House Blend earns the most gross profit (R107.1m); Single Origin has the best margin (44.3%); Bulk Commercial is the second-largest line by revenue (R271.8m) but only 19.9% margin.
- **Seasonality is strong.** Revenue declines from July to December and recovers from January to a June peak; December is the quietest month (R29.9m in 2023, R32.6m in 2024, roughly -13% on November).

**For Bongani (Sales)**
- **Customer concentration.** Halvane Retail Group generated R38.4m in FY2025 (about 8% of revenue), more than 11x the next account (Highveld Deli 85 at R3.4m).
- **Promotion gap.** The top three Wholesale customers with no promotion are Umhlanga Fresh 108 (R6.2m), Ridgeway Foods 308 (R4.4m) and Kalahari Kitchen 42 (R4.4m); I recommend a structured promotion and a rep review for them.
- **Rep performance.** 16 of 18 reps beat monthly target in FY2025 (Karabo van Wyk 250%, Kagiso Mkhize 250%); Sipho Steyn (98.0%) and Johan Naidoo (77.6%) fall short. Combined targets of R25.4m/month against roughly R40m actual suggest targets need resetting.
