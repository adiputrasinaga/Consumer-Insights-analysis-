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
<br/>
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


