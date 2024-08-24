# 8-Week SQL Challenge:Case Study #1: Danny's Diner

<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" width="500">

##The best solutions you can find on the internet are in this repo. I have solved and explained each question step by step. I also made explanations after each question as if I was preparing a report. I also shared the solution to this case on my YouTube channel (unfortunately only in Turkish).


[![Watch the video](https://img.youtube.com/vi/SfdS3WREZEo/maxresdefault.jpg)](https://youtu.be/SfdS3WREZEo)

## Case Study Questions
### 1. What is the total amount each customer spent at the restaurant?
#### Explanation 
The menu table contains the price for each product. We would have to join the sales and menu tables then sum the prices for the orders made by each customer after grouping by the customer_id .

``` sql
SELECT CUSTOMER_ID,
	SUM(PRICE)
FROM SALES S
JOIN MENU M ON M.PRODUCT_ID = S.PRODUCT_ID
GROUP BY 1;
```
#### Output
![Q1](https://github.com/user-attachments/assets/4a059577-db03-4e4f-b4f5-e9c5e74a5823)

### Conclusion

Conclusion
This table shows us the amount spent by each customer. 
The spending of customer A and customer B is very close to each other and about twice as much as the spending of customer C. 
Work could be done to increase Customer C's spending to the level of Customer A and Customer B, or to investigate why Customer C is not spending as much as the others.

### 2. How many days has each customer visited the restaurant?

#### Explanation 
When solving this question, we need to pay attention that we must use the distinct expression when counting the order dates. Because here Danny is asking us how many days each customer visited the restaurant. When we examine the table, customers can make more than one visit on the same day. 

``` sql
select customer_id,
	count(distinct order_date)
from sales
group by 1;
```
#### Output
![image](https://github.com/user-attachments/assets/075d7366-6624-4a50-b8b1-65d4861b3dc6)

### Conclusion
Customer A visited the restaurant 4 times, Customer B visited the restaurant 6 times and Customer C visited the restaurant twice. Customer B is the most loyal customer and Customer A could be encouraged to visit the restaurant more often. We should also check if customer C is still a customer and if not, we can investigate why not.

### 3. What was the first item from the menu purchased by each customer?

#### Explanation 
#### Stage_1

As the first step, I call row_number, rank and dense_rank window functions with customer_id, order_date and product_name. I group by customer and sort by order date. Then I need to decide which of these 3 window functions is suitable for me.

```sql
select customer_id,
		order_date,
		product_name,
		row_number() over(partition by customer_id order by order_date ),
		rank() over(partition by customer_id order by order_date ),
		dense_rank() over(partition by customer_id order by order_date )
from sales s
join menu m on m.product_id = s.product_id
order by customer_id, order_date;
```
#### Output
![stage_1](https://github.com/user-attachments/assets/bd77a5fc-b0ef-41f6-ba30-3e6932658b11)

#### Stage_2

If I choose row number, it will bring me the first one in the table from the products purchased on the same date. But since there is no time information on the dates, I don't know which one was purchased first. So I think it is a better choice to get all the products purchased on the same date and I choose to use the rank function. If I use the dense_rank function, I get the same output. 

```sql
with table_1 as(
select customer_id,
	order_date,
	product_name,
	rank() over(partition by customer_id order by order_date ) rn
from sales s
join menu m on m.product_id = s.product_id
)
select customer_id,
	product_name
from table_1
where rn = 1;
```
#### Output
![image](https://github.com/user-attachments/assets/db07dbe9-6239-4d17-8c92-abbf5bc32805)

#### Stage_3

Although this is a correct output, it is not very nice that the last two lines are the same. I use a distinct statement to get rid of this.

```sql
with table_1 as(
select customer_id,
	order_date,
	product_name,
	rank() over(partition by customer_id order by order_date ) rn
from sales s
join menu m on m.product_id = s.product_id
)
select distinct customer_id,
	product_name
from table_1
where rn = 1;
```
#### Output
![image](https://github.com/user-attachments/assets/3ad523d3-5220-45af-9e73-ef6e308cd718)

### Conclusion

Customer A ordered sushi and curry for the first time. We don't know which of these she ordered first because the data doesn't have time information, but we know that she ordered both on the same day. 
Customer B ordered curry for the first time.
Customer C ordered ramen for the first time. 

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

#### Explanation 
#### Stage_1

First we need to find out how many of each product are sold. 

```sql
SELECT product_name,
	count(s.product_id)
from sales s
join menu m on m.product_id = s.product_id
group by 1;
```
#### Output
![image](https://github.com/user-attachments/assets/35c4b3eb-75ce-4d5b-9ca2-b61ef62d036c)

#### Stage_2

At this stage, we sort the products according to the number of sales descending and take the first row with the limit command.
``` sql
SELECT product_name,
	count(s.product_id)
from sales s
join menu m on m.product_id = s.product_id
group by 1
order by 2 DESC
limit 1;
```
#### Output
![image](https://github.com/user-attachments/assets/61fc2058-7804-46d7-802d-4cb8bae23644)

### Conclusion
Since ramen is the most ordered product, it is important to pay close attention to the supply of the ingredients required for this product. Thus, it is necessary to make sure that the demand can always be met. 

### 5. Which item was the most popular for each customer?

#### Explanation 
#### Stage_1

First, we find out how many of each product each customer buys. 

```sql
SELECT customer_id,
	product_name,
	count(s.product_id) order_count
from sales s
join menu m on m.product_id = s.product_id
group by 1,2
order by customer_id, 3 DESC;
```
#### Output
![image](https://github.com/user-attachments/assets/0dbb859b-1708-4cf5-88ab-6b541c730d66)

#### Stage_2

At this stage, with the rank function, we sort the number of products ordered by customers based on customer_id in descending order. The reason we chose the rank function here is that there are equal number of ordered products. I thought it was a better choice to get all of the equal number of products. 
Then we turn the query into a CTE and list the products that each customer has ordered the most. 

```sql
with tablo as(
SELECT customer_id,
	product_name,
	count(s.product_id) order_count,
	rank() over(PARTITION by customer_id order by count(s.product_id) desc)	rn
from sales s
join menu m on m.product_id = s.product_id
group by 1,2) 
SELECT customer_id,
	product_name,
	order_count
from tablo
where rn = 1;
```
#### Output
![image](https://github.com/user-attachments/assets/c273eb57-19e3-49d7-8f02-8227d4bbe757)

### Conclusion
Customer A's most ordered product is ramen. Customer B ordered curry, sushi and ramen in equally . Customer C's favorite product is ramen

### 6. Which item was purchased first by the customer after they became a member?
#### Explanation 
#### Stage_1

First of all, I join the menu table with the sales table. Then I first join with the members table to show the difference. Then I do left join. 
The outputs are as follows. 
```sql
select * 
from sales s
join menu m on m.product_id = s.product_id
join members mem on s.customer_id = mem.customer_id;
```
![image](https://github.com/user-attachments/assets/a7672985-f920-46ac-a535-3fc475b3dd55)

```sql
select * 
from sales s
join menu m on m.product_id = s.product_id
left join members mem on s.customer_id = mem.customer_id
```
![image](https://github.com/user-attachments/assets/6df4bd39-9ce1-48ec-8691-8e3574670e67)

Since we did join operation in our first query, we obtained the common rows in both sales and members table in our output. Therefore, our table consists of 12 rows.
In our second query, since we did left join, values that are not in the members table also appear. so our table consists of 15 rows. 




