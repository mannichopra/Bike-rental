# Bike-rental SQL Project

## Project Overview

**Project Title**: Bike Rental Analysis  
**Database**: `Bike Rentals`

## Objectives

1. **Set up a bike rentals database**: Create and populate a bike rentals database with the provided data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the data.

  ## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `Bike Rentals`.
- **Table Creation**: A table named `customer` is created to store the sales data. The table structure includes columns for id, name, email
- **Table Creation**: A table named `bike` is created to store the sales data. The table structure includes columns for id, model, category, price_per_hour, price_per_day, status
- **Table Creation**: A table named `rental` is created to store the sales data. The table structure includes columns for id, customer_id, bike_id, start_timestamp, duration, total_paid
- **Table Creation**: A table named `membership_type` is created to store the sales data. The table structure includes columns for id, name, description, price
- **Table Creation**: A table named `membership` is created to store the sales data. The table structure includes columns for id, membership_type_id,customer_id, start_date,end_date, total_paid


```sql
CREATE DATABASE Bike Rentals;

drop table if exists customer;
create table customer
(
	id	int primary key,
	name	varchar(30),
	email	varchar(50)
);

drop table if exists bike;
create table bike
(
	id			int primary key,
	model			varchar(50),
	category		varchar(50),
	price_per_hour		decimal,
	price_per_day		decimal,
	status			varchar(20)
);


drop table if exists rental;
create table rental
(
	id			int primary key,
	customer_id		int references customer(id),
	bike_id			int references bike(id),
	start_timestamp		timestamp,
	duration		int,
	total_paid		decimal
);


drop table if exists membership_type;
create table membership_type
(
	id		int primary key,
	name		varchar(50),
	description	varchar(500),
	price		decimal
);


drop table if exists membership;
create table membership
(
	id			int primary key,
	membership_type_id	int references membership_type(id),
	customer_id		int references customer(id),
	start_date		date,
	end_date		date,
	total_paid		decimal
);
```
 ### 2. Data Exploration & Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Customer Count**: Find out how many unique customers are in the dataset.
- **Category Count**: Identify all unique product categories in the dataset.
- **Null Value Check**: Check for any null values in the dataset and delete records with missing data.

```sql
SELECT COUNT(*) FROM customer;
SELECT COUNT(DISTINCT customer_id) FROM rental;
SELECT DISTINCT category FROM bike;

SELECT * FROM customer
WHERE 
    id IS NULL OR name IS NULL OR email IS NULL

SELECT * FROM bike
WHERE 
    id IS NULL OR model IS NULL OR category IS NULL
    OR price_per_hour IS NULL OR price_per_day IS NULL
    OR status IS NULL

SELECT * FROM rental
WHERE 
    id IS NULL OR customer_id IS NULL OR bike_id IS NULL
    OR start_timestamp IS NULL OR duration IS NULL
    OR total_paid IS NULL

SELECT * FROM membership
WHERE 
    id IS NULL OR membership_type_id IS NULL OR customer_id IS NULL
    OR start_date IS NULL OR end_date IS NULL
    OR total_paid IS NULL

SELECT * FROM membership_type
WHERE 
    id IS NULL OR name IS NULL OR description IS NULL
    OR price IS NULL

DELETE FROM customer
WHERE 
    id IS NULL OR name IS NULL OR email IS NULL

DELETE FROM bike
WHERE 
    id IS NULL OR model IS NULL OR category IS NULL
    OR price_per_hour IS NULL OR price_per_day IS NULL
    OR status IS NULL

DELETE FROM rental
WHERE 
    id IS NULL OR customer_id IS NULL OR bike_id IS NULL
    OR start_timestamp IS NULL OR duration IS NULL
    OR total_paid IS NULL

DELETE FROM membership
WHERE 
    id IS NULL OR membership_type_id IS NULL OR customer_id IS NULL
    OR start_date IS NULL OR end_date IS NULL
    OR total_paid IS NULL

DELETE FROM membership_type
WHERE 
    id IS NULL OR name IS NULL OR description IS NULL
    OR price IS NULL
```

### 3. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

1. **Emily would like to know how many bikes the shop owns by category. Can you get this for her? 
Display the category name and the number of bikes the shop owns in each category (call this column number_of_bikes ).
Show only the categories where the number of bikes is greater than 2 .**:
```sql
select category, count(1) as number_of_bikes 
from bike
group by category
having count(1) > 2;
```

2. **Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 4 in the month of Nov-2022**:
```sql
SELECT 
  *
FROM retail_sales
WHERE 
    category = 'Clothing'
    AND 
    TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
    AND
    quantity >= 4
```

3. **Write a SQL query to calculate the total sales (total_sale) for each category.**:
```sql
SELECT 
    category,
    SUM(total_sale) as net_sale,
    COUNT(*) as total_orders
FROM retail_sales
GROUP BY 1
```

4. **Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category.**:
```sql
SELECT
    ROUND(AVG(age), 2) as avg_age
FROM retail_sales
WHERE category = 'Beauty'
```

5. **Write a SQL query to find all transactions where the total_sale is greater than 1000.**:
```sql
SELECT * FROM retail_sales
WHERE total_sale > 1000
```

6. **Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.**:
```sql
SELECT 
    category,
    gender,
    COUNT(*) as total_trans
FROM retail_sales
GROUP 
    BY 
    category,
    gender
ORDER BY 1
```

7. **Write a SQL query to calculate the average sale for each month. Find out best selling month in each year**:
```sql
SELECT 
       year,
       month,
    avg_sale
FROM 
(    
SELECT 
    EXTRACT(YEAR FROM sale_date) as year,
    EXTRACT(MONTH FROM sale_date) as month,
    AVG(total_sale) as avg_sale,
    RANK() OVER(PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY AVG(total_sale) DESC) as rank
FROM retail_sales
GROUP BY 1, 2
) as t1
WHERE rank = 1
```

8. **Emily wants to segment customers based on the number of rentals and
see the count of customers in each segment. Categorize customers based on their rental history as follows:
Customers who have had more than 10 rentals are categorized as 'more than 10' .
Customers who have had 5 to 10 rentals (inclusive) are categorized as 'between 5 and 10' .
Customers who have had fewer than 5 rentals should be categorized as 'fewer than 5' .
Calculate the number of customers in each category.
Display two columns: rental_count_category (the rental count category) and customer_count (the number of customers in each category).**:
```sql
with cte as 
    (select customer_id, count(1)
    , case when count(1) > 10 then 'more than 10' 
           when count(1) between 5 and 10 then 'between 5 and 10'
           else 'fewer than 5'
      end as category
    from rental
    GROUP by customer_id)
select category as rental_count_category
, count(*) as customer_count
from cte 
group by category
order by customer_count;
```

## Findings

- **Customer Demographics**: The dataset includes customers from various age groups, with sales distributed across different categories such as Clothing and Beauty.
- **High-Value Transactions**: Several transactions had a total sale amount greater than 1000, indicating premium purchases.
- **Sales Trends**: Monthly analysis shows variations in sales, helping identify peak seasons.
- **Customer Insights**: The analysis identifies the top-spending customers and the most popular product categories.

## Reports

- **Sales Summary**: A detailed report summarizing total sales, customer demographics, and category performance.
- **Trend Analysis**: Insights into sales trends across different months and shifts.
- **Customer Insights**: Reports on top customers and unique customer counts per category.

## Conclusion

This project serves as a comprehensive introduction to SQL for data analysts, covering database setup, data cleaning, exploratory data analysis, and business-driven SQL queries. The findings from this project can help drive business decisions by understanding sales patterns, customer behavior, and product performance.
