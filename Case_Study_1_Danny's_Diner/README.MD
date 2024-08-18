# 8-Week SQL Challenge:Case Study #1: Danny's Diner

<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" width="500">

## The best solutions you can find on the internet are in this repo. I also shared the solution to this case on my YouTube channel (unfortunately only in Turkish).


[![Watch the video](https://img.youtube.com/vi/SfdS3WREZEo/maxresdefault.jpg)](https://youtu.be/SfdS3WREZEo)

## Case Study Questions
1. What is the total amount each customer spent at the restaurant?
### Explanation 
The menu table contains the price for each product. We would have to join the sales and menu tables then sum the prices for the orders made by each customer after grouping by the customer_id .

``` sql
SELECT CUSTOMER_ID,
	SUM(PRICE)
FROM SALES S
JOIN MENU M ON M.PRODUCT_ID = S.PRODUCT_ID
GROUP BY 1;
```
![Q1](https://github.com/user-attachments/assets/4a059577-db03-4e4f-b4f5-e9c5e74a5823)

