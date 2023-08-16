# Case Study #7 Balanced Tree

*Note: All information and data related to the case study were obtained from [here](https://8weeksqlchallenge.com/case-study-7/).*
![Screenshot 2023-08-13 at 8 38 52 pm](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/99962f3b-57c9-45ea-b568-2e4fc4fd8c57)

## Business Task
Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!
Danny has tasked us with supporting the merchandising teams. Our mission is to delve into the sales data, analyze the performance, and produce a straightforward financial report that can be shared with the entire organization.

## General Insights
- *Product Heirarchy:* he SQL operations centered around creating a complete and hierarchical representation of products from the product_hierarchy table. This involved extracting and associating categories, segments, and styles.
- *Price Integration:* By merging the hierarchical structure with the product_prices table, there was an integration of pricing details with the product hierarchy. This allows for a clearer understanding of the product assortment alongside their respective pricing.
- *Product Naming Convention:* The query aimed to formulate a naming convention for products by concatenating style, segment, and category names. This provides a unified and descriptive naming structure that can be invaluable for frontend displays or customer interactions.

## Key SQL Syntax and Functions
- Joins(`INNER JOIN`, `CROSS JOIN`)
- Recursive Queries (`WITH RECURSIVE`)
- Analytical Function (`PERCENTILE_CONT`)
- Aggregation Functions (`SUM`, `COUNT`, `AVG`)
- Window Functions (`ROW_NUMBER`, `RANK`)
- Set Operations (`UNION`, `UNION ALL`)
- Temporary Table (`CREATE TEMP TABLE`)
- Array Functions (`UNNEST`)
- String Manmipulation (`CONCAT`)
- Conditional Logic (`CASE WHEN`)
- Common Table Expressions (CTE)

## Questions and Solutions
### Part A: High Level Sales Analysis
> 1. What was the total quantity sold for all products?
```sql
SELECT
    product_details.product_name
  , SUM(sales.qty) AS total_quantity
FROM balanced_tree.sales
INNER JOIN balanced_tree.product_details
  ON sales.prod_id = product_details.product_id
GROUP BY product_details.product_name
ORDER BY total_quantity DESC;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/a272d6f1-8ba6-4cdd-a5c0-6b97821bb3ad)

> 2. What is the total generated revenue for all products before discounts?
```sql
SELECT
  SUM(qty * price) AS total_revenue
FROM balanced_tree.sales
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/17058b5e-00ae-4c64-9732-c65bb549ee65)

> 3. What was the total discount amount for all products?
```sql
SELECT
  ROUND(SUM(qty * (price * (discount::NUMERIC/100))), 2) AS total_discount_amount
FROM balanced_tree.sales;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/0df365bb-40bf-4359-a754-1d520d09b3c8)

### Part B: Transaction Analysis
> 1. How many unique transactions were there?
```sql
SELECT
  COUNT(DISTINCT txn_id) AS unique_txn_count
FROM balanced_tree.sales;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/ed59da11-e138-4e82-92f2-1fb2cc0944bc)

> 2. What is the average unique products purchased in each transaction?
```sql
WITH product_count_per_transaction AS (
  SELECT
      txn_id
    , COUNT(1) AS prod_count
  FROM balanced_tree.sales
  GROUP By txn_id
)
SELECT
  ROUND(AVG(prod_count)) AS avg_count_per_txn
FROM product_count_per_transaction;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/c0c3661f-8b41-432d-97cd-4a99f8b32fe7)

> 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?
```sql
WITH transaction_revenue AS (
  SELECT
      txn_id
    , SUM(qty * price) AS revenue
  FROM balanced_tree.sales
  GROUP BY txn_id
)
SELECT
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY revenue) AS pct_25
  , PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY revenue) AS pct_50
  , PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY revenue) AS pct_75
FROM transaction_revenue;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/9f7dbd3b-712e-4e70-bd57-a1e3ed8620b9)

> 4. What is the average discount value per transaction?
```sql
WITH transaction_discount AS (
SELECT
    txn_id
  , SUM(qty * (price * (discount::NUMERIC / 100))) AS discount_amount
FROM balanced_tree.sales
GROUP BY txn_id
)
SELECT
  ROUND(AVG(discount_amount), 2) AS avg_dkiscount_amnount
FROM transaction_discount;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/21c51123-457d-4287-b5c9-76ef8aef50e3)

> 5. What is the percentage split of all transactions for members vs non-members?
```sql
WITH member_nonmember AS (
  SELECT
      member
    , COUNT(DISTINCT txn_id) AS transactions
  FROM balanced_tree.sales
  GROUP BY member
),
total_transactions AS (
  SELECT
    SUM(transactions) AS total
  FROM member_nonmember
)
SELECT
    member
  , transactions
  , ROUND(100 * transactions / total) as percentage
