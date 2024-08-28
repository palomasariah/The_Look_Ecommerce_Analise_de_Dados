# Análise de Dados do E-commerce The Look


### Objetivo do projeto:
Este projeto visa analisar o comportamento de compras dos clientes do e-commerce The Look, identificar tendências de vendas e fornecer insights acionáveis para a tomada de decisões estratégicas.


### Fonte dos Dados
The Look é uma loja de roupas online desenvolvida pela equipe do Google Looker. Esses dados de e-commerce estão hospedados no Google Big Query.


### Estrutura do Dataset:
O dataset completo é composto por 7 tabelas. Neste projeto foram usadas as 5 tabelas citadas abaixo:
  - Order_items
  - Orders
  - Products
  - Users
  - Events
    
Para ver o dataset, [clique aqui](https://console.cloud.google.com/bigquery?ws=!1m4!1m3!3m2!1sbigquery-public-data!2sthelook_ecommerce).

## Perguntas a serem Respondidas


### Desempenho de Vendas:

**Query 1:** Qual é a receita total gerada nos últimos 12 meses?
  
  ```sql
  SELECT ROUND(SUM(total_ano), 2) AS receita_ano
    FROM (
    SELECT 
      SUM(sale_price) AS total_ano
    FROM 
      `bigquery-public-data.thelook_ecommerce.order_items`
    WHERE 
      status = 'Complete' 
      AND created_at BETWEEN '2023-08-01' AND '2024-08-01'

    );
  ```


**Query 2:** Qual é a média mensal de vendas?
  
  ```sql
  SELECT ROUND (avg(vendas_mes),2) as media_vendas_mes
  
    FROM (
  
  SELECT
    EXTRACT(YEAR FROM created_at) AS ano,
    EXTRACT(MONTH FROM created_at) AS mes,
    ROUND(SUM(sale_price),3) as vendas_mes
  
  
  FROM `bigquery-public-data.thelook_ecommerce.order_items`
  WHERE status = 'Complete' and created_at BETWEEN '2023-01-01' AND '2023-12-31'
  GROUP BY mes, ano
  ORDER BY mes, ano

    );
  ```
  
**Query 3:** Quais são os meses com o maior e o menor faturamento?

```sql
SELECT ROUND (max(vendas_mes),2) as maior_venda,
  mes
       
    FROM (
  
  SELECT
    EXTRACT(MONTH FROM created_at) AS mes,
    ROUND(SUM(sale_price),3) as vendas_mes
  
  FROM `bigquery-public-data.thelook_ecommerce.order_items`
  WHERE status = 'Complete' and created_at BETWEEN '2023-01-01' AND '2023-12-31'
  GROUP BY mes
  
  )

GROUP BY mes
ORDER BY maior_venda DESC;
```
  

**Query 4:** Quais são os meses com o maior e o menor volume de vendas?

```sql
SELECT
  EXTRACT(MONTH FROM created_at) AS mes,
  COUNT(order_id) as volume_vendas

FROM `bigquery-public-data.thelook_ecommerce.orders`
WHERE status = 'Complete' and created_at BETWEEN '2023-01-01' AND '2023-12-31'

GROUP BY mes
ORDER BY volume_vendas, mes;
```


### Análise de Clientes:

**Query 5:** Quantos clientes únicos realizaram compras nos últimos 12 meses?

```sql
SELECT COUNT(DISTINCT u.id) AS total_clientes

FROM `bigquery-public-data.thelook_ecommerce.orders` AS o
JOIN `bigquery-public-data.thelook_ecommerce.users` AS u
  ON o.user_id = u.id
 
WHERE o.status = 'Complete'
  AND o.created_at BETWEEN '2023-08-01' AND '2024-08-01';
```

**Query 6:** Qual é o valor médio gasto por cliente?


```sql
SELECT ROUND(AVG(gasto_cliente), 2) AS gasto_medio_cliente
FROM (
  SELECT 
    user_id, 
    SUM(sale_price) AS gasto_cliente
  FROM 
    `bigquery-public-data.thelook_ecommerce.order_items`
  WHERE 
    status = 'Complete'
  GROUP BY 
    user_id
);
```

**Query 7:** Qual é a taxa de retorno de clientes (quantos clientes voltaram para fazer uma segunda compra)?

```sql
SELECT COUNT(DISTINCT user_id) AS total_clientes
FROM bigquery-public-data.thelook_ecommerce.orders
WHERE user_id IN (
    SELECT user_id
    FROM bigquery-public-data.thelook_ecommerce.orders
    WHERE status = 'Complete'
    GROUP BY user_id
    HAVING COUNT(DISTINCT created_at) > 1
);
```


### Análise de Produtos:

**Query 8:** Quais são os produtos mais vendidos?

```sql
SELECT COUNT(*) AS vendas,
  o.product_id,
  p.name
FROM bigquery-public-data.thelook_ecommerce.order_items AS o
JOIN bigquery-public-data.thelook_ecommerce.products AS p
  ON o.product_id = p.id
WHERE o.status = 'Complete'
GROUP BY o.product_id, p.name
ORDER BY vendas DESC
LIMIT 5;
```

**Query 9:** Quais produtos geraram a maior receita?

```sql
SELECT ROUND(SUM(o.sale_price), 2) AS valor_vendido,
  o.product_id,
  p.name
FROM bigquery-public-data.thelook_ecommerce.order_items AS o
JOIN bigquery-public-data.thelook_ecommerce.products AS p
  ON o.product_id = p.id
WHERE o.status = 'Complete'
GROUP BY o.product_id, p.name
ORDER BY valor_vendido DESC
LIMIT 5;
```


### Análise de Campanhas:

**Query 10:** Qual campanha teve o melhor desempenho em termos de aumento de vendas?

```sql
SELECT event_type, traffic_source, COUNT(*) as vendas

FROM bigquery-public-data.thelook_ecommerce.events

WHERE event_type = 'purchase'

GROUP BY traffic_source, event_type
ORDER BY 3 DESC;
```


### Análise Geográfica:

**Query 11:** Quais são os países com maior número de compras?

```sql
SELECT u.country, COUNT(*) as vendas

FROM bigquery-public-data.thelook_ecommerce.users AS u
JOIN bigquery-public-data.thelook_ecommerce.orders AS o
  ON u.id = o.user_id

WHERE o.status = 'Complete'

GROUP BY u.country
ORDER BY 2 DESC;
```

**Query 12:** Qual é a receita gerada por país?

```sql
SELECT u.country, ROUND(SUM(o.sale_price), 2) AS valor_vendido

FROM bigquery-public-data.thelook_ecommerce.order_items AS o
JOIN bigquery-public-data.thelook_ecommerce.users AS u
  ON o.user_id = u.id

WHERE status = 'Complete'
  AND o.created_at BETWEEN '2023-08-01' AND '2024-08-01'

GROUP BY u.country
ORDER BY 2 DESC;
```


### Análise de Categorias:


**Query 13:** Qual é a distribuição de vendas por categoria de produto?

```sql
SELECT p.category, COUNT(*) as vendas

FROM bigquery-public-data.thelook_ecommerce.products AS p
JOIN bigquery-public-data.thelook_ecommerce.order_items AS o
  ON p.id = o.product_id

WHERE o.status = 'Complete'

GROUP BY p.category
ORDER BY 2 DESC;
```

**Query 14:** Quais categorias de produtos têm o melhor desempenho em termos de receita?

```sql
SELECT
  p.category AS categoria,
  ROUND(SUM(o.sale_price),3) AS valor_vendas

FROM
bigquery-public-data.thelook_ecommerce.products AS p
JOIN bigquery-public-data.thelook_ecommerce.order_items AS o
  ON p.id = o.product_id

WHERE o.status = 'Complete'
GROUP BY p.category
ORDER BY valor_vendas DESC

LIMIT 5;
```


