# Case-Study-4-DATA-BANK
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

Customer Nodes Exploration

1. How many unique nodes are there on the Data Bank system?
```
SELECT COUNT(DISTINCT node_id) AS unique_nodes
FROM customer_nodes;
```

| metric        | value |
|---------------|-------|
| unique_nodes  | 5     |
