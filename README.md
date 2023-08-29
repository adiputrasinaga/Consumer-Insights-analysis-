**Consumer Insights Analysis**
<br/>

**1. Provide the list of markets in which customer "Atliq Exclusive" operates its
business in the APAC region**
````sql
select market from dim_customer
where customer = 'Atliq Exclusive' and region = 'APAC'
group by market
order by market;
````
**2. What is the percentage of unique product increase in 2021 vs. 2020?**
````sql
select X.A as unique_product_2020, Y.B as unique_products_2021, round((B-A)*100/A,2) as percentage_chg
from 
(
(select count(distinct(product_code)) as A from fact_sales_monthly 
where fiscal_year = 2020) X,
(select count(distinct(product_code)) as B from fact_sales_monthly
where fiscal_year = 2021) Y
) 
````
**3. Provide a report with all the unique product counts for each segment and
sort them in descending order of product counts**
````sql
SELECT segment, 
COUNT(DISTINCT(product_code)) AS product_count FROM dim_product
GROUP BY segment
ORDER BY product_count DESC;
````
**4. Which segment had the most increase in unique products in
2021 vs 2020?**
````sql
 with CTE1 as 
 (select P.segment as A, count(distinct(FS.product_code)) as B
 from dim_product P, fact_sales_monthly FS
 where P.product_code = FS.product_code
 group by FS.fiscal_year, P.segment
 having FS.fiscal_year = "2020"),
 
 CTE2 AS
 (
 select P.segment as C, count(distinct(FS.product_code)) as D
 from dim_product P,  fact_sales_monthly FS
 where P.product_code = FS.product_code
 group by FS.fiscal_year, P.segment
 having FS.fiscal_year = "2021"
 )
 
 SELECT CTE1.A AS segment, CTE1.B AS product_count_2020, CTE2.D AS product_count_2021, (CTE2.D-CTE1.B) AS difference  
FROM CTE1, CTE2
WHERE CTE1.A = CTE2.C ;
````
**5. Get the products that have the highest and lowest manufacturing costs**
````sql
select F.product_code, P.product, F.manufacturing_cost
from fact_manufacturing_cost F join dim_product P
on F.product_code = P.product_code
where manufacturing_cost
in (
select max(manufacturing_cost) from fact_manufacturing_cost
union 
select min(manufacturing_cost) from fact_manufacturing_cost
)
order by manufacturing_cost desc;
````
**6. Generate a report which contains the top 5 customers who received an
average high pre_invoice_discount_pct for the fiscal year 2021 and in the
Indian market**
````sql
with tbl1 as
(select customer_code as A, avg(pre_invoice_discount_pct) as b from fact_pre_invoice_deductions
where fiscal_year='2021'
group by customer_code),

tbl2 as
(select customer_code as C, customer as D from dim_customer
where market ='India')
select tbl2.C as customer_code, tbl2.D AS customer, round (tbl1.b, 4) as average_discount_percentage
from tbl1 join tbl2 
on tbl1.A=tbl2.C
order by average_discount_percentage desc
limit 5
````
**7. Get the complete report of the Gross sales amount for the customer “Atliq
Exclusive” for each month**
````sql
select concat(monthname(FS.date),'(', year(FS.date),')')as 'Month', FS.fiscal_year,
round(sum(G.gross_price*FS.sold_quantity),2) as Gross_sales_Amount
from fact_sales_monthly FS join dim_customer C on FS.customer_code = C.customer_code
join fact_gross_price G on FS.product_code = G.product_code
where C.customer = 'Atliq Exclusive'
group by month, FS.fiscal_year
order by FS.fiscal_year;
````
**8. In which quarter of 2020, got the maximum total_sold_quantity?**
````sql
select case
		when month(date) in (9,10,11) then 'Q1'
        when month(date) in (12,1,2) then 'Q2'
        when month(date) in (3,4,5) then 'Q3'
			else 'Q4'
            end as quarters, 
            sum(sold_quantity) as total_quantity_sold
		from fact_sales_monthly
        where fiscal_year = 2020
        group by quarters
        order by total_quantity_sold desc;
````
**9. Which channel helped to bring more gross sales in the fiscal year 2021
and the percentage of contribution?**
````sql
WITH Output AS
(
SELECT C.channel,
       ROUND(SUM(G.gross_price*FS.sold_quantity/1000000), 2) AS Gross_sales_mln
FROM fact_sales_monthly FS JOIN dim_customer C ON FS.customer_code = C.customer_code
						   JOIN fact_gross_price G ON FS.product_code = G.product_code
WHERE FS.fiscal_year = 2021
GROUP BY channel
)
SELECT channel, CONCAT(Gross_sales_mln,' M') AS Gross_sales_mln , CONCAT(ROUND(Gross_sales_mln*100/total , 2), ' %') AS percentage
FROM
(
(SELECT SUM(Gross_sales_mln) AS total FROM Output) A,
(SELECT * FROM Output) B
)
ORDER BY percentage DESC;
````
**10. Get the Top 3 products in each division that have a high
total_sold_quantity in the fiscal_year 2021?**
````sql
with output1 as
(select P.division, FS.product_code, P.product, sum(FS.sold_quantity) as Total_sold_quantity
from dim_product P join fact_sales_monthly FS
on P.product_code =FS.product_code
WHERE FS.fiscal_year = 2021 
GROUP BY  FS.product_code, division, P.product
),
output2 as 
(
select division, product_code, product, Total_sold_quantity,
 rank() over(partition by division order by Total_sold_quantity desc) as 'Rank_Order'
from output1
)
select output1.division, output1.product_code, output1.product, output2.Total_sold_quantity, output2.Rank_Order
from output1 join output2
on output1.product_code = output2.product_code
where output2.Rank_Order in (1,2,3)
````
