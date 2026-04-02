# Case Study 7 : Balanced Tree Clothing Company

https://8weeksqlchallenge.com/case-study-7/

---

## High Level Sales Analysis

**Question 1:** What was the total quantity sold for all products?

```sql
-- for all products
SELECT SUM(qty) as quantity_sold FROM balanced_tree.sales ;

--for each product
SELECT product_name, SUM(qty) as quantity_sold 
FROM balanced_tree.sales s LEFT JOIN 
balanced_tree.product_details pd ON s.prod_id = pd.product_id
GROUP BY product_name;
```
<img width="278" height="94" alt="image" src="https://github.com/user-attachments/assets/4c78ac78-b4d0-43b1-9688-a738848675f1" />
<img width="1522" height="578" alt="image" src="https://github.com/user-attachments/assets/87595d01-4be0-4a40-9f30-9590b6b88239" />

**Question 2:** What is the total generated revenue for all products before discounts?

```sql
--for all products
SELECT SUM(qty*price) as total_revenue_generated_before_discount FROM balanced_tree.sales ;

--breakdown for each product
SELECT product_name, SUM(qty*s.price) as total_revenue_generated_before_discount FROM balanced_tree.sales s LEFT JOIN 
balanced_tree.product_details pd ON s.prod_id = pd.product_id
GROUP BY product_name;
```
<img width="357" height="86" alt="image" src="https://github.com/user-attachments/assets/f9ff80ab-1ae3-4e35-ad8b-09fa9fd06077" />
<img width="1204" height="576" alt="image" src="https://github.com/user-attachments/assets/3fcc4d57-e33e-43fe-a735-a48aae73c0d0" />


**Question 3:** What was the total discount amount for all products?

```sql

--for all products
SELECT SUM(qty*price*discount/100)::NUMERIC as total_discount FROM balanced_tree.sales

--for each product
SELECT product_name, SUM((discount)*qty*s.price/100)::NUMERIC as total_discount
FROM balanced_tree.sales s LEFT JOIN 
balanced_tree.product_details pd ON s.prod_id = pd.product_id
GROUP BY product_name;
```
<img width="265" height="89" alt="image" src="https://github.com/user-attachments/assets/9786db69-c05e-47ea-8808-6bc0d992046e" />
<img width="1550" height="586" alt="image" src="https://github.com/user-attachments/assets/cdf5e0e5-f922-421c-a827-1b49b39c7ef7" />


---

## Transaction Analysis

**Question 1:** How many unique transactions were there?

```sql
SELECT COUNT(DISTINCT txn_id) as unique_transactions_count FROM balanced_tree.sales;
```
<img width="390" height="100" alt="image" src="https://github.com/user-attachments/assets/327a8412-0a6a-4e1a-b0a2-429ae8e825b7" />

---
**Question 2:** What is the average unique products purchased in each transaction?
```sql
-- formula avg = Sum of  all the unique products in each transaction / total number of  transactions
WITH pc As
(
SELECT  txn_id, COUNT(DISTINCT prod_id) as product_count
FROM balanced_tree.sales
GROUP BY txn_id
)

SELECT ROUND(SUM(product_count)/(SELECT COUNT(DISTINCT txn_id) FROM balanced_tree.sales),2) as average_unique_products FROM pc

```
<img width="383" height="103" alt="image" src="https://github.com/user-attachments/assets/08503981-efc2-4fc4-8707-cd3b6fba1848" />

---
**Question 3:** What are the 25th, 50th and 75th percentile values for the revenue per transaction?

```sql
WITH txn_revenue AS
(
SELECT txn_id, SUM(price*qty) as revenue
FROM balanced_tree.sales  
GROUP BY txn_id
)

SELECT 
percentile_cont(0.25) WITHIN GROUP(ORDER BY revenue) as percentile_25,
percentile_cont(0.5) WITHIN GROUP(ORDER BY revenue) as percentile_50,
percentile_cont(0.75) WITHIN GROUP(ORDER BY revenue) as percentile_75
FROM txn_revenue
```
Percentiles tell you how values are distributed across your dataset
Transactions below 375.75 → lower 25% (smallest revenues)
Transactions between 375.75 and 509.5 → 25%–50% (moderate-low revenues)
Transactions between 509.5 and 647 → 50%–75% (moderate-high revenues)
Transactions above 647 → top 25% (highest revenues)

