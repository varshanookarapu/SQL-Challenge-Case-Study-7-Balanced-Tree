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


