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



