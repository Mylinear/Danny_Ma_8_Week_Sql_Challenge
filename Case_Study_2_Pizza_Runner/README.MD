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
### Data Cleaning for `runner_orders` table

Tabloyu incelediğimizde `pickup_time` sütununda string halinde null yazdığını görüyoruz. `distance` sütununda aynı şekilde string halinde null yazılmış. ayrıca bu sütun sayısal veri tipinde olması gerekiyor ancak sütunda bir çok yerde `km` ifadelerinden geçiyor. Bizim bu harflerden oluşan ifadeleri temizlememiz gerekiyor.  Son olarak da sütunun veri tipini float tipine dönüştürmemiz gerekiyor.`cancellatinon` sütununda da string halinde null değerler var. Onların da `NULL` haline dönüştürülmesi gerekiyor.

```sql
UPDATE runner_orders
SET pickup_time = CASE
    when pickup_time = 'null' then NULL 
			else pickup_time 
END,
    distance = CASE
    when distance = 'null' then NULL
			when distance like '%km' then trim(distance,'km') else distance
END,
    duration = CASE
    when duration like 'null' then NULL
				when duration like '%mins' then trim(duration,'mins')
				when duration like '%minute' then trim(duration,'minute')
				when duration like '%minutes' then trim(duration,'minutes') else duration
END,
	cancellation = CASE
	when cancellation = 'null' or cancellation ='' then NULL else cancellation END;

```

### Veri tiplerinin değiştirilmesi
