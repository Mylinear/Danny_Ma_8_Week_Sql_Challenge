# Data Cleaning

## Data cleaning for customer orders table

`customer_orders` tablosunda exclusions ve extras sütunlarında hem boş stringler var hem de string halinde null yazan değerler var. Amacımız bu string halindeki null değerleri ve boş string ifadeleri `NULL` değerlere dönüştürmek. Bunu yapabilmek için case when yapısı kullanıyoruz.

```sql
select order_id,
		customer_id,
		pizza_id,
		case when exclusions = 'null' or exclusions = ''  then NULL else exclusions END as exclusions,
		case when extras = 'null' or extras = ''  then NULL else extras END as extras,
		order_time
FROM customer_orders;
```
output 
![image](https://github.com/user-attachments/assets/c5775798-a74d-4bc9-85c7-bf79f6b170f9)

istediğim tabloya ulaştığıma emin olduktan sonra tablomu güncelliyorum. Bunu da update ve set komutları ile gerçekleştiriyorum. 

```sql
UPDATE customer_orders
SET exclusions = CASE
    when exclusions = 'null' or exclusions = ''  then NULL else exclusions
END; 

UPDATE customer_orders
SET extras = CASE
    when extras = 'null' or extras = ''  then NULL else extras
END; 
```
