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
```

**Q4. Find customers whose total spending is more than 25,000.**
``` sql
```
**Q5. Find restaurant-wise number of not delivered orders with city.**
``` sql
```
**Q6. Rank restaurants by total revenue (last year) within each city.**
``` sql
```
**Q7. Find the most popular dish in each city.**
``` sql
```
**Q8. Find customers who ordered in Jan–Mar 2025 but not in April 1–5, 2025.**
``` sql
```
**Q9. Compare restaurant-wise cancellation rate for January and February.**
``` sql
```
**Q10. Find each rider’s average delivery time.**
``` sql
```
**Q11. Calculate monthly delivered-order growth ratio for each restaurant.**
``` sql
```
**Q12. Segment customers into Gold and Silver and show total orders and revenue per segment.**
``` sql
```
**Q13. Calculate each rider’s monthly earnings (8% of order amount).**
``` sql
```
**Q14. Count 5-star, 4-star and 3-star ratings for each rider based on delivery time.**
``` sql
```
**Q15. Find the peak order day of the week for each restaurant.**
``` sql
```
**Q16. Calculate customer lifetime value (total revenue per customer).**
``` sql
```
**Q17. Compare monthly sales with previous month and calculate growth rate.**
``` sql
```
**Q18. Find fastest and slowest riders based on average delivery time.**
``` sql
```
**Q19. Track monthly popularity of order items and identify demand trends.**
``` sql
```
**Q20. Rank each city by total revenue for the year 2023.**
``` sql
```
