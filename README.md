# Case-Study-4-DATA-BANK
![image alt](https://github.com/TemitopeHassan-Analyst/Case-Study-4-DATA-BANK/blob/cf27b6c6a8d6ffadc0fccfb8068467c673907691/Screenshot%202026-04-02%20155830.png)

## Introduction
Data Bank operates at the intersection of digital banking and cloud data storage. Unlike traditional financial institutions, Data Bank ties each customer's cloud storage allocation directly to their account balance; the more money a customer holds, the more storage they are entitled to.
While this model is innovative, it creates a forecasting challenge that the business has not yet solved.

The management team has two connected priorities. First, they want to grow their customer base. Second, and equally critical, they need to understand the storage implications of that growth. As customer numbers increase and account balances fluctuate, so does the total demand on Data Bank's storage infrastructure. Without a clear view of current usage patterns and future demand, the business risks either over-provisioning resources, wasting capital,  or under-provisioning them, failing customers at the worst possible moment.

## Business Problem
The core questions the business needs answered are:

- How many customers does Data Bank currently serve, and how are they distributed across regions and nodes?
- How do customer account balances behave over time — are they growing, declining, or volatile?
- Based on current balance trends and customer growth rates, how much cloud storage will the business need to provision in the future?
- Are there patterns in customer behavior that can help the business plan more precisely rather than relying on broad estimates?

## Objectives
The objective of this analysis is to move Data Bank from reactive infrastructure planning to data-driven forecasting, giving the management team the metrics, trends, and projections they need to make confident decisions about growth, resource allocation, and long-term platform development.

## Entity Relationship Diagram
The 3 key datasets for this case study include:
- Regions
- Customer nodes
- Customer transactions

![image alt](https://github.com/TemitopeHassan-Analyst/Case-Study-4-DATA-BANK/blob/122453ff7d0c6e6be6d2889f9ed46fa914c356e4/Screenshot%202026-04-02%20002628.png)

Table 1: regions

This regions table contains the region_id and their respective region_name values.

![image alt](https://github.com/TemitopeHassan-Analyst/Case-Study-4-DATA-BANK/blob/db478add1afaca4c7c27d9a940b08bbcdb1e9878/Screenshot%202026-04-02%20004405.png)

Table 2: customer_nodes

Customers are randomly distributed across the nodes according to their region. This random distribution changes frequently to reduce the risk of hackers getting into Data Bank’s system and stealing customer’s money and data!

![image alt](https://github.com/TemitopeHassan-Analyst/Case-Study-4-DATA-BANK/blob/1bdddfcada2d61cfe156679cf335e30747ac3f5e/Screenshot%202026-04-02%20004439.png)

Table 3: Customer Transactions

This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.

![image alt](https://github.com/TemitopeHassan-Analyst/Case-Study-4-DATA-BANK/blob/29fd9f84b8ee03656bc76788c389335cdc5a4e1e/Screenshot%202026-04-02%20004516.png)

### Questions and Solutions

A. Customer Nodes Exploration

1. How many unique nodes are there on the Data Bank system?
```
SELECT COUNT(DISTINCT node_id) AS unique_nodes
FROM customer_nodes;
```

| metric        | value |
|---------------|-------|
| unique_nodes  | 5     |

2. What is the number of nodes per region?
```
SELECT
  r.region_name,
  COUNT(DISTINCT cn.node_id) AS node_count
FROM regions AS r
JOIN customer_nodes AS cn
  ON r.region_id = cn.region_id
GROUP BY r.region_name;
```
| region_name | node_count |
|-------------|------------|
| Africa      | 5          |
| America     | 5          |
| Asia        | 5          |
| Europe      | 5          |
| Oceania     | 5          |

3. How many customers are allocated to each region

```
SELECT
  r.region_name,
  COUNT(DISTINCT cn.customer_id) AS customer_count
FROM regions AS r
JOIN customer_nodes AS cn
  ON r.region_id = cn.region_id
GROUP BY r.region_name
ORDER BY customer_count DESC;
```

| region_name | customer_count |
|-------------|----------------|
| Australia   | 110            |
| America     | 105            |
| Africa      | 102            |
| Asia        | 95             |
| Europe      | 88             |

4. How many days on average are customers reallocated to a different node?
```
WITH node_days AS (
  SELECT
    customer_id,
    node_id,
    DATEDIFF(DAY, start_date, end_date) AS days_in_node
  FROM data_bank.dbo.customer_nodes
  WHERE end_date != '9999-12-31'
  GROUP BY customer_id, node_id, start_date, end_date
),
total_node_days AS (
  SELECT
    customer_id,
    node_id,
    SUM(days_in_node) AS total_days_in_node
  FROM node_days
  GROUP BY customer_id, node_id
)
SELECT
  ROUND(AVG(CAST(total_days_in_node AS FLOAT)), 0) AS avg_node_reallocation_days
FROM total_node_days;
```

| metric                      | value |
|-----------------------------|-------|
| avg_node_reallocation_days  | 24    |

B. Customer Transactions

1. What is the unique count and total amount for each transaction type?
```
SELECT
  txn_type,
  COUNT(*) AS transaction_count,
  SUM(txn_amount) AS transaction_amount
FROM customer_transactions
GROUP BY txn_type;
```

| txn_type   | transaction_count | transaction_amount |
|------------|-------------------|--------------------|
| deposit    | 2671              | 1,359,168          |
| purchase   | 1617              | 806,537            |
| withdrawal | 1580              | 793,003            |

2. What is the average total historical deposit count and amounts for all customers?
```
WITH deposits AS (
  SELECT
    customer_id,
    COUNT(customer_id) AS txn_count,
    AVG(CAST(txn_amount AS FLOAT)) AS avg_amount
  FROM customer_transactions
  WHERE txn_type = 'deposit'
  GROUP BY customer_id
)
SELECT
  ROUND(AVG(CAST(txn_count AS FLOAT)), 0) AS avg_deposit_count,
  ROUND(AVG(avg_amount), 0) AS avg_deposit_amt
FROM deposits;

```

| metric             | value   |
|--------------------|---------|
| avg_deposit_count  | 5       |
| avg_deposit_amt    | 509     |

3. For each month, how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```
WITH monthly_transactions AS (
  SELECT
    customer_id,
    MONTH(txn_date) AS mth,
    SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count,
    SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
    SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count
  FROM customer_transactions
  GROUP BY customer_id, MONTH(txn_date)
)
SELECT
  mth,
  COUNT(DISTINCT customer_id) AS customer_count
FROM monthly_transactions
WHERE deposit_count > 1
  AND (purchase_count >= 1 OR withdrawal_count >= 1)
GROUP BY mth
ORDER BY mth;
```

| mth | customer_count |
|-----|----------------|
| 1   | 168            |
| 2   | 181            |
| 3   | 192            |
| 4   | 70             |

8. What is the closing balance for each customer at the end of the month? Also show the change in balance each month in the same table output.
```
WITH monthly_data AS (
    SELECT 
        customer_id,
        EOMONTH(txn_date) AS month_end,
        SUM(CASE 
            WHEN txn_type IN ('withdrawal', 'purchase') THEN -txn_amount
            ELSE txn_amount 
        END) AS monthly_change
    FROM data_bank.dbo.customer_transactions
    GROUP BY customer_id, EOMONTH(txn_date)
)

SELECT 
    customer_id,
    month_end,
    monthly_change,
    SUM(monthly_change) OVER (
        PARTITION BY customer_id 
        ORDER BY month_end
    ) AS ending_balance
FROM monthly_data
ORDER BY customer_id, month_end;
```

![image alt](https://github.com/TemitopeHassan-Analyst/Case-Study-4-DATA-BANK/blob/bceeaf62c34511aba581f3c177d93bad6f3f50fd/Screenshot%202026-04-02%20181740.png)

### Key Insights

1. Customer balance behavior is uneven
- A small group of customers usually holds a large portion of the total balances
- Many customers maintain low or near-zero balances
- Customer balances directly impact storage allocation

2. Cash flow volatility across months
- Some customers show large swings in monthly inflows and outflows
- Others remain stable with consistent deposits or withdrawals

3. Dormant or low-activity accounts exist
- A portion of customers show no transactions in certain months
- These accounts contribute little to revenue or engagement
Indicates possible inactivity or churn risk

4. Customer contribution is not equal
- A few customers consistently generate high positive balance growth
- Many contribute minimally or fluctuate around zero
Shows the need for customer segmentation

5. Net monthly balance changes vary significantly
- Some months show net inflow dominance (growth months)
- Others show net outflow dominance (decline months)
- Suggests seasonal or behavioral patterns in customer spending

### Recommendations

1. Segment customers for targeted strategy, which helps tailor marketing and retention strategies.

2. Retain high-value customers by offering loyalty benefits or lower fees

3. Reactivate dormant accounts

4. Improve cash flow stability

5. Monitor monthly balance trends by building dashboards tracking Net inflow/outflow per month

