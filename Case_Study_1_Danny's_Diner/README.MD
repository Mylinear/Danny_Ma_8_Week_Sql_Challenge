# 8-Week SQL Challenge: Danny's Diner

## Case Study #1: Danny's Diner

![image](https://8weeksqlchallenge.com/images/case-study-designs/1.png)

### Overview

This case study is part of the [8-Week SQL Challenge](https://8weeksqlchallenge.com) created by Danny Ma. The challenge focuses on analyzing customer behavior at Danny's Diner, a small fictional restaurant. The goal is to help the owner, Danny, understand customer ordering patterns, optimize menu offerings, and improve overall customer satisfaction through data-driven decisions.

---

### 📝 **Table of Contents**
1. [Introduction](#introduction)
2. [Problem Statement](#problem-statement)
3. [Dataset](#dataset)
4. [Entity Relationship Diagram (ERD)](#entity-relationship-diagram-erd)
5. [SQL Queries](#sql-queries)
6. [Results](#results)
7. [Conclusion](#conclusion)

---

## Introduction

Danny's Diner is a small restaurant, and the owner, Danny, wants to analyze customer behavior, particularly their ordering patterns. By understanding the customers' preferences, Danny aims to introduce new menu items, optimize the inventory, and improve customer satisfaction.

This case study involves querying the restaurant's sales and customer data to derive insights into customer behavior and order patterns.

## Problem Statement

The primary objectives of this case study are:
1. Identify the total sales for each customer.
2. Determine which menu items each customer ordered.
3. Analyze the frequency of visits by each customer.
4. Discover any trends in customer orders over time.

## Dataset

The dataset consists of three tables:
- **sales**
- **menu**
- **members**

### sales Table
| column_name | data_type |
|-------------|-----------|
| customer_id | VARCHAR   |
| order_date  | DATE      |
| product_id  | INTEGER   |

### menu Table
| column_name | data_type |
|-------------|-----------|
| product_id | INTEGER   |
| product_name | VARCHAR   |
| price       | INTEGER   |

### members Table
| column_name | data_type |
|-------------|-----------|
| customer_id | VARCHAR   |
| join_date   | DATE      |

---

## Entity Relationship Diagram (ERD)

![ERD](path/to/your/ERD/image.png)

The ERD shows the relationships between the three tables: `sales`, `menu`, and `members`.

---

## SQL Queries

Below are the SQL queries used to solve the case study problems. You can replace the placeholder with your own SQL code.

### 1. Total Sales per Customer
```sql
-- SQL Query to calculate total sales for each customer

-- Add your solution here
