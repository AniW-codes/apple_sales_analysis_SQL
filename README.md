
# ![Apple Logo](https://github.com/AniW-codes/apple_sales_analysis_SQL/blob/main/apple.jpeg) Apple Retail Sales SQL Project - Analyzing Millions of Sales Rows

**Apple Data Set**

- https://www.kaggle.com/datasets/aniruddhawarang/apple-data-set
 
## Project Overview

This project is designed to showcase advanced SQL querying techniques through the analysis of over 1 million rows of Apple retail sales data. The dataset includes information about products, stores, sales transactions, and warranty claims across various Apple retail locations globally. By tackling a variety of questions, from basic to complex, you'll demonstrate your ability to write sophisticated SQL queries that extract valuable insights from large datasets.


## Entity Relationship Diagram (ERD)

![ERD](https://github.com/AniW-codes/apple_sales_analysis_SQL/blob/main/erd.png)


---


## Database Schema

The project uses five main tables:

1. **stores**: Contains information about Apple retail stores.
   - `store_id`: Unique identifier for each store.
   - `store_name`: Name of the store.
   - `city`: City where the store is located.
   - `country`: Country of the store.

2. **category**: Holds product category information.
   - `category_id`: Unique identifier for each product category.
   - `category_name`: Name of the category.

3. **products**: Details about Apple products.
   - `product_id`: Unique identifier for each product.
   - `product_name`: Name of the product.
   - `category_id`: References the category table.
   - `launch_date`: Date when the product was launched.
   - `price`: Price of the product.

4. **sales**: Stores sales transactions.
   - `sale_id`: Unique identifier for each sale.
   - `sale_date`: Date of the sale.
   - `store_id`: References the store table.
   - `product_id`: References the product table.
   - `quantity`: Number of units sold.

5. **warranty**: Contains information about warranty claims.
   - `claim_id`: Unique identifier for each warranty claim.
   - `claim_date`: Date the claim was made.
   - `sale_id`: References the sales table.
   - `repair_status`: Status of the warranty claim (e.g., Paid Repaired, Warranty Void).

```sql

-- Apple Retails Millions Rows Sales Schemas


-- DROP TABLE command
DROP TABLE IF EXISTS warranty;
DROP TABLE IF EXISTS sales;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS category;
DROP TABLE IF EXISTS stores;

-- CREATE TABLE commands

CREATE TABLE stores(
store_id VARCHAR(5) PRIMARY KEY,
store_name	VARCHAR(30),
city	VARCHAR(25),
country VARCHAR(25)
);

DROP TABLE IF EXISTS category;
CREATE TABLE category
(category_id VARCHAR(10) PRIMARY KEY,
category_name VARCHAR(20)
);

CREATE TABLE products
(
product_id	VARCHAR(10) PRIMARY KEY,
product_name	VARCHAR(35),
category_id	VARCHAR(10),
launch_date	date,
price FLOAT,
CONSTRAINT fk_category FOREIGN KEY (category_id) REFERENCES category(category_id)
);

CREATE TABLE sales
(
sale_id	VARCHAR(15) PRIMARY KEY,
sale_date	DATE,
store_id	VARCHAR(10), -- this fk
product_id	VARCHAR(10), -- this fk
quantity INT,
CONSTRAINT fk_store FOREIGN KEY (store_id) REFERENCES stores(store_id),
CONSTRAINT fk_product FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE warranty
(
claim_id VARCHAR(10) PRIMARY KEY,	
claim_date	date,
sale_id	VARCHAR(15),
repair_status VARCHAR(15),
CONSTRAINT fk_orders FOREIGN KEY (sale_id) REFERENCES sales(sale_id)
);

-- Success Message
SELECT 'Schema created successful' as Success_Message;


```

## Improving Query Performance

```sql
select * from category;
select * from products;
select * from sales;
select * from stores;
select * from warranty;

---	Improving Query Performance	---

--et = 80 ms
--pt = 9 ms
EXPLAIN ANALYZE
select * from sales
where product_id = 'P-44';

--To optimise we created a index on product id column within sales table.
CREATE INDEX sales_p_id on sales(product_id);
--pt = 0.13 ms
--et = 3.45 ms

--pt = 0.094 ms
--et = 80.448 ms
EXPLAIN ANALYZE
select * from sales
where store_id = 'ST-33';


CREATE INDEX sales_s_id on sales(store_id);
--pt = 1.979 ms
--et = 1.49 ms

CREATE INDEX sales_sale_date on sales(sale_date);

```

###### Business Problems #######


1. Find the number of stores in each country.
```sql
select country, Count(*) 
from stores
group by country
order by 2 desc


```
2. Calculate the total number of units sold by each store.

```sql
SELECT 
	s.store_id,
	st.store_name,
	SUM(s.quantity) as total_unit_sold
FROM sales as s
JOIN
stores as st
ON st.store_id = s.store_id
GROUP BY 1, 2
ORDER BY 3 DESC

```
3. Identify how many sales occurred in December 2023.

```sql
SELECT 
	COUNT(sale_id) as total_sale 
FROM sales
WHERE TO_CHAR(sale_date, 'MM-YYYY') = '12-2023'

```
4. Determine how many stores have never had a warranty claim filed.
```sql
SELECT COUNT(*) FROM stores
WHERE store_id NOT IN (
						SELECT 
							DISTINCT store_id
						FROM sales as s
						RIGHT JOIN warranty as w
						ON s.sale_id = w.sale_id
						);

```

5. Calculate the percentage of warranty claims marked as "Warranty Void".
```sql

SELECT 
	ROUND
		(COUNT(claim_id)/
						(SELECT COUNT(*) FROM warranty)::numeric 
		* 100, 
	2)as warranty_void_percentage
FROM warranty
WHERE repair_status = 'Warranty Void'

```

6. Identify which store had the highest total units sold in the last year.
```sql
SELECT 
	s.store_id,
	st.store_name,
	SUM(s.quantity)
FROM sales as s
JOIN stores as st
ON s.store_id = st.store_id
WHERE sale_date >= (CURRENT_DATE - INTERVAL '1 year')
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 1



```


7. Count the number of unique products sold in the last year.
```sql
SELECT 
	COUNT(DISTINCT product_id)
FROM sales
WHERE sale_date >= (CURRENT_DATE - INTERVAL '1 year')


```
8. Find the average price of products in each category.

```sql

select 
	p.category_id,
	c.category_name,
	Avg(p.price) as average_price
	from products as p
join category as c
on p.category_id = c.category_id
group by 1,2
order by 3 desc


```
9. How many warranty claims were filed in 2020?
```sql

select 
COUNT(*)
from warranty
where TO_CHAR(claim_date,'YYYY') = '2020'

xxxxxxxxxxxxxx

select
	COUNT(*)
from warranty
where EXTRACT(YEAR from claim_date) = 2020


```

10. For each store, identify the best-selling day based on highest quantity sold.

```sql
select store_id, day_name, total_qty, rank
from
			(select 
				store_id,
				TO_CHAR(sale_date,'Day') as day_name,
				sum(quantity) as total_qty,
				RANK() OVER(Partition by store_id order by sum(quantity) desc) as rank
				
				from sales
			group by 1,2)
where rank = 1



```

11. Identify the least selling product in each country for each year based on total units sold.
```sql

select * from 
			(select 
				country,
				p.product_id,
				p.product_name,
				SUM(quantity) as total_qty,
				RANK() OVER(Partition by country order by SUM(quantity)) as Rank
				from sales as s
			join products as p
			on s.product_id = p.product_id
			join stores as st
			on st.store_id = s.store_id
			group by 1,2,3)
where Rank = 1



```

12. Calculate how many warranty claims were filed within 180 days of a product sale.
```sql
select 
	Count(*)
	from warranty as w
left join sales as s
on s.sale_id = w.sale_id
where claim_date - sale_date <= (180)



```


13. Determine how many warranty claims were filed for products launched in the last two years.

```sql
select 
	product_name,
	COUNT(w.claim_id),
	COUNT(s.sale_id)
	from warranty as w
right join sales as s
on s.sale_id  = w.sale_id
join products as p
on p.product_id = s.product_id
where p.launch_date >= CURRENT_DATE - INTERVAL '3 Years'
group by 1
having COUNT (w.claim_id) > 0


```

14. List the months in the last three years where sales exceeded 5,000 units in the USA.
```sql
select 
	TO_CHAR(sale_date,'MM-YYYY'),
	SUM(quantity) as total_qty
	from sales as s
join stores as st
on st.store_id = s.store_id
where 
	country = 'USA' 
	and
	sale_date >= CURRENT_DATE - INTERVAL '4 Year'
group by 1
having SUM(quantity) > 5000



```

15. Identify the product category with the most warranty claims filed in the last two years.
```sql
select 
	Category_name,
	COUNT(claim_id) as total_claims
	from warranty as w
left join sales as s
on s.sale_id = w.sale_id
join products as p
on p.product_id = s.product_id
join category as c
on c.category_id = p.category_id
where 
	claim_date >= CURRENT_DATE - Interval '2 Years'
group by 1
order by 2 desc

```

16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```sql
SELECT 
	country,
	total_unit_sold,
	total_claim,
	COALESCE(total_claim::numeric/total_unit_sold::numeric * 100, 0)
	as risk
FROM
(SELECT 
	st.country,
	SUM(s.quantity) as total_unit_sold,
	COUNT(w.claim_id) as total_claim
FROM sales as s
JOIN stores as st
ON s.store_id = st.store_id
LEFT JOIN 
warranty as w
ON w.sale_id = s.sale_id
GROUP BY 1) t1
ORDER BY 4 DESC

```
17. Analyze the year-by-year growth ratio for each store.
```sql
WITH yearly_sales
AS
(
	SELECT 
		s.store_id,
		st.store_name,
		EXTRACT(YEAR FROM sale_date) as year,
		SUM(s.quantity * p.price) as total_sale
	FROM sales as s
	JOIN
	products as p
	ON s.product_id = p.product_id
	JOIN stores as st
	ON st.store_id = s.store_id
	GROUP BY 1, 2, 3
	ORDER BY 2, 3 
),

growth_ratio as

	(select 
		store_name,
		year,
		total_sale as current_sale,
		LAG(total_sale,1) OVER(Partition by store_name order by year) as previous_year_sale
		from yearly_sales
	)

select
	store_name,
	year,
	current_sale,
	previous_year_sale,
	ROUND(
			(current_sale - previous_year_sale)::numeric/previous_year_sale ::numeric * 100
		,2) as growth_ratio
	from growth_ratio
where previous_year_sale is not null
	and
year <> 2024

```
18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
```sql
With correlation_claim_price 
as
(select
		claim_id,
		CASE
				When price < 500 then 'Less Expensive Product' 
				When price between 500 and 1000 then 'Mid Range Product'
				Else 'Expensive Product'
		END as price_brackets,
		price
from warranty as w
left join sales as s
on w.sale_id = s.sale_id
join products as p
on s.product_id = p.product_id
where claim_date >= CURRENT_DATE - Interval '5 Years'
)


Select 
	price_brackets,
	COUNT(claim_id) as claims
from correlation_claim_price
group by 1

```
19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```sql
With paid_repaired
as
	(select
		store_id,
		COUNT(claim_id) as total_paid_repairs
		from warranty as w
	left join sales as s
	on w.sale_id = s.sale_id
	where repair_status = 'Paid Repaired'
	group by 1
),

total_repaired
as
	(select
			store_id,
			COUNT(claim_id) as total_repairs
			from warranty as w
		left join sales as s
		on w.sale_id = s.sale_id
		group by 1
)

select
	pr.store_id,
	pr.total_paid_repairs,
	store_name,
	country,
	tr.total_repairs,
	ROUND((pr.total_paid_repairs::numeric/tr.total_repairs::numeric *100),2) as paid_repaired_percentage
	from paid_repaired as pr
join total_repaired as tr
on pr.store_id = tr.store_id
join stores as st
on st.store_id = pr.store_id
order by 6 desc

```
20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
```sql
With sale_year_month as
		(select 
			store_id,
			EXTRACT(YEAR from sale_date) as Year,
			EXTRACT(MONTH from sale_date) as Month,
			SUM(quantity*price) as sale_value
			from sales as s
		join products as p
		on s.product_id = p.product_id
		group by 1,2,3
		order by 1,2,3
		)
		
select
	store_id,
	Year,
	Month,
	sale_value,
	SUM(sale_value) OVER(PARTITION by store_id order by Year, Month) as running_total
	from sale_year_month

```

21. Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
```sql
select 
	sale_date,
	launch_date, 
	quantity,
	CASE
		When sale_date between launch_date and launch_date + Interval '6 Months' then '0-6 Months'
		When sale_date between launch_date + Interval '6 Months' and launch_date + Interval '12 Months' then '6-12 Months'
		When sale_date between launch_date + Interval '12 Months' and launch_date + Interval '18 Months' then '12-18 Months'
		When sale_date between launch_date + Interval '18 Months' and launch_date + Interval '24 Months' then '18-24 Months'
		When sale_date between launch_date + Interval '24 Months' and launch_date + Interval '30 Months' then '24-30 Months'
		else 'Greater than 30 Months'
	END as Product_Bucket
from sales as s
join products as p
on s.product_id = p.product_id

xxxxxxxxxxxx

select 
	product_name,
	CASE
		When sale_date between launch_date and launch_date + Interval '6 Months' then '0-6 Months'
		When sale_date between launch_date + Interval '6 Months' and launch_date + Interval '12 Months' then '6-12 Months'
		When sale_date between launch_date + Interval '12 Months' and launch_date + Interval '18 Months' then '12-18 Months'
		When sale_date between launch_date + Interval '18 Months' and launch_date + Interval '24 Months' then '18-24 Months'
		When sale_date between launch_date + Interval '24 Months' and launch_date + Interval '30 Months' then '24-30 Months'
		else 'Greater than 30 Months'
	END as Product_Bucket,
	SUM(quantity) as total_quantity_sold
from sales as s
join products as p
on s.product_id = p.product_id
group by 1,2
order by 1,2


```
## Project Focus

This project primarily focuses on developing and showcasing the following SQL skills:

1.**Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.

2.**Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.

3.**Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.

4.**Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.

5.**Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.


## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans multiple years (2019-2023), allowing for long-term trend analysis.
- **Geographical Coverage**: Sales data from Apple stores across various countries.

**Aniruddha Warang**  
Data Analyst  
Connect with me Professionally:


📧 Email: waranganya@gmail.com  

🔗 [LinkedIn](https://www.linkedin.com/in/aniruddhawarang/)


---
