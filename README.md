# Khadok Food Delivery SQL Data Analysis Project
![image alt](https://github.com/mojahid1252/khadok_sql_project/blob/main/Screenshot%202026-02-09%20192256.png?raw=true)



##  Project Overview
**Khadok** is a food delivery platform (inspired by Foodpanda/Pathao Food) 
built to demonstrate real-world SQL analytics skills.

This project analyzes **end-to-end food delivery operations** from customer ordering 
behavior to restaurant performance, rider efficiency, and revenue growth using 
**advanced SQL techniques  like CTEs, window functions, and aggregation.** in PostgreSQL.

## Project Structure
- **Database Setup:** Creation of the `khadok_db` database and the required tables.
- **Data Import:** Inserting sample data into the tables.
- **Data Cleaning:** Handling null values and ensuring data integrity.
- **Business Problems:** Solving 20 specific business problems using SQL queries.
  
 💡 **Goal:** Extract actionable business insights that a data-driven food delivery 
> company would use to improve operations, increase revenue, and reduce cancellations.

## 📊 Dataset Overview
```
The project uses 5 interconnected tables simulating a complete food delivery ecosystem:

Table: customers | Records: 34 | Description: Customer profiles including names and registration dates throughout 2024
Table: restaurants | Records: 50 | Description: Detailed restaurant profiles including city locations across Bangladesh and operating hours
Table: orders | Records: 2,000 | Description: Transactional records tracking order items, dates, times, completion status, and total amounts
Table: riders | Records: 34 | Description: Profiles of delivery partners including their sign-up dates in 2024
Table: deliveries | Records: 2,000 | Description: Logistical data tracking delivery status (Delivered, In Transit, Not Delivered) and completion times for every order
```






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
-- Handle null values in total_amount
UPDATE orders
SET total_amount = COALESCE(total_amount, 0);

-- Verify no orphan records exist
SELECT * FROM orders 
WHERE customer_id NOT IN (SELECT customer_id FROM customers);

-- Check for duplicate order IDs
SELECT order_id, COUNT(*) 
FROM orders 
GROUP BY order_id 
HAVING COUNT(*) > 1;
``` 

### Business Problems solved
**1.Find the top 5 most frequently ordered dishes by customer called "Ayaan Rahman" in the last 1 year 2 month.**
```
Objective: Identify the favorite dishes of a specific customer for
personalized marketing.
```
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
```
Insight: Helps marketing send personalized offers to high-value customers based on their strongest food preferences.
```
## 2.Popular Time Slots based on 2-hour intervals.
```
Objective: Identify the most popular ordering time slots using 2-hour intervals to understand demand peaks.
Insight: Supports staffing, rider allocation, and time-based promotions (optimize peak operations and boost off-peak demand).
```
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
```
Insight: Identifies peak and off-peak hours to optimize rider staffing and run time-based promotions.
```
**Q3. Find average order value of customers who placed more than 60 orders.**
```
Objective: Calculate the average order value for customers who placed more than 60 orders to evaluate high-frequency customer value.
```
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
```
Insight: Shows spending behavior of high-frequency customers, helping design loyalty and upsell strategies.
```
**Q4. Find customers whose total spending is more than 25,000.**
```
Objective: Identify customers whose total spending exceeds 25,000 to find premium/high-value customers.
 ```
``` sql
SELECT c.customer_name,COUNT(o.order_id) AS total_order,SUM(*) AS total_order_amount
FROM customers AS c
JOIN orders AS o
ON c.customer_id = o.customer_id
GROUP BY c.customer_name
HAVING sum(o.total_amount)>=25000
ORDER BY total_order_amount DESC;
```
```
Insight: Identifies premium customers for VIP retention, personalized rewards, and churn prevention.
```
**Q5. Find restaurant-wise number of not delivered orders with city.**
```
Objective: Find restaurant-wise count of “Not Delivered” orders along with city to spot fulfillment issues.
```
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
```
Insight: Highlights restaurants/cities with delivery failures so operations can fix issues and improve reliability.
```
**Q6. Rank restaurants by total revenue (last year) within each city.**
```
Objective: Rank restaurants by total revenue (last year) within each city to identify top performers. 
```
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
```
Insight: Reveals top revenue-generating restaurants per city for partnerships, featuring, and targeted campaigns.
```
**Q7. Find the most popular dish in each city.**
```
Objective: Identify the most popular dish in each city to understand local customer preferences.
```
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
```
Insight: Captures city-wise taste preferences to support geo-targeted marketing and menu planning.
```
**Q8. Find customers who ordered in Jan–Mar 2025 but not in April 1–5, 2025.**
```
Objective: Identify customers who ordered in Jan–Mar 2025 but did not order in Apr 1–5, 2025 to detect early churn.
```
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
```
Insight: Finds likely churn customers early, enabling win-back campaigns (coupons, targeted messages).
```
**Q9. Compare restaurant-wise cancellation rate for January and February.**
```
Objective: Compare restaurant-wise cancellation rates for January and February to monitor changes in service quality.
```
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
```
Insight: Detects restaurants with worsening cancellation trends, signaling service/stock/prep issues.
```
**Q10. Find each rider’s average delivery time.**
```
Objective: Calculate each rider’s average delivery time to evaluate delivery efficiency.
```
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
```
Insight: Ranks rider efficiency to reward top riders and improve slow riders via training/route optimization.
```
**Q11. Calculate monthly delivered-order growth ratio for each restaurant.**
```
Objective: Calculate monthly delivered-order growth ratio for each restaurant to track performance trends over time.
```
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
```
Insight: Tracks restaurant growth/decline over months to guide promotion and operational support decisions.
```
**Q12. Segment customers into Gold and Silver and show total orders and revenue per segment.**
```
Objective: Segment customers into Gold and Silver based on order value to support targeted marketing.
```
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
```
Insight: Enables differentiated marketing—retain Gold customers and encourage Silver customers to increase spend. 
```
**Q13. Calculate each rider’s monthly earnings (8% of order amount).**
```
Objective: Calculate each rider’s monthly earnings (8% of delivered order amount) for compensation tracking.
```
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
```
Insight: Provides month-wise rider earnings for payroll planning and cost monitoring.
```
**Q14. Count 5-star, 4-star and 3-star ratings for each rider based on delivery time.**
```
Objective: Classify deliveries into rating buckets (5-star/4-star/3-star) based on delivery time and count ratings per rider.
```
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
```
Insight: Shows rider service quality distribution (based on delivery speed) to support incentives and performance management.
```
**Q15. Find the peak order day of the week for each restaurant.**
```
Objective: Identify the peak order day of the week for each restaurant to understand weekly demand patterns.
```
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
```
Insight: Helps restaurants plan staffing and inventory on their busiest weekday; supports day-specific promotions.
```
**Q16. Calculate customer lifetime value (total revenue per customer).**
```
Objective: Calculate customer lifetime value (CLV) as total revenue per customer.
```
``` sql
select c.customer_id,c.customer_name,sum(o.total_amount) as CLV 
from customers as c
join orders as o
on c.customer_id=o.customer_id
WHERE o.order_status IN ('Completed', 'Pending')
group by 1,2
order by 3 desc;
```
```
Insight: Identifies high-CLV customers to prioritize retention and personalize engagement.
```
**Q17. Compare monthly sales with previous month and calculate growth rate.**
```
Objective: Compare monthly sales with the previous month and calculate growth rate to monitor business performance.
```
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
```
Insight: Monitors business health through month-over-month sales changes and flags periods needing investigation.
```
**Q18. Find fastest and slowest riders based on average delivery time.**
```
Objective: Identify the fastest and slowest riders based on average delivery time to benchmark performance extremes.
```
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
```
Insight: Quantifies performance gap between fastest and slowest riders to drive targeted improvements.
```
**Q19. Track monthly popularity of order items and identify demand trends.**
```
Objective: Track monthly popularity of order items to identify demand trends over time.
```
``` sql
select order_item,to_char(order_date,'month') as month,count(*)as total_order
from orders
where order_status in ('Completed', 'Pending')
group by 1,2
order by 1,3 desc;
```
```
Insight: Reveals item-level demand trends to support forecasting, menu decisions, and seasonal promotions.
```
**Q20. Rank each city by total revenue for the year.**
```
Objective: Rank each city by total annual revenue to identify top and underperforming markets.
```
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
```
 Insight: Identifies top and underperforming cities to guide budget allocation and expansion strategy.
```

