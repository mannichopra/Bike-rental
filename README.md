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

2. **Emily needs a list of customer names with the total number of memberships purchased by each. For each customer, display the customer's name and the count of memberships purchased (call this column membership_count ). Sort the results by membership_count , starting with the customer who has purchased the highest number of memberships.
Keep in mind that some customers may not have purchased any memberships yet. In such a situation, display 0 for the membership_count .**:
```sql
select c.name, count(m.id) as membership_count 
from membership m
right join customer c on c.id=m.customer_id
group by c.name
order by count(1) desc;
```

3. **Emily is working on a special offer for the winter months. Can you help her prepare a list of new rental prices? For each bike, display its ID, category,
old price per hour (call this column old_price_per_hour ), discounted price per hour (call it new_price_per_hour ), old price per day (call it old_price_per_day ),
and discounted price per day (call it new_price_per_day ).
Electric bikes should have a 10% discount for hourly rentals and a 20% discount for daily rentals. Mountain bikes should have a 20% discount for hourly rentals and a
50% discount for daily rentals. All other bikes should have a 50% discount for all types of rentals. Round the new prices to 2 decimal digits.**:
```sql
  select id, category,
  price_per_hour as old_price_per_hour,
  case when category = 'electric' then round(price_per_hour - (price_per_hour*0.1) ,2)
  when category = 'mountain bike' then round(price_per_hour - (price_per_hour*0.2) ,2)
  else round(price_per_hour - (price_per_hour*0.5) ,2)
  end as new_price_per_hour,
  price_per_day as old_price_per_day,
  case when category = 'electric' then round(price_per_day - (price_per_day*0.2) ,2)
  when category = 'mountain bike' then round(price_per_day - (price_per_day*0.5) ,2)
  else round(price_per_day - (price_per_day*0.5) ,2)
  end as new_price_per_day
  from bike;
```

4. **Emily is looking for counts of the rented bikes and of the available bikes in each category. Display the number of available bikes (call this column  available_bikes_count ) and
the number of rented bikes (call this column rented_bikes_count ) by bike category..**:
```sql
select category,
count(case when status ='available' then 1 end) as available_bikes_count,
count(case when status ='rented' then 1 end) as rented_bikes_count
from bike
group by category;
```

5. **Emily is preparing a sales report. She needs to know the total reven from rentals by month, the total by year, and the all-time across all the years.
Display the total revenue from rentals for each month, the total for each year, and the total across all the years. Do not take memberships into account.
There should be 3 columns: year , month , and revenue . Sort the results chronologically.
Display the year total after all the month totals for the corresponding year. Show the all-time total as the last row.**:
```sql
select extract(year from start_timestamp) as year,
extract(month from start_timestamp) as month,
sum(total_paid) as revenue
from rental
group by extract(year from start_timestamp), extract(month from start_timestamp)
union all
select extract(year from start_timestamp) as year,
null as month, sum(total_paid) as revenue
from rental
group by extract(year from start_timestamp)
union all
select null as year, null as month, sum(total_paid) as revenue
from rental
order by year, month;
```

6. **Emily has asked you to get the total revenue from memberships for each combination of year, month, and membership type.
Display the year, the month, the name of the membership type (call this column membership_type_name ), and the total revenue (call this column total_revenue ) for every combination of year, month, and membership type. Sort the results by year, month, and name of membership type.**:
```sql
select extract(year from start_date) as year
, extract(month from start_date) as month
, mt.name as membership_type_name
, sum(total_paid) as total_revenue
from membership m
join membership_type mt on m.membership_type_id = mt.id
group by year, month, mt.name
order by year, month, mt.name

```

7. **Next, Emily would like data about memberships purchased in 2023, with subtotals and grand totals for all the different combinations of membership types and months.
Display the total revenue from memberships purchased in 2023 for each combination of month and membership type. Generate subtotals and grand totals for all possible combinations.
There should be 3 columns: membership_type_name , month , and total_revenue . Sort the results by membership type name alphabetically and then chronologically by month**:
```sql
select mt.name as membership_type_name
, extract(month from start_date) as month
, sum(total_paid) as total_revenue
from membership m
join membership_type mt on m.membership_type_id = mt.id
where extract(year from start_date) = 2023
group by CUBE(membership_type_name, month)
order by membership_type_name, month;
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

- **Customer Demographics**: The dataset includes customers, with sales distributed across different categories such as mountain bike, road bike, hybrid and electric.
- **High-Value Transactions**: Several transactions had a total amount greater than 400, indicating premium membership purchases.
- **Sales Trends**: Monthly analysis shows variations in sales, helping identify peak seasons.
- **Customer Insights**: The analysis identifies the top-spending customers and the most popular product categories.

## Reports

- **Sales Summary**: A detailed report summarizing total sales, customer demographics, and category performance.
- **Trend Analysis**: Insights into sales trends across different model and category.
- **Customer Insights**: Reports on top customers and unique customer counts per category.

## Conclusion

This project covers database setup, data cleaning, exploratory data analysis, and business-driven SQL queries. The findings from this project can help drive business decisions by understanding sales patterns, customer behavior, and product performance.
