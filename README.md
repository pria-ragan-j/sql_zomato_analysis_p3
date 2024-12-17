# sql_zomato_analysis_p3

## Project Overview

**Project Title**: Zomato analysis  
**Database**: Zomato_analysis_p3

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries.
## Objectives

1. **Set up a zomato sales database**: Create and populate a retail sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `Zomato_analysis_p3`.
- **Table Creation**: 
```sql
create database Zomato_analysis_p3;

create table customers(
customer_id	varchar(10) primary key,
customer_name varchar(50),	
reg_date date
);

create table restaurants(
restaurant_id varchar(10) primary key,
restaurant_name	varchar(50),
city varchar(20),
opening_hours varchar(50)
);

create table riders (
rider_id varchar(10) primary key,
rider_name varchar(30),
sign_up date
);

create table orders(

order_id varchar(10) primary key,
customer_id	varchar(10),
restaurant_id varchar(10),
order_item varchar(60),
order_date date,	
order_time time,
order_status varchar(55),
total_amount float,
foreign key (customer_id) references customers(customer_id),
foreign key (restaurant_id) references restaurants(restaurant_id)
);


create table deliveries (

delivery_id	varchar(10) primary key,
order_id varchar(10),
delivery_status	varchar(40),
delivery_time time,
rider_id varchar(10),
foreign key (order_id) references orders(order_id),
foreign key (rider_id) references riders(rider_id)
);

```

### 2. Data Exploration & Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Customer Count**: Find out how many unique customers are in the dataset.
- **Category Count**: Identify all unique product categories in the dataset.
- **Null Value Check**: Check for any null values in the dataset and delete records with missing data.

```sql
select count(*) from customers
where customer_name is null 
or 
reg_date is null ;

select count(*) from restaurants
where restaurant_name is null
or city is null
or opening_hours is null;

select count(*) from orders
where order_item is null 
or order_date is null
or order_time is null
or order_status is null
or total_amount is null;


select count(*) from riders
where rider_name is null
or sign_up is null;


select * from deliveries
where delivery_status is null
or delivery_time is null;
    
```

### 3. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

**Task 1.Write a query to find the top 5 most frequently ordered dishes by customer called "Arjun Mehta" in the last 1 year.Note Last one year means (let say today date 2024-09-28 then last 1 year = same period last year on wards)**:
```sql
select * from 
(select c.customer_name,o.order_item,count(o.order_item) as number_of_item,
dense_rank()over(order by count(o.order_item) desc) as Ranks
from customers c join orders o
using (customer_id)
where order_date >= '2023-09-28'
and order_date <= '2024-09-29' 
and c.customer_name = 'Arjun Mehta'
group by c.customer_name,o.order_item
order by number_of_item desc)
where Ranks <= 5 ;
```

**Task 2.Identify the time slots during which the most orders are placed, based on 2-hour intervals and 1-hour interval.**:
```sql
select 
case 
when extract (hour from order_time ) between 0 and 1 then '00:00-02:00'
when extract (hour from order_time ) between 2 and 3 then '02:00-04:00'
when extract (hour from order_time ) between 4 and 5 then '04:00-06:00'
when extract (hour from order_time ) between 6 and 7 then '06:00-08:00'
when extract (hour from order_time ) between 8 and 9 then '08:00-10:00'
when extract (hour from order_time ) between 10 and 11 then '10:00-12:00'
when extract (hour from order_time ) between 12 and 13 then '12:00-14:00'
when extract (hour from order_time ) between 14 and 15 then '14:00-16:00'
when extract (hour from order_time ) between 16 and 17 then '16:00-18:00'
when extract (hour from order_time ) between 18 and 19 then '18:00-20:00'
when extract (hour from order_time ) between 20 and 21 then '20:00-22:00'
when extract (hour from order_time ) between 22 and 23 then '22:00-00:00'
end as time_slot,
count(order_id) as order_count
from orders
group by time_slot
order by order_count desc;
```

**Task 3. .Find the average order value (AOV) per customer who has placed more than 750 orders.
   Return: customer_name, aov (average order value).in DESC of AOV.**:
```sql
select c.customer_name,count(*) as number_of_orders,
round(avg(o.total_amount)) as AOV
from customers c join orders o 
using(customer_id)
group by  c.customer_name
order by AOV desc;
```

**Task 4.List the customers who have spent more than 100K in total on food orders. Return: customer_name, customer_id.**:
```sql
select * from 
(select c.customer_name,c.customer_id,sum(total_amount) as Total_spending 
from customers c join orders o 
using(customer_id)
group by  c.customer_name,c.customer_id)t1
where Total_spending > 100000
order by Total_spending desc;
```

**Task 5.Write a query to find orders that were placed but not delivered. Return: restaurant_name, city, and the number of not delivered orders.**:
```sql
select r.restaurant_name,r.city,count(*) as cnt
from orders o left join restaurants r
using (restaurant_id) left join deliveries d
using (order_id)
where delivery_id is null
group by r.restaurant_name,r.city
order by count(*) desc;
```

**Task 6.Rank restaurants by their total revenue from the last year. Return: restaurant_name, city,total_revenue, and their rank within their city.**:
```sql
select r.restaurant_name,r.city,sum(total_amount) as total_revenue,
dense_rank()over(partition by r.city order by sum(total_amount)desc)
from orders as o left join restaurants as r
using(restaurant_id)
where extract(year from order_date) = 2023
group by r.restaurant_name,r.city,extract(year from order_date);
```

**Task 7.Identify the most popular dish in each city based on the number of orders.
   Return : city,most popular dish,total order, rnk.**:
```sql
select * from 
(select r.city,o.order_item,count(*) number_of_orders ,
dense_rank()over(partition by r.city order by count(*) desc) as ranks
from orders o join restaurants r
using (restaurant_id)
group by r.city,o.order_item)
where ranks = 1; 
```

**Task 8.Find customers who haven’t placed an order in 2024 but did in 2023.**:
```sql
with cust_cte as 
(select c.customer_id,c.customer_name,extract(year from o.order_date) as year
from customers c join orders o
using(customer_id))
select distinct customer_id,customer_name from cust_cte
where year = 2023
and customer_id not in (select customer_id from cust_cte where year = 2024);
```

**Task 9.Calculate and compare (restaurant are in both year) the order cancellation(not delivered)rate 
for each restaurant between the current year and the previous year.**:
```sql
with cte_2023 as

(select r.restaurant_id,r.restaurant_name,count(*) as cancel_2023 from orders o
left join restaurants r using (restaurant_id)
left join deliveries d using (order_id)
where delivery_status is null and extract(year from order_date) = 2023
group by 1,2
order by 3 desc)

, cte_2024 as 

(select r.restaurant_id,r.restaurant_name,count(*) as cancel_2024 from orders o
left join restaurants r using (restaurant_id)
left join deliveries d using (order_id)
where delivery_status is null and extract(year from order_date) = 2024
group by 1,2
order by 3 desc)

select cte_2023.restaurant_id,cte_2023.restaurant_name,cte_2023.cancel_2023,cte_2024.cancel_2024
from cte_2023 left join cte_2024
using(restaurant_id);
```