FROM member_nonmember, total_transactions;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/19115fad-78f5-4d4c-b1bd-431a3c038410)

> 6. What is the average revenue for member transactions and non-member transactions?
```sql
WITH revenue_per_members_transaction AS (
  SELECT
      member
    , txn_id
    , SUM(qty * price) AS transaction_revenue
  FROM balanced_tree.sales
  GROUP BY member, txn_id
)
SELECT
    member
  , ROUND(AVG(transaction_revenue), 2) AS avg_revenue
FROM revenue_per_members_transaction
GROUP BY member;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/67338bed-783f-4e9a-abcf-06b807d19398)

### Part C: Product Analysis
> 1. What are the top 3 products by total revenue before discount?
```sql
SELECT 
    sales.prod_id AS product_id
  , product_details.product_name
  , SUM(sales.qty * sales.price) AS total_revenue
FROM balanced_tree.sales
INNER JOIN balanced_tree.product_details
  ON sales.prod_id = product_details.product_id
GROUP BY sales.prod_id, product_details.product_name
ORDER BY total_revenue DESC
LIMIT 3;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/96541b15-610e-4432-bc8c-63c4a365d8a5)

> 2. What is the total quantity, revenue and discount for each segment?
```sql
SELECT
    product_details.segment_id
  , product_details.segment_name
  , SUM(sales.qty) AS total_quantity
  , SUM(sales.qty * sales.price) AS total_revenue
  , ROUND(SUM(sales.qty * (sales.price * (sales.discount::NUMERIC / 100))), 2) AS total_discount
FROM balanced_tree.sales
INNER JOIN balanced_tree.product_details
  ON sales.prod_id = product_details.product_id
GROUP BY 
    product_details.segment_id
  , product_details.segment_name
ORDER BY total_revenue DESC;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/ed849299-78c1-4657-88e2-5ca40c11e1d0)

> 3. What is the top selling product for each segment?
```sql
WITH segment_product_ranking AS (
SELECT
    product_details.segment_id
  , product_details.segment_name
  , sales.prod_id AS product_id
  , product_details.product_name
  , SUM(sales.qty) AS product_quantity
  , ROW_NUMBER() OVER (
                PARTITION BY product_details.segment_id 
                ORDER BY SUM(sales.qty) DESC
              ) AS _ranking
FROM balanced_tree.sales
INNER JOIN balanced_tree.product_details
  ON sales.prod_id = product_details.product_id
GROUP BY 
    product_details.segment_id
  , product_details.segment_name
  , sales.prod_id
  , product_details.product_name
)
SELECT
    segment_id
  , segment_name
  , product_id
  , product_name
  , product_quantity
FROM segment_product_ranking
WHERE _ranking = 1
ORDER BY product_quantity DESC;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/7e0122be-91ab-4ecc-ae4a-0fcb293c942f)

> 4. What is the total quantity, revenue and discount for each category?
```sql
SELECT
    product_details.category_id
  , product_details.category_name
  , SUM(sales.qty) AS total_quantity
  , SUM(sales.qty * sales.price) AS total_revenue
  , ROUND(SUM(sales.qty * (sales.price * (sales.discount::NUMERIC / 100))), 2) AS total_discount
FROM balanced_tree.sales
INNER JOIN balanced_tree.product_details
  ON sales.prod_id = product_details.product_id
GROUP BY
    product_details.category_id
  , product_details.category_name
ORDER BY total_revenue DESC;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/50f05785-2301-4d77-9def-6aa5abac9634)

> 5. What is the top selling product for each category?
```sql
WITH category_product_ranking AS (
SELECT
    product_details.category_id
  , product_details.category_name
  , sales.prod_id AS product_id
  , product_details.product_name
  , SUM(sales.qty) AS total_quantity
  , RANK() OVER (
          PARTITION BY product_details.category_id
          ORDER BY SUM(sales.qty) DESC
        ) AS _ranking
FROM balanced_tree.sales
INNER JOIN balanced_tree.product_details
  ON sales.prod_id = product_details.product_id
GROUP BY 
    product_details.category_id
  , product_details.category_name
  , sales.prod_id
  , product_details.product_name
)
SELECT
    category_id
  , category_name
  , product_id
  , product_name
  , total_quantity
FROM category_product_ranking
WHERE _ranking = 1;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/d373a723-e56c-4d69-87b3-5b6d6d81e09a)

