# khadok_sql_project
SQL queries and analysis on a food delivery database. Includes customer behavior, order trends, restaurant performance, rider efficiency, revenue analysis, and growth metrics using advanced SQL techniques like CTEs, window functions, and aggregation.

## Project Structure
- **Database Setup:** Creation of the `khadok_db` database and the required tables.
- **Data Import:** Inserting sample data into the tables.
- **Data Cleaning:** Handling null values and ensuring data integrity.
- **Business Problems:** Solving 20 specific business problems using SQL queries.

## Database Setup
```sql
CREATE DATABASE khadok_db;
```
### 1. Dropping Existing Table
```sql
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS restaurants;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS riders;
DROP TABLE IF EXISTS deliveries;
```
--**Creating Table**
```sql
create table customers(
customer_id int primary key,
customer_name varchar(30),
reg_date date
);

create table restaurants(
restaurant_id int primary key,
restaurant_name varchar (50),
city	varchar (20),
opening_hours varchar (55)
);

create table orders(
order_id int primary key,	
customer_id	int ,
restaurant_id int ,
order_item varchar ( 50),
order_date date,	
order_time time ,
order_status varchar (20),
total_amount NUMERIC(10,2)
);
-- adding fk constraint
alter table orders
Add constraint fk_customers
foreign key(customer_id)
references customers(customer_id);

alter table orders
add constraint fk_restaurant
foreign key(restaurant_id)
references restaurants(restaurant_id);

create table riders(
rider_id int primary key,
rider_name varchar ( 40),
sign_up date
);

CREATE TABLE deliveries (
delivery_id INT PRIMARY KEY,
order_id INT,        
delivery_status VARCHAR(20),
delivery_time TIME,
rider_id INT,

CONSTRAINT fk_orders
FOREIGN KEY (order_id)
REFERENCES orders(order_id),

CONSTRAINT fk_riders
FOREIGN KEY (rider_id)
REFERENCES riders(rider_id)
);
  ```
### Data Import
### Data Cleaning and Handling Null Values
- **Before performing analysis, I ensured that the data was clean and free from null values where necessary. For instance:**
``` sql
UPDATE orders
SET total_amount = COALESCE(total_amount, 0); 
```
### Business Problems solved
**1.Find the top 5 most frequently ordered dishes by customer called "Ayaan Rahman" in the last 1 year 2 month.**
``` sql
select * from
(select c.customer_id,c.customer_name,o.order_item as dishes,count(*) as total_order,
DENSE_RANK() OVER (order by count(*) desc) as rank
from customers as c
join orders as o
on c.customer_id= o.customer_id
where c.customer_name='Ayaan Rahman'
and o.order_date >= CURRENT_DATE - INTERVAL '1 year 2 month'
group by 1,2,3 
order by total_order desc) as t1
where rank<=5;
```
## 2.Popular Time Slots based on 2-hour intervals.
**Approach 1**
``` sql
SELECT
FLOOR (extract (hour from order_time)/2)*2 as start_time,
FLOOR (extract (hour from order_time)/2)*2 +2 as end_time,
count(*)as total_order
from orders
group by 1,2
order by 3 desc;
```
**Approach 2**
``` sql
SELECT time_slot,order_count,
DENSE_RANK() OVER (ORDER BY order_count DESC) AS r(
SELECT CASE
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 0 AND 1 THEN '00:00-02:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 2 AND 3 THEN '02:00-04:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 4 AND 5 THEN '04:00-06:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 6 AND 7 THEN '06:00-08:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 8 AND 9 THEN '08:00-10:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 10 AND 11 THEN '10:00-12:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 12 AND 13 THEN '12:00-14:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 14 AND 15 THEN '14:00-16:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 16 AND 17 THEN '16:00-18:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 18 AND 19 THEN '18:00-20:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 20 AND 21 THEN '20:00-22:00'
WHEN EXTRACT(HOUR FROM order_time) BETWEEN 22 AND 23 THEN '22:00-00:00'
END AS time_slot,
COUNT(order_item) AS order_count
FROM orders
GROUP BY time_slot
)t2
ORDER BY order_count DESC;
```

**Q3. Find average order value of customers who placed more than 60 orders.**
``` sql
SELECT c.customer_name,COUNT(*) AS total_order,
AVG(o.total_amount) AS avg_order_amount
FROM customers AS c
JOIN orders AS o
ON c.customer_id = o.customer_id
GROUP BY c.customer_name
HAVING COUNT(*) >= 60
ORDER BY avg_order_amount DESC;
```

