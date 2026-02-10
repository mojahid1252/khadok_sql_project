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
--**creating Table**
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
                                              