> 6. What is the percentage split of revenue by product for each segment?
```sql
WITH segment_product_revenue AS (
  SELECT
      product_details.segment_id
    , product_details.segment_name
    , sales.prod_id AS product_id
    , product_details.product_name
    , SUM(sales.qty * sales.price) AS product_revenue
  FROM balanced_tree.sales
  INNER JOIN balanced_tree.product_details
    ON sales.prod_id = product_details.product_id
  GROUP BY
      product_details.segment_id
    , product_details.segment_name
    , sales.prod_id
    , product_details.product_name
),
segment_revenue AS (
  SELECT
      segment_id
    , SUM(product_revenue) AS product_total_revenue
  FROM segment_product_revenue
  GROUP BY segment_id
)
SELECT 
    spr.segment_id
  , spr.segment_name
  , spr.product_id
  , spr.product_name
  , spr.product_revenue
  , ROUND(100 * spr.product_revenue::NUMERIC / sr.product_total_revenue, 2) AS segment_product_percentage
FROM segment_product_revenue AS spr
INNER JOIN segment_revenue AS sr
  USING (segment_id)
ORDER BY 
    segment_id
  , product_revenue DESC;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/f0b08301-19b7-4d9c-9ead-24f04b5bf610)

> 7. What is the percentage split of revenue by segment for each category?
```sql
WITH category_segment_revenue AS (
  SELECT
      product_details.category_id
    , product_details.category_name
    , product_details.segment_id
    , product_details.segment_name
    , SUM(sales.qty * sales.price) AS segment_revenue
  FROM balanced_tree.sales
  INNER JOIN balanced_tree.product_details
    ON sales.prod_id = product_details.product_id
  GROUP BY
      product_details.category_id
    , product_details.category_name
    , product_details.segment_id
    , product_details.segment_name
)
,
category_revenue AS (
  SELECT
      category_id
    , SUM(segment_revenue) AS segment_total_revenue
  FROM category_segment_revenue
  GROUP BY category_id
)
SELECT 
    csr.category_id
  , csr.category_name
  , csr.segment_id
  , csr.segment_name
  , csr.segment_revenue
  , ROUND(100 * csr.segment_revenue::NUMERIC / cr.segment_total_revenue, 2) AS category_segment_percentage
FROM category_segment_revenue AS csr
INNER JOIN category_revenue AS cr
  USING (category_id)
ORDER BY 
    category_id
  , segment_revenue DESC;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/1b982b0d-ce63-4ef4-804a-ecdcc8ceef03)

> 8. What is the percentage split of total revenue by category?
```sql
WITH category_revenue AS (
  SELECT
      product_details.category_id
    , product_details.category_name
    , SUM(sales.qty * sales.price) AS revenue
  FROM balanced_tree.sales
  INNER JOIN balanced_tree.product_details
    ON sales.prod_id = product_details.product_id
  GROUP BY
      product_details.category_id
    , product_details.category_name
),
total_revenue AS (
  SELECT
      SUM(revenue) AS total_revenue
  FROM category_revenue
)
SELECT 
    cr.category_id
  , cr.category_name
  , cr.revenue
  , ROUND(100 * cr.revenue::NUMERIC / tr.total_revenue, 2) AS category_percentage
FROM category_revenue AS cr, total_revenue AS tr
ORDER BY category_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/523de02d-eadc-488d-91f5-05ce17f58370)

> 9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
```sql
WITH product_transaction AS (
  SELECT
      sales.prod_id AS product_id
    , product_details.product_name
    , COUNT(DISTINCT sales.txn_id) AS product_transaction_count
  FROM balanced_tree.sales
  INNER JOIN balanced_tree.product_details
    ON sales.prod_id = product_details.product_id
  WHERE qty >= 1
  GROUP BY 
      sales.prod_id
    , product_details.product_name
), 
total_product_transaction AS (
  SELECT
    COUNT(DISTINCT sales.txn_id) AS total_transaction
  FROM balanced_tree.sales
)
SELECT
    product_id
  , product_name
  , ROUND(100 * product_transaction_count::NUMERIC / total_transaction, 2 ) AS penetration_percentage
FROM product_transaction, total_product_transaction
ORDER BY penetration_percentage DESC;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/431acfe5-d158-4f9a-b993-045c3587f964)

