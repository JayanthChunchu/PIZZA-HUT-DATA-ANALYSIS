## PIZZA-HUT-DATA-ANALYSIS
![Pizza-Hut-Logo](https://github.com/user-attachments/assets/21d97493-b874-4fef-a023-8425e5ff3544)

## 1.Pizza Sales Analysis Project
This project is focused on analyzing the sales data of a fictional pizza restaurant to derive meaningful business insights. The analysis aims to uncover trends in customer orders, revenue generation, and popular pizza types. By leveraging SQL queries, we can efficiently process and analyze large datasets to support data-driven decision-making.


## 2.Database Schema
The database for this project includes the following tables:

- pizzas: Contains details about each pizza, such as pizza ID, type ID, size, and price.
- pizza_types: Stores information about different types of pizzas, including type ID, name, category, and ingredients.
- orders: Records information about customer orders, including order ID, date, and time.
- order_details: Contains details about each order, such as order details ID, order ID, pizza ID, and quantity.

## 3.Analysis Goals
The analysis is designed to answer several key business questions, including:

-->The total number of orders placed.
-->Total revenue generated from pizza sales.
-->Identification of the highest-priced pizza.
-->Most common pizza size ordered.
-->Top 5 most ordered pizza types.
-->Total quantity of each pizza category ordered.
-->Distribution of orders by hour of the day.
-->Average number of pizzas ordered per day.
-->Category-wise distribution of pizzas.
-->Cumulative revenue over time.
-->Top 3 most ordered pizza types based on revenue for each category.

## 4.Tools Used
# (a.)SQL
Structured Query Language (SQL) is the primary tool used in this project for data extraction, manipulation, and analysis. SQL allows us to write queries that efficiently retrieve and process data from the database, enabling us to answer complex business questions.

# (b.)Database Management System
A relational database management system (RDBMS) such as MySQL  is used to store and manage the data. The RDBMS provides a robust and scalable environment for handling the pizza sales data.

```Create Database
CREATE DATABASE pizzahut;
USE pizzahut;
```
##  Retrieve the total number of orders placed.
SELECT 
    COUNT(order_id) AS total_orders
FROM
    orders;
## Objective :To retrieve the total number of orders placed by counting the order_id values from the orders table.
``` SELECT 
    COUNT(order_id) AS total_orders
FROM
    orders;
```
## Objective :Compute total revenue from pizza sales with rounding.
```
SELECT 
    ROUND(SUM(od.quantity * p.price), 2) AS total_revenue
FROM
    order_details od
JOIN
    pizzas p ON od.pizza_id = p.pizza_id;
```
## Find the highest-priced pizza and its name.
```
SELECT 
    pt.name AS pizza_name, p.price
FROM
    pizzas p
        JOIN
    pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
ORDER BY p.price DESC
LIMIT 1;
```
## Objective :Find the most ordered pizza size and its count.
```
SELECT 
    size, COUNT(*) AS size_count
FROM
    pizzas p
        JOIN
    order_details od ON p.pizza_id = od.pizza_id
GROUP BY size
ORDER BY size_count DESC
LIMIT 1;
```
## Objective :Retrieve the top 5 most ordered pizza types with their total quantities.
```
SELECT 
    pt.name, SUM(od.quantity) AS total_quantity
FROM
    order_details od
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id
        JOIN
    pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name
ORDER BY total_quantity DESC
LIMIT 5;
```
## Objective :Find the total quantity ordered for each pizza category.
``` SELECT 
    pt.category, SUM(od.quantity) AS total_quantity
FROM
    order_details od
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id
        JOIN
    pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category
ORDER BY total_quantity DESC;
```
## Objective :Objective : Analyze order distribution by hour of the day.
```
SELECT 
    HOUR(order_time) AS order_hour,
    COUNT(order_id) AS total_orders
FROM
    orders
GROUP BY HOUR(order_time)
ORDER BY order_hour;
```
## Objective : Find the distribution of pizza sales by category.
```
SELECT 
    pt.category, COUNT(od.order_details_id) AS order_count
FROM
    order_details od
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id
        JOIN
    pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category
ORDER BY order_count DESC;
```
## Objective : Calculate the average number of pizzas ordered per day.
```
SELECT 
    ROUND(AVG(daily_orders.total_pizzas), 0) AS avg_pizzas_per_day
FROM
    (SELECT 
        order_date, SUM(quantity) AS total_pizzas
    FROM
        orders o
    JOIN order_details od ON o.order_id = od.order_id
    GROUP BY order_date) daily_orders;
```
## Objective : Identify the top 3 pizza types generating the highest revenue.
```
SELECT 
    pt.name, SUM(od.quantity * p.price) AS total_revenue
FROM
    order_details od
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id
        JOIN
    pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name
ORDER BY total_revenue DESC
LIMIT 3;
```
## Objective : Calculate each pizza type's percentage contribution to total revenue.
```
SELECT 
    pr.name,
    pr.pizza_rev,
    ROUND((pr.pizza_rev / tr.total_rev) * 100, 2) AS percentage_contribution
FROM
    (SELECT 
        pt.name, SUM(od.quantity * p.price) AS pizza_rev
    FROM
        order_details od
    JOIN pizzas p ON od.pizza_id = p.pizza_id
    JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
    GROUP BY pt.name) pr,
    (SELECT 
        SUM(od.quantity * p.price) AS total_rev
    FROM
        order_details od
    JOIN pizzas p ON od.pizza_id = p.pizza_id) tr
ORDER BY percentage_contribution DESC;  
```
## Objective : Track cumulative revenue growth over time.
```
SELECT 
    order_date, 
    ROUND(SUM(daily_revenue) OVER(ORDER BY order_date), 2) AS cumulative_revenue
FROM (
    SELECT 
        order_date, 
        SUM(od.quantity * p.price) AS daily_revenue
    FROM 
        orders o
    JOIN 
        order_details od ON o.order_id = od.order_id
    JOIN 
        pizzas p ON od.pizza_id = p.pizza_id
    GROUP BY 
        order_date
) daily_revenues
ORDER BY 
    order_date;
```
## Objective : Identify the top 3 highest-revenue pizza types within each category.
```
SELECT category, name, ROUND(revenue, 2) AS revenue
FROM (
    SELECT 
        category,
        name,
        revenue,
        RANK() OVER(PARTITION BY category ORDER BY revenue DESC) AS rn
    FROM (
        SELECT 
            pt.category, 
            pt.name, 
            SUM(od.quantity * p.price) AS revenue
        FROM 
            pizza_types pt
        JOIN 
            pizzas p ON pt.pizza_type_id = p.pizza_type_id
        JOIN 
            order_details od ON od.pizza_id = p.pizza_id
        GROUP BY 
            pt.category, pt.name
    ) AS a
) AS b
WHERE rn <= 3
ORDER BY category, revenue DESC;
```
## Finding And Conclusions
-1.Total revenue and top-selling pizzas identified.
-2.Peak order hours and daily average pizza sales analyzed.
-3.Most popular pizza categories and sizes determined.
-4.Cumulative revenue trend and percentage contribution calculated.
-5.Insights for promotions, inventory, and business growth derived.




























