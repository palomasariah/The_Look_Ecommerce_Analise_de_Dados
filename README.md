# Análise de Dados do E-commerce The Look


### Objetivo do projeto:
Este projeto visa analisar o comportamento de compras dos clientes do e-commerce The Look, identificar tendências de vendas e fornecer insights acionáveis para a tomada de decisões estratégicas.


### Fonte dos Dados
The Look é uma loja de roupas online desenvolvida pela equipe do Google Looker. Esses dados de e-commerce estão hospedados no Google Big Query.


### Estrutura do Dataset
- **Sales**: Contém informações sobre vendas, incluindo datas, produtos, quantidades e valores.
- **Customers**: Inclui dados dos clientes, como IDs, nomes e informações demográficas.
- **Products**: Detalha os produtos, incluindo IDs, nomes, categorias e preços.
- **Transactions**: Fornece informações sobre transações, como IDs de pedidos, datas e valores totais.
- **Campaigns**: Contém dados sobre campanhas promocionais, incluindo tipos, datas e impactos esperados.




## Perguntas a serem Respondidas


### Desempenho de Vendas

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
  
  )
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
ORDER BY maior_venda DESC
```
  

**Query 4:** Quais são os meses com o maior e o menor volume de vendas?

```sql
SELECT
  EXTRACT(MONTH FROM created_at) AS mes,
  COUNT(order_id) as volume_vendas

FROM `bigquery-public-data.thelook_ecommerce.orders`
WHERE status = 'Complete' and created_at BETWEEN '2023-01-01' AND '2023-12-31'

GROUP BY mes
ORDER BY volume_vendas, mes
```

### Desempenho de Vendas


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
)
```

