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
SELECT SUM(qty*price) as total_revenue_generated_before_discount FROM balanced_tree.sales ;
```
<img width="357" height="86" alt="image" src="https://github.com/user-attachments/assets/f9ff80ab-1ae3-4e35-ad8b-09fa9fd06077" />


**Question 3:** What was the total discount amount for all products?

```sql
SELECT SUM(qty*price*discount/100)::NUMERIC as total_discount FROM balanced_tree.sales
```
<img width="265" height="89" alt="image" src="https://github.com/user-attachments/assets/9786db69-c05e-47ea-8808-6bc0d992046e" />
