1. What are the standard ingredients for each pizza?

Bu soruda öncelikle `pizza_recipes` tablosunu yeniden düzenlemem gerekiyor. Toppings sütunundaki her bir değeri ayrı bir satırda göstermem gerekiyor. Bunu iki yolla yapabilirim.

tablonun orjinal hali bu şekilde 

![image](https://github.com/user-attachments/assets/78c56123-e642-4f56-a254-aeaa22afb5d2)

1. yol: Regex kullanarak virgülle ayrılmış değerlerin her birini ayrı bir satırda yazdırabilirm.

```sql
SELECT
  pizza_id,
  REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER AS top_id
FROM pizza_recipes
```
bu kodda integer a cast etmemin sebebi regex bize string şeklinde çıktı verir. Bu yeni tabloyu başka bir tablo ile joinlemek istersem veri tipi uyuşmadığı için hata mesajı alırım.
Output:

![image](https://github.com/user-attachments/assets/75e68d29-1502-44ba-b022-f03e9200ced5)

Böylece pizza içerisindeki her bir ürünü ayrı bir satırda gösterebiliyoruz.

2. Yol: unnest ve string_to_array  fonksiyonları kullanarak aynı tabloyu elde edebiliriz.

```sql
SELECT 
  pizza_id,
  TRIM(both ' ' FROM unnest(string_to_array(toppings, ',')))::int AS topping_id
FROM pizza_recipes c
```
yine ayni şekilde çıkan sonucu int cast ediyorum. Böylece başka tablolar ile joninleme işlemi yaptığımda hata almayacağım.

output:

![image](https://github.com/user-attachments/assets/b53979d2-028c-4fcf-830b-a2d73cc6791c)


Stage 2
Bu oluşturmuş olduğum tabloyu `pizza_names` ve `pizza_toppings` tabloları ile joinliyorum. Böylece pizza isimlerini ve topping isimlerini birlikte görebiliyoruz.

```sql
WITH toppings_cte AS (
SELECT
  pizza_id,
  REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER AS top_id
FROM pizza_recipes
)
SELECT pizza_name,
		tcte.pizza_id,
		top_id,
		topping_name
from toppings_cte tcte
join pizza_names pn on pn.pizza_id = tcte.pizza_id
join pizza_toppings pt on pt.topping_id = tcte.top_id
```

Stage 3

Son aşamada ise pizza isimlerine göre gruplayıp topping isimlerini string_agg fonksiyonu ile birleştiriyorum.

```sql
WITH toppings_cte AS (
SELECT
  pizza_id,
  REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER AS top_id
FROM pizza_recipes
),
joined_table as (
SELECT pizza_name,
		tcte.pizza_id,
		top_id,
		topping_name
from toppings_cte tcte
join pizza_names pn on pn.pizza_id = tcte.pizza_id
join pizza_toppings pt on pt.topping_id = tcte.top_id
	)
SELECT
	pizza_name,
	STRING_AGG(topping_name, ', ') AS toppings_list
from joined_table
group by pizza_name
```
output

![image](https://github.com/user-attachments/assets/60eb45be-5bac-4fc2-b6ba-d02508f05017)

2. 