**Q4. Find customers whose total spending is more than 25,000.**
``` sql
SELECT c.customer_name,COUNT(o.order_id) AS total_order,SUM(*) AS total_order_amount
FROM customers AS c
JOIN orders AS o
ON c.customer_id = o.customer_id
GROUP BY c.customer_name
HAVING sum(o.total_amount)>=25000
ORDER BY total_order_amount DESC;
```
**Q5. Find restaurant-wise number of not delivered orders with city.**
``` sql
SELECT r.restaurant_name,r.city,COUNT(*) AS not_delivered
FROM orders AS o
JOIN restaurants AS r
ON o.restaurant_id = r.restaurant_id
JOIN deliveries AS d
ON o.order_id = d.order_id
WHERE d.delivery_status = 'Not Delivered' 
GROUP BY 1,2
ORDER BY r.city DESC,not_delivered DESC;
```
**Q6. Rank restaurants by total revenue (last year) within each city.**
``` sql
select city,restaurant_name,total_revenue,ranking from
(select r.restaurant_name,r.city,sum(o.total_amount) as total_revenue,
DENSE_RANK() over(partition by r.city order by sum(o.total_amount) desc) as ranking
from orders as o
join restaurants as r
on r.restaurant_id= o.restaurant_id
where o.order_date>=current_date-INTERVAL'1year' 
group by 1,2
order by city desc,total_revenue desc, ranking desc) as t6
where ranking=1;
```
**Q7. Find the most popular dish in each city.**
``` sql
with popular_table AS
(select r.city,o.order_item,count(*)AS total_order,
row_number () OVER (PARTITION BY r.city ORDER BY COUNT(*) DESC) AS rank_in_city
from orders as o
join restaurants as r
on o.restaurant_id= r.restaurant_id
group by 1,2
order by city,total_order desc)

select * from popular_table
where rank_in_city=1;
```
**Q8. Find customers who ordered in Jan–Mar 2025 but not in April 1–5, 2025.**
``` sql
SELECT DISTINCT c.customer_name
FROM customers c
JOIN orders o
ON c.customer_id = o.customer_id
WHERE o.order_date BETWEEN '2025-01-01' AND '2025-03-31'
AND c.customer_id NOT IN(
SELECT customer_id
FROM orders
WHERE order_date BETWEEN '2025-04-01' AND '2025-04-5'
);
```
**Q9. Compare restaurant-wise cancellation rate for January and February.**
``` sql
WITH monthly_orders AS 
(select r.restaurant_name, DATE_TRUNC('month', o.order_date) AS order_month,
COUNT(*) AS total_orders,sum(case when o.order_status='Cancelled' then 1 else 0 end) as cancelled_orders
FROM orders o
join restaurants r
ON o.restaurant_id = r.restaurant_id 
WHERE o.order_date BETWEEN '2025-01-01' AND '2025-02-28'
group by 1,2)
select restaurant_name,
-- January
MAX(CASE WHEN order_month='2025-01-01' THEN
round(cancelled_orders * 100.0 / total_orders,2) END) AS jan_cancellation_rate,
-- February
MAX(CASE WHEN order_month = '2025-02-01' THEN
round (cancelled_orders * 100.0 / total_orders,2) END) AS feb_cancellation_rate
FROM monthly_orders
GROUP BY restaurant_name
ORDER BY restaurant_name;
```
**Q10. Find each rider’s average delivery time.**
``` sql
WITH riders_average_delivery_time AS 

(SELECT r.rider_name,o.order_time,d.delivery_time,
EXTRACT(EPOCH FROM (d.delivery_time - o.order_time +
CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 day' ELSE
INTERVAL '0 day' END))/60 as time_difference_in_min
from deliveries as d
join orders as o
on d.order_id= o.order_id
join riders as r
on r.rider_id=d.rider_id
where delivery_status='Delivered')

select rider_name,ROUND(avg(time_difference_in_min),2)AS avg_delivery_time_min,
DENSE_RANK()OVER(ORDER BY ROUND(avg(time_difference_in_min),2) ASC) AS RANKING
from riders_average_delivery_time 
group by rider_name;
```
**Q11. Calculate monthly delivered-order growth ratio for each restaurant.**
``` sql
WITH growth_ratio AS
(SELECT
    o.restaurant_id,
    TO_CHAR(o.order_date, 'mm-yy') as month,
    COUNT(o.order_id) as cr_month_orders,
    LAG(COUNT(o.order_id), 1) OVER(PARTITION BY o.restaurant_id ORDER BY TO_CHAR(o.order_date, 'mm-yy')) as prev_month_orders
FROM orders as o
JOIN
deliveries as d
ON o.order_id = d.order_id
WHERE d.delivery_status = 'Delivered'
GROUP BY 1, 2
ORDER BY 1, 2)

SELECT
    restaurant_id,
    month,
    prev_month_orders,
    cr_month_orders,
    round((cr_month_orders::numeric-prev_month_orders::numeric)/prev_month_orders::numeric * 100,2)
    as growth_ratio
FROM growth_ratio 
```
**Q12. Segment customers into Gold and Silver and show total orders and revenue per segment.**
``` sql
WITH Customer_Segmentation AS
(select c.customer_name,count(order_id) as total_order,avg(o.total_amount) as average_order_value
from orders as o
join customers as c
on o.customer_id=c.customer_id
where order_status='Completed'
group by 1)
select customer_name,total_order,ROUND(average_order_value, 2) AS average_order_value,
CASE WHEN ROUND(average_order_value, 2)>=ROUND(AVG(average_order_value) OVER (),2) THEN 'Gold'ELSE 'Silver'
    END AS customer_segment
FROM customer_segmentation
; 
```
**Q13. Calculate each rider’s monthly earnings (8% of order amount).**
``` sql
SELECT r.rider_name,TO_CHAR(o.order_date,'YYYY-MM') AS Month,
SUM(o.total_amount) AS revenue,SUM(o.total_amount) * 0.08 AS monthly_earning
FROM deliveries d
JOIN orders o
ON o.order_id = d.order_id
JOIN riders r
ON r.rider_id = d.rider_id
WHERE d.delivery_status = 'Delivered'
GROUP BY r.rider_name,TO_CHAR(o.order_date, 'YYYY-MM')
ORDER BY r.rider_name,Month;
```
**Q14. Count 5-star, 4-star and 3-star ratings for each rider based on delivery time.**
``` sql
SELECT rider_id,rider_rating,count(rider_rating)as total_rating
from
(WITH Rider_Ratings as
(SELECT d.rider_id, o.order_id,o.order_time,d.delivery_time,
EXTRACT(EPOCH FROM (d.delivery_time - o.order_time +CASE
WHEN d.delivery_time < o.order_time THEN INTERVAL '1 day' ELSE INTERVAL '0 day' END)) / 60 AS delivery_took_time
FROM orders AS o
JOIN deliveries AS d
ON o.order_id = d.order_id
WHERE delivery_status = 'Delivered')
select rider_id,order_id,delivery_time,delivery_took_time,
CASE WHEN delivery_took_time<= 80 THEN '5_star'
WHEN delivery_took_time BETWEEN 81 AND 100 THEN '4_star'ELSE '3_star' END AS rider_rating 
from Rider_Ratings
order by rider_rating desc)--......
t1
group by 1,2
order by 1,2 desc

```
**Q15. Find the peak order day of the week for each restaurant.**
``` sql
SELECT * FROM (
SELECT r.restaurant_name,TO_CHAR(o.order_date, 'Day'), COUNT(o.order_id) AS total,
DENSE_RANK() OVER (PARTITION BY r.restaurant_name ORDER BY COUNT(o.order_id) DESC) AS rank
FROM orders AS o
JOIN restaurants AS r
ON o.restaurant_id = r.restaurant_id
GROUP BY 1, 2
ORDER BY 1, 3 DESC
)AS t1
WHERE rank = 1;
```
**Q16. Calculate customer lifetime value (total revenue per customer).**
``` sql
select c.customer_id,c.customer_name,sum(o.total_amount) as CLV 
from customers as c
join orders as o
on c.customer_id=o.customer_id
WHERE o.order_status IN ('Completed', 'Pending')
group by 1,2
order by 3 desc;
```
**Q17. Compare monthly sales with previous month and calculate growth rate.**
``` sql
WITH monthly_sales AS (
SELECT
EXTRACT(YEAR FROM order_date) AS year,
EXTRACT(MONTH FROM order_date) AS month,
SUM(total_amount) AS total_sales
FROM orders
WHERE order_status IN ('Completed', 'Pending')
GROUP BY 1, 2)
SELECT year,month,total_sales,LAG(total_sales) OVER (ORDER BY year, month) AS prev_month_sales,
ROUND((total_sales - LAG(total_sales) OVER (ORDER BY year, month))
/ LAG(total_sales) OVER (ORDER BY year, month) * 100,2) AS mon_growth_pct

FROM monthly_sales
ORDER BY year, month;
```
**Q18. Find fastest and slowest riders based on average delivery time.**
``` sql
with Rider_Efficiency as
(SELECT d.rider_id, o.order_id,o.order_time,d.delivery_time,
EXTRACT(EPOCH FROM (d.delivery_time - o.order_time +CASE
WHEN d.delivery_time < o.order_time THEN INTERVAL '1 day' ELSE INTERVAL '0 day' END)) / 60 AS delivery_took_time
FROM orders AS o
JOIN deliveries AS d
ON o.order_id = d.order_id
WHERE delivery_status = 'Delivered'),
-- we are addding another cte
rider_time as
(select rider_id,avg(delivery_took_time)as avg_delivery
from Rider_Efficiency
group by rider_id
order by 1)
select min(avg_delivery)AS fastest_rider_avg_time,max(avg_delivery)AS slowest_rider_avg_time
from rider_time;
```
**Q19. Track monthly popularity of order items and identify demand trends.**
``` sql
select order_item,to_char(order_date,'month') as month,count(*)as total_order
from orders
where order_status in ('Completed', 'Pending')
group by 1,2
order by 1,3 desc;
```
**Q20. Rank each city by total revenue for the year.**
``` sql
SELECT
    r.city,
    SUM(total_amount) AS total_revenue,
    RANK() OVER (ORDER BY SUM(total_amount) DESC) AS city_rank
FROM orders AS o
JOIN restaurants AS r
ON o.restaurant_id = r.restaurant_id
GROUP BY 1;
```