> 10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction? Super bonus - what are the quantity, revenue, discount and net revenue from the top 3 products in the transactions where all 3 were purchased?
```sql
DROP TABLE IF EXISTS temp_product_combos;
CREATE TEMP TABLE temp_product_combos AS
WITH RECURSIVE input(product) AS (
  SELECT product_id::TEXT
  FROM balanced_tree.product_details
),
output_table AS (
  SELECT
      ARRAY[product] AS combo
    , product
    , 1 AS product_counter
  FROM input
UNION 
  SELECT
      ARRAY_APPEND(output_table.combo, input.product)
    , input.product
    , product_counter + 1
  FROM output_table
  INNER JOIN input
    ON input.product > output_table.product
  WHERE output_table.product_counter <= 2
)
SELECT *
FROM output_table
WHERE product_counter = 3;

WITH transaction_products AS (
  SELECT
      txn_id
    , ARRAY_AGG(prod_id::TEXT ORDER BY prod_id) AS products 
  FROM balanced_tree.sales
  GROUP BY txn_id
),
combo_transactions AS (
  SELECT
      txn_id
    , combo
    , products 
  FROM transaction_products
  CROSS JOIN temp_product_combos
  WHERE combo <@ products 
),
ranked_combos AS (
  SELECT
    combo
  , COUNT(DISTINCT txn_id) AS transaction_count
  , RANK() OVER (ORDER BY COUNT(DISTINCT txn_id) DESC) AS combo_rank
  , ROW_NUMBER() OVER (ORDER BY COUNT(DISTINCT txn_id) DESC) AS combo_id
  FROM combo_transactions
  GROUP BY combo
),
most_common_combo_product_transactions AS (
  SELECT
      combo_transactions.txn_id
    , ranked_combos.combo_id
    , UNNEST(ranked_combos.combo) AS prod_id
  FROM combo_transactions
  INNER JOIN ranked_combos
    ON combo_transactions.combo = ranked_combos.combo
  WHERE ranked_combos.combo_rank = 1
)
SELECT
    product_details.product_id
  , product_details.product_name
  , COUNT(DISTINCT sales.txn_id) AS combo_transaction_count 
  , SUM(sales.qty) AS quantity
  , SUM(sales.qty * sales.price) AS revenue 
  , ROUND(
        SUM(sales.qty * sales.price * sales.discount::NUMERIC / 100),
        2
      ) AS discount
  , ROUND(
        SUM(sales.qty * sales.price * (1 - sales.discount::NUMERIC / 100)),
        2
      ) AS net_revenue
FROM balanced_tree.sales
INNER JOIN most_common_combo_product_transactions AS top_combo
  ON sales.txn_id = top_combo.txn_id
  AND sales.prod_id = top_combo.prod_id
INNER JOIN balanced_tree.product_details
  ON sales.prod_id = product_details.product_id
GROUP BY
    product_details.product_id
  , product_details.product_name;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/646437d1-6b06-4223-bb1a-9256773f7566)

### Part D: Bonus Challenge
> > Use a single SQL query to transform the product_hierarchy and product_prices datasets to the product_details table.
> 
> Hint: you may want to consider using a recursive CTE to solve this problem!

```sql
DROP TABLE IF EXISTS temp_product_details;
CREATE TEMP TABLE temp_product_details AS
WITH RECURSIVE output_table (id, category_id, segment_id, style_id, category_name, segment_name, style_name)
AS (
  SELECT
      id
    , id AS category_id
    , NULL::INT AS segment_id
    , NULL::INT AS style_id
    , level_text AS segment_name
    , NULL AS segment_name
    , NULL AS style_name
  FROM balanced_tree.product_hierarchy
  WHERE level_name = 'Category'
  
UNION ALL
  
  SELECT
      product_hierarchy.id
    , CASE WHEN product_hierarchy.level_name = 'Category'
            THEN product_hierarchy.id
            ELSE output_table.category_id
          END AS category_id
    , CASE WHEN product_hierarchy.level_name = 'Segment'
            THEN product_hierarchy.id
            ELSE output_table.segment_id
          END AS segment_id
    , CASE WHEN product_hierarchy.level_name = 'Style'
            THEN product_hierarchy.id
            ELSE output_table.style_id
          END AS style_id
    , CASE WHEN product_hierarchy.level_name = 'Category'
            THEN product_hierarchy.level_text
            ELSE output_table.category_name
          END AS category_name
    , CASE WHEN product_hierarchy.level_name = 'Segment'
            THEN product_hierarchy.level_text
            ELSE output_table.segment_name
          END AS segment_name
    , CASE WHEN product_hierarchy.level_name = 'Style'
            THEN product_hierarchy.level_text
            ELSE output_table.style_name
          END AS style_name
  FROM output_table
  INNER JOIN balanced_tree.product_hierarchy
    ON output_table.id = product_hierarchy.parent_id
    AND product_hierarchy.parent_id IS NOT NULL
)
SELECt
    product_prices.product_id
  , product_prices.price
  , CONCAT(style_name, ' ', segment_name , ' - ', category_name) AS product_name 
  , category_id 
  , segment_id
  , style_id
  , category_name
  , segment_name
  , style_name
FROM output_table
INNER JOIN balanced_tree.product_prices
  ON output_table.id = product_prices.id
WHERE style_name IS NOT NULL;

SELECT * FROM temp_product_details;

SELECT * FROM balanced_tree.product_details;
```
![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/a03579c9-6e7b-4a08-8c81-af9a8023dc03)

![image](https://github.com/jef-fortunahamid/CaseStudy7_BalancedTree/assets/125134025/635568f2-58bb-40d7-8025-cc072663c6f9)
