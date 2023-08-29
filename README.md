**Consumer Insights Analysis**
<br/>

**Provide the list of markets in which customer "Atliq Exclusive" operates its
business in the APAC region**
````sql
select market from dim_customer
where customer = 'Atliq Exclusive' and region = 'APAC'
group by market
order by market;
````
**What is the percentage of unique product increase in 2021 vs. 2020?**
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
**Provide a report with all the unique product counts for each segment and
sort them in descending order of product counts**
````sql
SELECT segment, 
COUNT(DISTINCT(product_code)) AS product_count FROM dim_product
GROUP BY segment
ORDER BY product_count DESC;
````
**Which segment had the most increase in unique products in
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
**Get the products that have the highest and lowest manufacturing costs**
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


