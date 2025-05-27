```sql
use Case2

CREATE TABLE runners (
  "runner_id" INTEGER,
  "registration_date" DATE
);
INSERT INTO runners
  ("runner_id", "registration_date")
VALUES
  (1, '2021-01-01'),
  (2, '2021-01-03'),
  (3, '2021-01-08'),
  (4, '2021-01-15');


CREATE TABLE customer_orders (
  "order_id" INTEGER,
  "customer_id" INTEGER,
  "pizza_id" INTEGER,
  "exclusions" VARCHAR(4),
  "extras" VARCHAR(4),
  "order_time" DATETIME
);

INSERT INTO customer_orders
  ("order_id", "customer_id", "pizza_id", "exclusions", "extras", "order_time")
VALUES
  ('1', '101', '1', '', '', '2020-01-01 18:05:02'),
  ('2', '101', '1', '', '', '2020-01-01 19:00:52'),
  ('3', '102', '1', '', '', '2020-01-02 23:51:23'),
  ('3', '102', '2', '', NULL, '2020-01-02 23:51:23'),
  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
  ('4', '103', '2', '4', '', '2020-01-04 13:23:46'),
  ('5', '104', '1', 'null', '1', '2020-01-08 21:00:29'),
  ('6', '101', '2', 'null', 'null', '2020-01-08 21:03:13'),
  ('7', '105', '2', 'null', '1', '2020-01-08 21:20:29'),
  ('8', '102', '1', 'null', 'null', '2020-01-09 23:54:33'),
  ('9', '103', '1', '4', '1, 5', '2020-01-10 11:22:59'),
  ('10', '104', '1', 'null', 'null', '2020-01-11 18:34:49'),
  ('10', '104', '1', '2, 6', '1, 4', '2020-01-11 18:34:49');


CREATE TABLE runner_orders (
  "order_id" INTEGER,
  "runner_id" INTEGER,
  "pickup_time" VARCHAR(19),
  "distance" VARCHAR(7),
  "duration" VARCHAR(10),
  "cancellation" VARCHAR(23)
);

INSERT INTO runner_orders
  ("order_id", "runner_id", "pickup_time", "distance", "duration", "cancellation")
VALUES
  ('1', '1', '2020-01-01 18:15:34', '20km', '32 minutes', ''),
  ('2', '1', '2020-01-01 19:10:54', '20km', '27 minutes', ''),
  ('3', '1', '2020-01-03 00:12:37', '13.4km', '20 mins', NULL),
  ('4', '2', '2020-01-04 13:53:03', '23.4', '40', NULL),
  ('5', '3', '2020-01-08 21:10:57', '10', '15', NULL),
  ('6', '3', 'null', 'null', 'null', 'Restaurant Cancellation'),
  ('7', '2', '2020-01-08 21:30:45', '25km', '25mins', 'null'),
  ('8', '2', '2020-01-10 00:15:02', '23.4 km', '15 minute', 'null'),
  ('9', '2', 'null', 'null', 'null', 'Customer Cancellation'),
  ('10', '1', '2020-01-11 18:50:20', '10km', '10minutes', 'null');


CREATE TABLE pizza_names (
  "pizza_id" INTEGER,
  "pizza_name" VARCHAR(100)
);
INSERT INTO pizza_names
  ("pizza_id", "pizza_name")
VALUES
  (1, 'Meatlovers'),
  (2, 'Vegetarian');


CREATE TABLE pizza_recipes (
  "pizza_id" INTEGER,
  "toppings" VARCHAR(100)
);
INSERT INTO pizza_recipes
  ("pizza_id", "toppings")
VALUES
  (1, '1, 2, 3, 4, 5, 6, 8, 10'),
  (2, '4, 6, 7, 9, 11, 12');


CREATE TABLE pizza_toppings (
  "topping_id" INTEGER,
  "topping_name" VARCHAR(100)
);
INSERT INTO pizza_toppings
  ("topping_id", "topping_name")
VALUES
  (1, 'Bacon'),
  (2, 'BBQ Sauce'),
  (3, 'Beef'),
  (4, 'Cheese'),
  (5, 'Chicken'),
  (6, 'Mushrooms'),
  (7, 'Onions'),
  (8, 'Pepperoni'),
  (9, 'Peppers'),
  (10, 'Salami'),
  (11, 'Tomatoes'),
  (12, 'Tomato Sauce');

select * from runners
select * from customer_orders
select * from runner_orders
select * from pizza_names
select * from pizza_recipes
select * from pizza_toppings

---------- A. Pizza Metrics
------ Câu 1A: Tổng đã đặt bao nhiêu chiếc pizza?
select count(order_id) as total_pizza
from customer_orders

------ Câu 2A: Có bao nhiêu khách hàng đã đặt?
select count(distinct customer_id) as total_customer
from customer_orders

------ Câu 3A: Mỗi người giao hàng đã giao thành công bao nhiêu đơn hàng?
select runner_id, count(order_id) as total_order
from runner_orders
where cancellation is null or cancellation = 'NULL' or cancellation = ''
group by runner_id

------ Câu 4A: Có bao nhiêu chiếc pizza của mỗi loại đã được giao thành công?
select pizza_name, count(co.pizza_id) as total_pizza
from pizza_names pn
join customer_orders co on pn.pizza_id = co.pizza_id
join runner_orders ro on co.order_id = ro.order_id
where cancellation is null or cancellation = 'NULL' or cancellation = ''
group by pizza_name 

------ Câu 5A: Mỗi khách hàng đã gọi bao nhiêu Meatlovers và Vegetarian?
select customer_id, pizza_name, count(co.pizza_id)
from customer_orders co 
join pizza_names pn on co.pizza_id = pn.pizza_id
group by customer_id, pizza_name
order by customer_id
	
------ Câu 6A: Số lượng pizza tối đa được giao trong một đơn hàng là bao nhiêu?
select top 1 co.order_id, count(pizza_id) as total_pizza
from customer_orders co
join runner_orders ro on co.order_id = ro.order_id
where cancellation is null or cancellation = 'NULL' or cancellation = ''
group by co.order_id
order by total_pizza desc

------ Câu 7A: Đối với mỗi khách hàng, có bao nhiêu chiếc pizza được giao có thay đổi topping và bao nhiêu chiếc không thay đổi topping ?
select customer_id,
	count(case
		when not(
			(exclusions is null or exclusions = 'NULL' or exclusions = '') and
			(extras is null or extras = 'NULL' or extras = '')
		)then 1
		end) as total_change,
	count(case
		when(
			(exclusions is null or exclusions = 'NULL' or exclusions = '') and
			(extras is null or extras = 'NULL' or extras = '')
		)then 1
		end) as total_not_change
from customer_orders co 
join runner_orders ro on co.order_id = ro.order_id
where cancellation is null or cancellation = 'NULL' or cancellation = ''
group by customer_id

------ Câu 8A: Có bao nhiêu chiếc pizza được giao có cả phần loại trừ và phần thêm vào?
select count(case
		when not(
			(exclusions is null or exclusions = 'NULL' or exclusions = '') or
			(extras is null or extras = 'NULL' or extras = '')
		)then 1
		end) as total_change
from customer_orders co 
join runner_orders ro on co.order_id = ro.order_id
where cancellation is null or cancellation = 'NULL' or cancellation = ''

------ Câu 9A: Tổng lượng pizza được đặt hàng trong mỗi giờ trong ngày là bao nhiêu?
select datepart(hour, order_time) as order_hour,
	count(pizza_id) as total_pizza
from customer_orders
group by datepart(hour, order_time)
order by order_hour

------ Câu 10A: Khối lượng đơn hàng cho mỗi ngày trong tuần là bao nhiêu?
select datepart(day, order_time) as order_day,
	count(pizza_id) as total_pizza
from customer_orders
group by datepart(day, order_time)
order by order_day

---------- B. Runner and Customer Experience
------ Câu 1B: Có bao nhiêu người giao hàng đăng ký cho mỗi tuần? (tức là tuần bắt đầu 2021-01-01)
------ Câu 2B: Thời gian trung bình tính bằng phút để mỗi người giao hàng đến trụ sở Pizza Runner để nhận đơn hàng là bao lâu?
------ Câu 3B: Có mối liên hệ nào giữa số lượng pizza và thời gian chuẩn bị đơn hàng không?
------ Câu 4B: Khoảng cách trung bình của mỗi khách hàng là bao nhiêu?
------ Câu 5B: Sự khác biệt giữa thời gian giao hàng dài nhất và ngắn nhất cho tất cả các đơn hàng là gì?
------ Câu 6B: Tốc độ trung bình của mỗi người chạy trong mỗi lần giao hàng là bao nhiêu và bạn có nhận thấy xu hướng nào cho các giá trị này không?
------ Câu 7B: Tỷ lệ giao hàng thành công của mỗi người vận chuyển là bao nhiêu?


---------- C. Ingredient Optimisation
------ Câu 1C: Thành phần tiêu chuẩn cho mỗi chiếc pizza là gì?
------ Câu 2C: Loại phụ gia nào được thêm vào nhiều nhất?
------ Câu 3C: Loại trừ phổ biến nhất là gì?
------ Câu 4C: 
------ Câu 5C:
------ Câu 6C: Tổng lượng của từng loại nguyên liệu được sử dụng trong tất cả các loại pizza được giao 
---------------theo thứ tự xuất hiện thường xuyên nhất trước là bao nhiêu?

---------- D. Pricing and Ratings
------ Câu 1D: Nếu một chiếc pizza Meat Lovers có giá 12 đô la và Vegetarian có giá 10 đô la và không tính phí thay đổi 
---------------thì Pizza Runner đã kiếm được bao nhiêu tiền nếu không tính phí giao hàng?
------ Câu 2D:
------ Câu 3D: Nhóm Pizza Runner hiện muốn thêm một hệ thống xếp hạng bổ sung cho phép khách hàng đánh giá người phục vụ của họ, 
---------------bạn sẽ thiết kế bảng bổ sung cho tập dữ liệu mới này như thế nào - tạo lược đồ cho bảng mới này và chèn dữ liệu của 
---------------riêng bạn để đánh giá cho mỗi đơn hàng thành công của khách hàng từ 1 đến 5.
------ Câu 4D:
------ Câu 5D: Nếu một chiếc pizza Meat Lovers có giá cố định là 12 đô la và Vegetarian có giá cố định là 10 đô la, 
---------------không tính thêm phí cho đồ ăn thêm và mỗi người được trả 0,30 đô la cho mỗi km di chuyển
---------------thì Pizza Runner còn lại bao nhiêu tiền sau những lần giao hàng này?
