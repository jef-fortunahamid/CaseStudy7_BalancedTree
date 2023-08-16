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






