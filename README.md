# Assessment_Q2_sql

## Introduction
Transaction Frequency Analysis
The finance team wants to analyze how often customers transact to segment them (e.g., frequent vs. occasional users).

---

## Objective

To segment customers based on how frequently they transact, the finance team requested a report categorizing each user into:

- High Frequency (≥10 transactions per month)

- Medium Frequency (3–9 transactions per month)

- Low Frequency (≤2 transactions per month)

The final output includes:

- Frequency category

- Number of customers in each category

- Their average monthly transaction count

---

## My Approach
I structured the SQL solution in 4 steps using Common Table Expressions (WITH clauses):

1. Monthly Transaction Count per Customer
I first counted how many transactions each customer made in each month:
```
SELECT
    owner_id,
    YEAR(transaction_date) AS txn_year,
    MONTH(transaction_date) AS txn_month,
    COUNT(*) AS txn_count
FROM savings_savingsaccount
GROUP BY owner_id, YEAR(transaction_date), MONTH(transaction_date)
```
2. Average Monthly Transactions
I calculated the average of these monthly counts to understand customer transaction frequency over time.
```
SELECT
    owner_id,
    AVG(txn_count) AS avg_txn_per_month
FROM monthly_transaction_counts
GROUP BY owner_id
```
3. Categorization into Frequency Tiers
Customers were labeled based on their average:

- High Frequency if avg_txn_per_month >= 10

- Medium Frequency if between 3 and 9

- Low Frequency if <= 2
```
CASE
    WHEN AVG(txn_count) >= 10 THEN 'High Frequency'
    WHEN AVG(txn_count) BETWEEN 3 AND 9 THEN 'Medium Frequency'
    ELSE 'Low Frequency'
END AS frequency_category
```

4. Output
The final result showed how many customers fell into each category and their average behavior.
```
SELECT
    frequency_category,
    COUNT(*) AS customer_count,
    ROUND(AVG(avg_transactions_per_month), 1) AS avg_transactions_per_month
FROM categorized_customers
GROUP BY frequency_category;
```
---

## Challenges & Resolutions
Challenge 1: DATE_FORMAT() not working
Initially, I tried grouping by: 

```
DATE_FORMAT(transaction_date, '%Y-%m')
```
But this raised syntax errors due to compatibility issues in the user's SQL environment (likely a version or client issue).

I replaced it with syntax below, which is more broadly supported and produces equivalent grouping:

```
YEAR(transaction_date), MONTH(transaction_date)
```
Challenge 2: Data Truncation Warning on AVG
When storing AVG(txn_count) into a temporary table, MySQL defaulted to an INTEGER column, causing rounding and truncation of decimal places.
Resolution: I explicitly defined the column as DECIMAL(10,2):

```
CREATE TEMPORARY TABLE avg_txn_per_customer (
    owner_id CHAR(32),
    avg_txn_per_month DECIMAL(10,2)
);
```
---
## Complete Query
CREATE DATABASE Assessment_Q2_SQL;

SELECT * FROM adashi_staging.savings_savingsaccount;
SELECT * FROM adashi_staging.users_customuser;

WITH monthly_transaction_counts AS (
    SELECT
        owner_id,
        YEAR(transaction_date) AS txn_year,
        MONTH(transaction_date) AS txn_month,
        COUNT(*) AS txn_count
    FROM savings_savingsaccount
    GROUP BY owner_id, YEAR(transaction_date), MONTH(transaction_date)
),
average_transactions AS (
    SELECT
        owner_id,
        AVG(txn_count) AS avg_txn_per_month
    FROM monthly_transaction_counts
    GROUP BY owner_id
),
categorized_customers AS (
    SELECT
        owner_id,
        CASE
            WHEN AVG(txn_count) >= 10 THEN 'High Frequency'
            WHEN AVG(txn_count) BETWEEN 3 AND 9 THEN 'Medium Frequency'
            ELSE 'Low Frequency'
        END AS frequency_category,
        AVG(txn_count) AS avg_transactions_per_month
    FROM monthly_transaction_counts
    GROUP BY owner_id
)
SELECT
    frequency_category,
    COUNT(*) AS customer_count,
    ROUND(AVG(avg_transactions_per_month), 1) AS avg_transactions_per_month
FROM categorized_customers
GROUP BY frequency_category;


