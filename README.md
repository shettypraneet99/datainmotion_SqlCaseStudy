# datainmotion_SqlCaseStudy
#SQL Case Study - 1 of Data In Motion, LLC.
#This challenge involves the sales analysis of Tiny Shop over time. The database consists of four different tables that tell the overview of products-customers-#orders. I have used MySQL server.

The functions used were:
•Basic aggregations
•CASE WHEN statements
•Window Functions
•Joins
•Date time functions
•CTEs

-- 1) Which product has the highest price? Only return a single row.
select * from products where price =(select max(price) from products);

-- 2) Which customer has made the most orders?
select c.customer_id, c.first_name, c.last_name, count(o.order_id) as no_of_orders from customers as c
inner join orders as o on c.customer_id=o.customer_id group by c.customer_id;

-- 3) What’s the total revenue per product?
select * from order_items;
select * from products;
select p.product_id, p.product_name, Sum(p.price*o.quantity) as revenue from products as p
join order_items as o on p.product_id=o.product_id
group by p.product_id,p.product_name;

-- 4) Find the day with the highest revenue.
select orders.order_date, Sum(products.price * order_items.quantity) as Total_revenue from orders
inner join order_items on orders.order_id=order_items.order_id
inner join products on order_items.product_id=products.product_id
group by orders.order_date order by Total_revenue DESC LIMIT 1;

-- 5) Find the first order (by date) for each customer.
select c.customer_id, c.first_name, c.last_name, o.order_id, o.order_date from customers as c
join orders as o on c.customer_id=o.customer_id
where o.order_date=
(Select Min(order_date) from orders where customer_id=c.customer_id)
order by o.order_date;

-- 6) Find the top 3 customers who have ordered the most distinct products
select c.customer_id, c.first_name, c.last_name, count(distinct ot.product_id) as special_products from customers as c 
join orders as o on c.customer_id=o.customer_id
join order_items as ot on o.order_id=ot.order_id
group by c.customer_id,c.first_name, c.last_name
order by special_products DESC Limit 3;


-- 7) Which product has been bought the least in terms of quantity?
select p.product_id, p.product_name, sum(ot.quantity) as least_quantity from products as p
join order_items as ot on p.product_id=ot.product_id
group by p.product_id, p.product_name
order by least_quantity ASC; 

-- 8) What is the median order total?
with order_totals as (
select sum(p.price* ot.quantity) as order_total from order_items as ot
join products as p on ot.product_id=p.product_id
group by ot.order_id)
select median_order_total from (
select order_total as median_order_total,
row_number() over (ORDER BY order_total) as row_num,
count(*) over() as total_rows
from order_totals) as subquery 
where row_num=ceil(total_rows/2.0);


-- 9) For each order, determine if it was ‘Expensive’ (total over 300), ‘Affordable’ (total over 100), or ‘Cheap’.
select order_id, order_total,
case when order_total > 300 then "Expensive"
	 when order_total > 100 then "Affordable"
else "Cheap"
end as order_category
from(
select ot.order_id, sum(p.price*ot.quantity) as order_total from order_items as ot
join products as p on ot.product_id=p.product_id
group by ot.order_id) as order_total;

-- 10) Find customers who have ordered the product with the highest price.
select c.customer_id, c.first_name, c.last_name, p.price from customers as c
join orders as o on c.customer_id=o.customer_id
join order_items as ot on o.order_id=ot.order_id
join products as p on ot.product_id=p.product_id
where p.price=(select max(price) from products)
group by c.customer_id, c.first_name, c.last_name, p.price;
