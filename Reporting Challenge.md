# Reporting Challenge

Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.

He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).

Feel free to split up your final outputs into as many tables as you need - but be sure to explicitly reference which table outputs relate to which question for full marks :)



----

To answer the reporting challenge first I will segregate the above questions to the following categories 

**Segment Summary**

1.What is the total quantity, revenue and discount for each segment?
2.What is the top selling product for each segment?
3.What is the percentage split of revenue by product for each segment?

**Category Summary**
1.What is the total quantity, revenue and discount for each category?
2.What is the top selling product for each category?
3.What is the percentage split of revenue by segment for each category?
4.What is the percentage split of total revenue by category?

**Product Summary**
1.What are the top 3 products by total revenue before discount?

My idea is to create three CTEs then filter the data for january month gather the insights , then proceed to check for other months , the minimal change that I would be making is just updating the month number and we generate the insights from the CTES. 

---
## Segment Analysis 

```sql
WITH segment_summary AS

(

WITH segment_summary AS
(
SELECT 
  
  segment_id, 
  segment_name,
  product_name, 
  EXTRACT('month' FROM start_txn_time) as month, 
  TO_CHAR(start_txn_time,'Month') as month_name,  
  SUM(qty) as total_quantity, SUM(qty*s.price) as total_revenue ,
  SUM(qty*s.price*discount*0.10) as total_discount,
  ROUND(((SUM(qty*s.price))/SUM((SUM(qty*s.price))) OVER(PARTITION BY segment_name))::NUMERIC*100,2) as revenue_percentage_split 

FROM
balanced_tree.sales  s LEFT JOIN
balanced_tree.product_details pd ON
s.prod_id =pd.product_id
GROUP BY segment_id,segment_name,product_name,month, month_name

)

SELECT *,RANK() OVER(PARTITION BY segment_name,month ORDER BY total_revenue DESC) as rank FROM segment_summary 
WHERE month=1
-- Change the month number to get the insights for other months.
ORDER BY segment_id,month

```
Above CTE gives us insights on 
--total_quantity 
--total revenue 
--total_discount 
--revenue percentage 
--rank 
of every product under each segment, you can easily get the insights for other months by simply changing the month number in the WHERE clause of the SQL query to get details for subsequents months like February and March.

<img width="1905" height="627" alt="image" src="https://github.com/user-attachments/assets/40a4856e-178c-4760-a0d9-c509b4e521ba" />

