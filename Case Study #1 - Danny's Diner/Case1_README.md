```sql
use Case1

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');


CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');


CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');

select * from sales
select * from menu
select * from members

------ Câu 1: Tổng số tiền mà mỗi khách hàng chi tiêu tại nhà hàng là bao nhiêu?
select customer_id, sum(price) as total_price
from sales s
join menu m on s.product_id = m.product_id
group by customer_id

------ Câu 2: Mỗi khách hàng đã ghé thăm nhà hàng bao nhiêu ngày?
select customer_id, count(distinct order_date) as total_buy
from sales s 
join menu m on s.product_id = m.product_id
group by customer_id

------ Câu 3: Món đầu tiên trong thực đơn mà mỗi khách hàng mua là gì?
with first_meal as (
	select customer_id, order_date, product_name,
	dense_rank() over(
		partition by customer_id
		order by order_date) as rank
	from sales s 
	join menu m on s.product_id = m.product_id
	)
select distinct customer_id, product_name
from first_meal
where rank = 1

------ Câu 4: Món ăn nào trong thực đơn được mua nhiều nhất và được tất cả khách hàng mua bao nhiêu lần?
select top 1 product_name, count(s.product_id) as total_buy
from sales s 
join menu m on s.product_id = m.product_id
group by product_name
order by total_buy desc

------ Câu 5: Mỗi khách hàng ưa chuộng món ăn nào nhất?
with popular_meal as (
	select customer_id, product_name, count(s.product_id) as total_buy,
	dense_rank() over(
		partition by customer_id
		order by count(s.product_id) desc ) as rank
	from sales s
	join menu m on s.product_id = m.product_id
	group by customer_id, product_name
	)
select customer_id, product_name, total_buy
from popular_meal
where rank = 1

------ Câu 6: Khách hàng đã mua món ăn nào đầu tiên sau khi trở thành thành viên?
with first_meal_mem as (
	select s.customer_id, product_name, order_date,
	dense_rank() over(
		partition by s.customer_id
		order by order_date) as rank
	from sales s 
	join menu m on s.product_id = m.product_id
	join members me on s.customer_id = me.customer_id
	where order_date >= join_date
	)
select customer_id, product_name
from first_meal_mem
where rank = 1

------ Câu 7: Món ăn nào được mua ngay trước khi khách hàng trở thành thành viên?
with first_meal_mem as (
	select s.customer_id, product_name, order_date,
	dense_rank() over(
		partition by s.customer_id
		order by order_date desc) as rank
	from sales s 
	join menu m on s.product_id = m.product_id
	join members me on s.customer_id = me.customer_id
	where order_date < join_date
	)
select customer_id, product_name
from first_meal_mem
where rank = 1

------ Câu 8: Tổng số món ăn và số tiền mà mỗi thành viên đã chi trước khi trở thành thành viên là bao nhiêu?
select s.customer_id, count(distinct s.product_id) as total_product, sum(price) as total_price
from sales s
join menu m on s.product_id = m.product_id
join members me on s.customer_id = me.customer_id
where order_date < join_date
group by s.customer_id

------ Câu 9: Nếu mỗi $1 chi tiêu tương ứng với 10 điểm và sushi có hệ số nhân điểm là 2x thì mỗi khách hàng sẽ có bao nhiêu điểm?
select customer_id, 
	sum(
		case
		when product_name = 'shusi' then price *20
		else price * 10
		end) as total_score
from sales s
join menu m on s.product_id = m.product_id
group by customer_id

------ Câu 10: Trong tuần đầu tiên sau khi khách hàng tham gia chương trình (bao gồm cả ngày tham gia), 
---------------họ sẽ kiếm được gấp đôi điểm cho tất cả các mặt hàng, không chỉ sushi
---------------khách hàng A và B có bao nhiêu điểm vào cuối tháng 1?
with dates_cte as (
select
	customer_id, 
	join_date,
    dateadd(day, 6, join_date) as valid_date,
    eomonth('2021-01-31') as last_date
from members
)
select 
	s.customer_id, 
	sum(
		case
		when s.order_date between dates.join_date and dates.valid_date then 2 * 10 * m.price
		else 10 * m.price 
		end) as points
from sales s
join dates_cte as dates on s.customer_id = dates.customer_id
  and dates.join_date <= s.order_date
  and s.order_date <= dates.last_date
join menu m on s.product_id = m.product_id
group by s.customer_id
