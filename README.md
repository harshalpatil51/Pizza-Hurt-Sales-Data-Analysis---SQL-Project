--total number of orders placed.
select count(order_id) as TOTAL_ORDERS
from orders;

--Calculate the total revenue generated from pizza sales.
select SUM(od.quantity * pi.price) as TOTAL_REVENUE
from order_details od
JOIN pizzas pi
ON pi.pizza_id = od.pizza_id;

--Identify the highest-priced pizza.
select pt.name, p.price as highest_price_pizza
from pizzas p
JOIN pizza_types pt
ON pt.pizza_type_id = p.pizza_type_id
ORDER BY p.price DESC LIMIT 1;

--Identify the most common pizza size ordered.
select 
      p.size as size,
      count(od.order_details_id) as order_count
from pizzas p
JOIN order_details od ON od.pizza_id = p.pizza_id
GROUP BY p.size 
ORDER BY order_count desc limit 1;

--List the top 5 most ordered pizza types along with their quantities.
SELECT pizzas.pizza_type_id,SUM(order_details.quantity) as topfive_order_pizzas
FROM order_details
JOIN pizzas 
ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.pizza_type_id
ORDER BY topfive_order_pizzas DESC
LIMIT 5;
------- or ------
SELECT 
      pt.name,
      p.pizza_type_id,
      sum(od.quantity) as quantity
FROM pizzas p
JOIN order_details od ON od.pizza_id = p.pizza_id
JOIN pizza_types pt ON pt.pizza_type_id = p.pizza_type_id
GROUP BY p.pizza_type_id,pt.name
order by quantity desc limit 5;

-- Find the Total Quantity of each Pizza Category ordered.
SELECT 
      pt.category,
      SUM(od.quantity) AS Total_Orders
FROM pizza_types pt
JOIN pizzas p ON p.pizza_type_id = pt.pizza_type_id
JOIN order_details od ON p.pizza_id = od.pizza_id
GROUP BY pt.category
ORDER BY total_orders DESC;

--Determine the distribution of orders by Hour of the day.
SELECT
       EXTRACT(HOUR FROM order_time) AS order_hour,
       COUNT(order_id) as total_order
FROM orders
GROUP BY order_hour
ORDER BY total_order desc;

--Find the Category-Wise Distribution of pizzas.

SELECT category,
            count(name)
FROM pizza_types
GROUP BY category;

--Group the orders by date and calculate the average number of pizzas ordered per day.
select round(avg(quantity)) As AVG_PerDay_Orders 
from (SELECT orders.order_date,
      sum(order_details.quantity) as quantity
FROM orders
JOIN order_details ON order_details.order_id = orders.order_id
GROUP BY order_date
ORDER BY order_date) as order_quantity;

--Determine the Top 3 Most Ordered pizza types based on Revenue.

SELECT 
    pizza_types.name,
    SUM(order_details.quantity * pizzas.price) AS Revenue
FROM pizza_types
JOIN pizzas ON pizzas.pizza_type_id = pizza_types.pizza_type_id
JOIN order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;

--Analyze the Cumulative Revenue generated over time.
SELECT 
      order_date,
      SUM(daily_revenue) OVER(order by order_date) as cummulative_revenue
FROM (SELECT 
      DATE(orders.order_date) as order_date,
      SUM(order_details.quantity * pizzas.price) as daily_revenue
FROM pizzas
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
JOIN orders ON orders.order_id = order_details.order_id
GROUP BY orders.order_date
ORDER BY orders.order_date) AS t1;

--Determine the top 3 most ordered pizza types based on revenue for each pizza category.
SELECT a,b,revenue,rn
FROM(SELECT 
      t1.a,
      t1.b,
    t1.revenue,
      RANK() OVER(partition by A order by revenue desc) as rn
FROM (SELECT pizza_types.category as a,
         pizza_types.name as b,
         SUM(order_details.quantity*pizzas.price) AS revenue
FROM pizza_types
JOIN pizzas ON pizzas.pizza_type_id = pizza_types.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category,pizza_types.name
ORDER BY pizza_types.category) AS t1) AS T2
WHERE rn <= 3;
