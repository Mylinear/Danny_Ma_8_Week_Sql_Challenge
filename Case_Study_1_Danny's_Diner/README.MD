# 8-Week SQL Challenge:Case Study #1: Danny's Diner

<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" width="500">

## Overview
This repository contains the best solutions you'll find on the internet for Danny's Diner case study. I've solved and explained each question step by step, providing detailed explanations after each question, almost like preparing a report. Additionally, I've shared the solution to this case on my [YouTube channel](https://youtu.be/SfdS3WREZEo) (only available in Turkish).

<p align="center">
  <a href="https://youtu.be/SfdS3WREZEo">
    <img src="https://img.youtube.com/vi/SfdS3WREZEo/maxresdefault.jpg" alt="YouTube Video" width="600">
  </a>
</p>


## Case Study Questions
### 1. What is the total amount each customer spent at the restaurant?

**Explanation**  
To calculate the total amount each customer spent, we need to join the `sales` and `menu` tables, then sum the prices of the orders made by each customer. We group the results by `customer_id` to get the totals.


``` sql
SELECT CUSTOMER_ID,
	SUM(PRICE)
FROM SALES S
JOIN MENU M ON M.PRODUCT_ID = S.PRODUCT_ID
GROUP BY 1;
```
#### Output
![Q1](https://github.com/user-attachments/assets/4a059577-db03-4e4f-b4f5-e9c5e74a5823)


**Conclusion**  
This table shows the total amount spent by each customer. Customers A and B spent almost the same amount, which is about twice as much as what Customer C spent. We could explore strategies to increase Customer C's spending to match that of Customers A and B, or investigate why Customer C is spending less.

### 2. How many days has each customer visited the restaurant?

**Explanation**  
When answering this question, it's important to use the `DISTINCT` keyword when counting the order dates, as Danny is asking for the number of distinct days each customer visited the restaurant. Customers may have made multiple visits on the same day, so we need to count unique dates.

```sql
SELECT CUSTOMER_ID,
	COUNT(DISTINCT ORDER_DATE)
FROM SALES
GROUP BY 1;
```

#### Output
![image](https://github.com/user-attachments/assets/075d7366-6624-4a50-b8b1-65d4861b3dc6)

**Conclusion**  
Customer A visited the restaurant 4 times, Customer B visited 6 times, and Customer C visited twice. Customer B is the most loyal customer. We could encourage Customer A to visit more frequently and investigate why Customer C's visits are so few.

### 3. What was the first item from the menu purchased by each customer?

**Explanation**  
**Stage 1:**  
First, I used the `ROW_NUMBER`, `RANK`, and `DENSE_RANK` window functions with `customer_id`, `order_date`, and `product_name`. I grouped by customer and sorted by order date. The goal was to determine which of these window functions would be most appropriate for identifying the first item each customer purchased.

```sql
SELECT CUSTOMER_ID,
	ORDER_DATE,
	PRODUCT_NAME,
	ROW_NUMBER() OVER (PARTITION BY CUSTOMER_ID ORDER BY ORDER_DATE),
	RANK() OVER (PARTITION BY CUSTOMER_ID ORDER BY ORDER_DATE),
	DENSE_RANK() OVER (PARTITION BY CUSTOMER_ID ORDER BY ORDER_DATE)
FROM SALES S
JOIN MENU M ON M.PRODUCT_ID = S.PRODUCT_ID
ORDER BY CUSTOMER_ID, ORDER_DATE;
```

#### Output
![stage_1](https://github.com/user-attachments/assets/bd77a5fc-b0ef-41f6-ba30-3e6932658b11)

**Stage 2:**  
If I choose `ROW_NUMBER`, it will return only the first product purchased on the same date. However, since there's no time information, we don't know which item was purchased first on that date. Therefore, I chose the `RANK` function, which returns all items purchased on the same date, as a better option. The `DENSE_RANK` function would give the same result in this context.

```sql
WITH table_1 AS (
SELECT CUSTOMER_ID,
	ORDER_DATE,
	PRODUCT_NAME,
	RANK() OVER (PARTITION BY CUSTOMER_ID ORDER BY ORDER_DATE) RN
FROM SALES S
JOIN MENU M ON M.PRODUCT_ID = S.PRODUCT_ID
)
SELECT CUSTOMER_ID,
	PRODUCT_NAME
FROM table_1
WHERE RN = 1;
```

#### Output
![image](https://github.com/user-attachments/assets/db07dbe9-6239-4d17-8c92-abbf5bc32805)

**Stage 3:**  
Although this output is correct, it's not ideal to have duplicate rows for the same customer. To resolve this, I used the `DISTINCT` keyword.

```sql
WITH table_1 AS (
SELECT CUSTOMER_ID,
	ORDER_DATE,
	PRODUCT_NAME,
	RANK() OVER (PARTITION BY CUSTOMER_ID ORDER BY ORDER_DATE) RN
FROM SALES S
JOIN MENU M ON M.PRODUCT_ID = S.PRODUCT_ID
)
SELECT DISTINCT CUSTOMER_ID,
	PRODUCT_NAME
FROM table_1
WHERE RN = 1;
```

#### Output
![image](https://github.com/user-attachments/assets/3ad523d3-5220-45af-9e73-ef6e308cd718)

**Conclusion**  
Customer A ordered sushi and curry on their first visit. We don't know which was ordered first due to the lack of time information, but we know both were ordered on the same day. Customer B's first order was curry, and Customer C's was ramen.


### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

**Explanation**  
**Stage 1:**  
First, we need to determine how many of each product was sold.

```sql
SELECT PRODUCT_NAME,
	COUNT(S.PRODUCT_ID)
FROM SALES S
JOIN MENU M ON M.PRODUCT_ID = S.PRODUCT_ID
GROUP BY 1;
```

#### Output
![image](https://github.com/user-attachments/assets/35c4b3eb-75ce-4d5b-9ca2-b61ef62d036c)

**Stage 2:**  
Next, we sort the products by the number of sales in descending order and select the top product using the `LIMIT` command.

```sql
SELECT PRODUCT_NAME,
	COUNT(S.PRODUCT_ID)
FROM SALES S
JOIN MENU M ON M.PRODUCT_ID = S.PRODUCT_ID
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

#### Output
![image](https://github.com/user-attachments/assets/61fc2058-7804-46d7-802d-4cb8bae23644)

**Conclusion**  
Ramen is the most ordered item on the menu. It's crucial to ensure that the ingredients required for ramen are always in stock to meet customer demand.

### 5. Which item was the most popular for each customer?

**Explanation**  
**Stage 1:**  
First, we calculate how many of each product each customer bought.

```sql
SELECT CUSTOMER_ID,
	PRODUCT_NAME,
	COUNT(S.PRODUCT_ID) ORDER_COUNT
FROM SALES S
JOIN MENU M ON M.PRODUCT_ID = S.PRODUCT_ID
GROUP BY 1,2
ORDER BY CUSTOMER_ID, 3 DESC;
```

#### Output
![image](https://github.com/user-attachments/assets/0dbb859b-1708-4cf5-88ab-6b541c730d66)

#### Stage_2


In this stage, we use the RANK() function to rank the products for each customer based on how many times they ordered them. The product with the highest rank is the one they ordered the most. We chose the RANK() function because it allows us to capture all products that have the same order count. This way, if a customer ordered multiple products the same number of times, all of those products will be considered equally popular.

Next, we turn this query into a Common Table Expression (CTE) and then retrieve the most frequently ordered product(s) for each customer.

```sql
with table_1 as(
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
from table_1
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

#### Stage_2

Since the question says `after the customer becomes a member`, I decided to use join instead of left join. 
I also added a condition. I add the condition that order_date is greater than or equal to join_date. The reason I put equal is that the customer may have ordered on the day he became a member. In other words, he may have been a member and then ordered on the same day. 

```sql
select * 
from sales s
join menu m on m.product_id = s.product_id
join members mem on s.customer_id = mem.customer_id
where order_date >= join_date
order by 1
```
![image](https://github.com/user-attachments/assets/01a99d79-ded2-4e6b-a0c0-3fd2034a7252)

#### Stage_3

In the last stage, I added row_number by grouping by customer and sorting by date. Then I turned this query into a cte and called customer_id and product_name from those with row_number value 1.
```sql
with table_1 as(
select s.customer_id,
	order_date,
	product_name,
	row_number() over(partition by s.customer_id order by order_date) rn
from sales s
join menu m on m.product_id = s.product_id
join members mem on s.customer_id = mem.customer_id
where order_date > join_date
)
select customer_id,
		product_name
from table_1
where rn = 1
```
![image](https://github.com/user-attachments/assets/906ca43e-fe58-4781-99b1-1dc323c6dc2e)

-- 7. Which item was purchased just before the customer became a member?

#### Stage_1
Öncelikle rank() fonksiyonu kullanarak müşterilerin üye olmadan önce sipariş ettikleri ürünleri tarih sırasına göre azalacak şekilde sıraladım. Burada rank fonksiyonu kullanmamın sebebi A müşterisinin aynı tarihte ürün sipariş etmesinden dolayı (dense_rank() fonksiyonu da kullanılabilir di).
``` sql
select s.customer_id,
		order_date,
		product_name,
		rank() over(partition by s.customer_id order by order_date desc) rn
from sales s
join menu m on m.product_id = s.product_id
join members mem on s.customer_id = mem.customer_id
where order_date < join_date
```
![image](https://github.com/user-attachments/assets/3da6543e-4a73-4431-8789-f51b983a3ccd)

#### Stage_2

Bu aşamada yukarıdaki tabloyu bir CTE haline getiriyorum ve sıra numarası 1 olan customer_id ve product_name alıyorum. Böylece üye olmadan önceki en son siparişlerini bulabiliyorum.

```sql
with table_1 as(
select s.customer_id,
		order_date,
		product_name,
		rank() over(partition by s.customer_id order by order_date desc) rn
from sales s
join menu m on m.product_id = s.product_id
join members mem on s.customer_id = mem.customer_id
where order_date < join_date
)
select customer_id,
	product_name
from table_1
where rn = 1
```
![image](https://github.com/user-attachments/assets/5680fb55-449e-4c48-9f7d-953e9012a33e)

### Conclusion
A mmüşterisi üye olmadan hemen önce sushi ve curry siparişini aynı gün vermiş. B müşterisi ise sushi sipariş etmiş.

-- 8. What is the total items and amount spent for each member before they became a member?

Müşteri bazında gruplayarak sipariş ettikleri ürünleri saydırdım ve ödedikleri miktarı topladım

```sql
select s.customer_id,
		count (s.product_id),
		sum(price)
from sales s
join menu m on m.product_id = s.product_id
join members mem on mem.customer_id = s.customer_id
where join_date>order_date
group by 1
order by 1
```
#### Output
![image](https://github.com/user-attachments/assets/60e2cdbe-d545-4e0b-90a9-da1e3b8a15d8)

### Conclusion
A müşterisi üye olmadan önce 2 ürün almış ve 25$ lık harcama yapmış. B müşterisi ise 3 ürün almış ve 40$ lık harcama yapmış.

9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

#### Stage_1

Öncelikle müşterilerin her bir aldıkları ürünü onun fiyatını ve bu üründen kaç puan kazandıklarını gösteren bir tablo oluşturdum.

```sql
SELECT CUSTOMER_ID,
		S.PRODUCT_ID,
		PRODUCT_NAME,
		PRICE,
		CASE 
			WHEN PRODUCT_NAME = 'SUSHI' THEN PRICE * 2 * 10
			ELSE PRICE * 10
			END POINTS
FROM SALES S
JOIN MENU M ON M.PRODUCT_ID = S.PRODUCT_ID
ORDER BY 1
```
#### Output
![image](https://github.com/user-attachments/assets/b014e9f4-bc8b-42c1-acc6-b44e34e3895e)

#### Stage_2

Bu aşamada yukarıdaki sorguyu bir CTE haline getirerek her bir müşterinin kaç puan aldıklarını gösterdim.

```sql
with tablo as(
select customer_id,
		s.product_id,
		product_name,
		price,
		case 
			when product_name = 'sushi' then price * 2* 10
			else price * 10
			end points
from sales s
full outer join menu m on m.product_id = s.product_id
)
select customer_id,
		sum(points)
from tablo
group by 1
order by 2 desc
```

#### Output

![image](https://github.com/user-attachments/assets/f5eaf71c-70b4-46bd-a6c6-f32b855e1add)

### Conclusion

10. n the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

#### Explanation 
#### Stage_1
ilk haftayı bulabilmek için end date adında bir sutunu join_date sütununa 6 ekleyerek buldum. Arından case when yapısı ile eğer order_date join_date ile end_date arasında ise iki kat puan hesapladım. Sushi için de iki kat puan hesaplamaya devam ettim. Aksi halde normal puan hesapladım. Son olarak order_date i ocak ayı sonu olarak belirledim.

```sql
Select s.customer_id,
		join_date start_date,
		join_date + 6 end_date,
		order_date,		
		product_name,
		price,
		case 
			when order_date BETWEEN join_date and join_date + 6 then price * 2*10
			when product_name = 'sushi' then price * 2* 10
			else price*10
			end as points
from sales s
join members mem on mem.customer_id = s.customer_id
join menu m on m.product_id = s.product_id
WHERE order_date <= '2021-01-31'
```
#### Stage_2
Bu aşamada müşteri bazında gruplayarak toplam puanları buldum.

```sql
with table_1 as(
Select s.customer_id,
		join_date start_date,
		join_date + 6 end_date,
		order_date,		
		product_name,
		price,
		case 
			when order_date BETWEEN join_date and join_date + 6 then price * 2*10
			when product_name = 'sushi' then price * 2* 10
			else price*10
			end as points
from sales s
join members mem on mem.customer_id = s.customer_id
join menu m on m.product_id = s.product_id
WHERE order_date <= '2021-01-31'
)
SELECT customer_id,
		sum(points)
from table_1
group by 1
```
### Conclusion

En çok puan toplayan müşterimiz A müşterisinin 1370 puanı var, onu B müşterisi 820 puan ile takip ediyor.

# Bonus Questions

1. Join All The Things

#### Explanation 
Aşağıdaki birleştirilmiş tablo ile hızlı bir şekilde içgörü elde edebiliriz. Member sütundaki "N" harfi alışveriş yapıldığı tarihte o müşterinin henüz üyemiz olmadığını gösteriyor. "Y" harfi ise o müşterinin alışveriş yapıldığı tarihte müşterimiz olduğunu gösteriyor.

#### Stage_1
members sutünu ile left join yaptım. Çünkü eğer join işlemi yapsaydım C müşterisini göremezdim. Ayrıca C müşterisinin join_date değeri olmadığı için order_date ile join_date arasında bir kıyaslama yapamıyor. bundan dolayı member sutünuna "N" değeri geliyor.

```sql
SELECT s.customer_id,
		order_date,
		product_name,
		price, 
		join_date,
	case 
		when order_date >= join_date then 'Y' else 'N' end as member

from sales s
join menu m on s.product_id = m.product_id
LEFT join members mem on mem.customer_id = s.customer_id
order by 1,2
```
![image](https://github.com/user-attachments/assets/0ccde785-ba8d-4951-a348-fc9f2ad01f13)

#### Stage_2
Yukarıdaki yaklaşım bize her ne kadar doğru sonuçlar veriyor olsa da join_date in null olduğu değerler için ayrı bir koşul konmasının daha doğru olduğunu düşünüyorum. Böylece eğer else kısmındaki değerler "Y" bile olsa bize doğru sonuç verir. join_date sütununu çıkarıyorum ve istenen tabloyu elde ediyorum.

```sql
SELECT s.customer_id,
		order_date,
		product_name,
		price, 
		case 
			when join_date is null then 'N' 
			when order_date >= join_date then 'Y' else 'N' end as member

from sales s
join menu m on s.product_id = m.product_id
LEFT join members mem on mem.customer_id = s.customer_id
order by 1,2
```
![image](https://github.com/user-attachments/assets/b2181d57-8c93-4b65-8bfd-533e2b1066a4)

2. Rank All The Things

#### Explanation 
Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

Bu sorunun çözümünde case when yapısı ile eğer member değil ise null değerler verdiriyorum. eğer üye ise rank ile sıralama yapıyorum. Burada en önemli nokta sıralama yaparken hem customer_id için hem de member için partition by kullanmak gerektiğidir. aksi halde null değerler için de bir sıralama olur ve istediğimiz sıralamaya ulaşamayız.

```sql
with table_1 as(
SELECT s.customer_id,
		order_date,
		product_name,
		price, 
		case 
			when join_date is null then 'N' 
			when order_date >= join_date then 'Y' else 'N' end as member

from sales s
join menu m on s.product_id = m.product_id
LEFT join members mem on mem.customer_id = s.customer_id
order by 1,2
	)
SELECT *,
		case 
			WHEN member = 'N' then NULL
			else
				rank() OVER(PARTITION by customer_id, member order by order_date) end as ranking
from table_1
```