<img width="1410" height="104" alt="image" src="https://github.com/user-attachments/assets/fcaeb3d1-078a-44c3-9ada-7b7f055f59ba" />

---
**Question 4:** What is the average discount value per transaction?
```sql
WITH discount As
(
SELECT  txn_id, SUM(qty*price*discount/100) as discount_value
FROM balanced_tree.sales
GROUP BY txn_id
)

SELECT ROUND(AVG(discount_value)) as average_discount_value FROM discount
```
<img width="476" height="96" alt="image" src="https://github.com/user-attachments/assets/7154147d-ed97-419b-a235-995927a934e5" />

---
**Question 5:** What is the percentage split of all transactions for members vs non-members?
```sql

WITH members AS
(
SELECT  SUM(case WHEN member ='t' THEN 1 END) as members , SUM(case WHEN member ='f' THEN 1 END) as non_members , COUNT(txn_id) as total_txns 
FROM balanced_tree.sales
)


SELECT members,non_members,total_txns,  ROUND((members::NUMERIC/total_txns::NUMERIC)*100,2) as members_percentage ,ROUND((non_members::NUMERIC/total_txns::NUMERIC)*100,2) as non_members_percentage
FROM members
```
<img width="1618" height="103" alt="image" src="https://github.com/user-attachments/assets/2236a4a2-4728-4c2b-b057-b20f63dd9cd7" />

---
**Question 6:** What is the average revenue for member transactions and non-member transactions?
```sql
WITH revenue AS
(
SELECT  SUM(CASE WHEN member ='t' THEN qty*price END) as total_members_revenue  ,SUM(CASE WHEN member ='f' THEN qty*price END) AS total_non_members_revenue , COUNT( DISTINCT CASE WHEN member ='t' THEN txn_id END) as member_txns_count,COUNT(DISTINCT CASE WHEN member ='f' THEN txn_id END) as non_member_txns_count
FROM balanced_tree.sales
)

SELECT 
total_members_revenue,total_non_members_revenue,member_txns_count,non_member_txns_count,
ROUND((total_members_revenue/member_txns_count)::NUMERIC,2) as avg_members_revenue,
ROUND((total_non_members_revenue/non_member_txns_count)::NUMERIC,2) as avg_non_members_revenue
from revenue

```
<img width="1661" height="101" alt="image" src="https://github.com/user-attachments/assets/60dccd0b-59ca-4bf4-ac8b-e14e5f4778bd" />

---
## Product Analysis

**Question 1:** What are the top 3 products by total revenue before discount?
```sql

SELECT prod_id, product_name, SUM(qty*s.price) as total_revenue 
FROM 
balanced_tree.sales  s LEFT JOIN
balanced_tree.product_details pd ON
s.prod_id =pd.product_id
GROUP BY prod_id,product_name
ORDER BY total_revenue DESC
LIMIT 3
```
<img width="1441" height="217" alt="image" src="https://github.com/user-attachments/assets/b37335be-73e9-48e7-8eaf-31fa870cc176" />

---

**Question 2:** What is the total quantity, revenue and discount for each segment?
```sql
SELECT segment_id, segment_name, SUM(qty) as total_quantity, SUM(qty*s.price) as total_revenue ,
SUM(qty*s.price*discount)/100 as total_discount
FROM 
balanced_tree.sales  s LEFT JOIN
balanced_tree.product_details pd ON
s.prod_id =pd.product_id
GROUP BY segment_id,segment_name
ORDER BY segment_id
```
<img width="1520" height="258" alt="image" src="https://github.com/user-attachments/assets/335b569b-952b-43dd-a684-e854576c04e2" />

