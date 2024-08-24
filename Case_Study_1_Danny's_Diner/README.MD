# 8-Week SQL Challenge:Case Study #1: Danny's Diner

<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" width="500">

## The best solutions you can find on the internet are in this repo. I also shared the solution to this case on my YouTube channel (unfortunately only in Turkish).


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