---
**Question 3:** What is the top selling product for each segment?
```sql
-- I calculated the top selling product based on revenue , the question is ambigious , we an also calculate top selling product based on quantity
WITH products AS(
  
SELECT segment_id, segment_name,product_name, SUM(qty*s.price) as total_revenue ,
RANK() OVER(PARTITION BY segment_name ORDER BY SUM(qty*s.price) DESC) as rank
FROM 
balanced_tree.sales  s LEFT JOIN
balanced_tree.product_details pd ON
s.prod_id =pd.product_id
GROUP BY segment_id,segment_name,product_name
ORDER BY segment_id
  )
  
SELECT segment_name,product_name,total_revenue FROM products WHERE rank =1
```
<img width="1426" height="232" alt="image" src="https://github.com/user-attachments/assets/4b55a193-fd09-4bab-aa65-ae277ff19c77" />

---
**Question 4:** What is the total quantity, revenue and discount for each category?
```sql
SELECT category_name, SUM(qty) as total_quantity, SUM(qty*s.price) as total_revenue ,
SUM(qty*s.price*discount)/100 as total_discount
FROM 
balanced_tree.sales  s LEFT JOIN
balanced_tree.product_details pd ON
s.prod_id =pd.product_id
GROUP BY category_name
ORDER BY category_name
```
<img width="1509" height="153" alt="image" src="https://github.com/user-attachments/assets/99d11c2e-b9c9-46b7-9df6-cbf8edd5cc2f" />

---
**Question 5:** What is the top selling product for each category?
```sql
-- I calculated the top selling product based on revenue , the question is ambigious , we an also calculate top selling product based on quantity
WITH products AS(
  
SELECT category_name,product_name, SUM(qty*s.price) as total_revenue ,
RANK() OVER(PARTITION BY category_name ORDER BY SUM(qty*s.price) DESC) as rank
FROM 
balanced_tree.sales  s LEFT JOIN
balanced_tree.product_details pd ON
s.prod_id =pd.product_id
GROUP BY category_name,product_name
ORDER BY category_name
  )
  
SELECT category_name,product_name,total_revenue FROM products WHERE rank =1
```
<img width="1563" height="194" alt="image" src="https://github.com/user-attachments/assets/6409d4d1-8e47-47b8-a8ba-f47462facff4" />

---
**Question 6:** What is the percentage split of revenue by product for each segment?
```sql

WITH revenue AS
(
SELECT  segment_name,product_name,
SUM(qty*s.price) as product_revenue
FROM 
balanced_tree.sales  s LEFT JOIN
balanced_tree.product_details pd ON
s.prod_id =pd.product_id
GROUP BY segment_name,product_name
)

SELECT segment_name, product_name, product_revenue,
ROUND((product_revenue/
SUM(product_revenue) OVER(PARTITION BY segment_name))::NUMERIC*100,2) as revenue_percentage_split
FROM revenue
ORDER BY segment_name

```
---
**Question 7:** What is the percentage split of revenue by segment for each category?
```sql
```
---
**Question 8:** What is the percentage split of total revenue by category?
```sql
WITH revenue AS
(
SELECT  SUM(CASE WHEN category_name = 'Mens' THEN (qty*s.price) END) as  mens_total_revenue ,
SUM(CASE WHEN category_name = 'Womens' THEN (qty*s.price) END) as  womens_total_revenue ,
SUM(qty*s.price) as total_revenue
FROM 
balanced_tree.sales  s LEFT JOIN
balanced_tree.product_details pd ON
s.prod_id =pd.product_id
)

SELECT  ROUND((mens_total_revenue::NUMERIC/total_revenue::NUMERIC)*100,2) AS mens_revenue_percentage,ROUND((womens_total_revenue::NUMERIC/total_revenue::NUMERIC)*100,2) AS womens_revenue_percentage FROM revenue
```
<img width="1377" height="96" alt="image" src="https://github.com/user-attachments/assets/d9af774a-b4fc-40a4-b8b6-befb2b604b7a" />

---
**Question 9:** What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
```sql
```
---
**Question 10:** What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

```sql
```
---
