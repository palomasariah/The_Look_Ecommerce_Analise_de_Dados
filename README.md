# Análise de Dados do E-commerce The Look


### Objetivo do projeto:
Este projeto visa analisar o comportamento de compras dos clientes do e-commerce The Look, identificar tendências de vendas e fornecer insights para a tomada de decisões estratégicas.


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

#### **Query 1:** Qual é a receita total gerada nos últimos 12 meses?
*Solução:*
1. Subquery:
  - Seleciona a soma dos preços de venda (sale_price) de todos os itens de pedido da tabela order_items.
  - Filtra os itens de pedido onde o status é ‘Complete’ e a data de criação (created_at) está entre ‘2023-08-01’ e ‘2024-08-01’.
  - Calcula a soma total desses preços de venda e a nomeia como total_ano.
2. Query Externa:
  - Soma os valores de total_ano obtidos da subquery interna.
  - Arredonda o resultado para duas casas decimais.
  - Nomeia o resultado final como receita_ano.

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


#### **Query 2:** Qual é a média mensal de vendas?
*Solução:*
1. Subquery:
  - Extrai o ano e o mês da data de criação (created_at) de cada item de pedido.
  - Calcula a soma dos preços de venda (sale_price) para cada mês e ano, arredondando o valor para três casas decimais, e nomeia como vendas_mes.
  - Agrupa os resultados por mês e ano.
  - Ordena os resultados por mês e ano.
2. Query Externa:
  - Calcula a média dos valores mensais de vendas (vendas_mes) obtidos da subquery interna.
  - Arredonda o resultado para duas casas decimais.
  - Nomeia o resultado final como media_vendas_mes.
  
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
  
#### **Query 3:** Quais são os meses com o maior e o menor faturamento?
*Solução:*
1. Subquery:
  - Extrai o mês da data de criação (created_at) de cada item de pedido.
  - Calcula a soma dos preços de venda (sale_price) para cada mês, arredondando o valor para três casas decimais, e nomeia como vendas_mes.
  - Agrupa os resultados por mês.
2. Query Externa:
  - Agrupa os resultados da subquery interna por mês.
  - Calcula o valor máximo das vendas mensais (vendas_mes) para cada mês.
  - Arredonda o valor máximo para duas casas decimais.
  - Nomeia o resultado final como maior_venda.
  - Ordena os resultados pela maior venda em ordem decrescente

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
  

#### **Query 4:** Quais são os meses com o maior e o menor volume de vendas?
*Solução:*
  - Extrai o mês da data de criação (created_at) de cada pedido.
  - Conta o número de pedidos (order_id) para cada mês e nomeia como volume_vendas.
  - Filtra os pedidos onde o status é ‘Complete’ e a data de criação está entre ‘2023-01-01’ e ‘2023-12-31’.
  - Agrupa os resultados por mês.
  - Ordena os resultados pelo volume de vendas (volume_vendas) e, em seguida, pelo mês (mes).

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

#### **Query 5:** Quantos clientes únicos realizaram compras nos últimos 12 meses?
*Solução:*
  - Conta o número de clientes distintos (u.id) e nomeia como total_clientes.
  - Realiza um JOIN entre as tabelas orders (pedidos) e users (usuários) com base no campo user_id.
  - Filtra os pedidos onde o status é ‘Complete’ e a data de criação está entre ‘2023-08-01’ e ‘2024-08-01’.

```sql
SELECT COUNT(DISTINCT u.id) AS total_clientes

FROM `bigquery-public-data.thelook_ecommerce.orders` AS o
JOIN `bigquery-public-data.thelook_ecommerce.users` AS u
  ON o.user_id = u.id
 
WHERE o.status = 'Complete'
  AND o.created_at BETWEEN '2023-08-01' AND '2024-08-01';
```

#### **Query 6:** Qual é o valor médio gasto por cliente?
*Solução:*
1. Subquery:
  - Seleciona o user_id de cada cliente.
  - Calcula a soma dos preços de venda (sale_price) para cada cliente e nomeia como gasto_cliente.
  - Filtra os itens de pedido onde o status é ‘Complete’.
  - Agrupa os resultados por user_id.
2. Query Externa:
  - Calcula a média dos gastos dos clientes (gasto_cliente) obtidos da subquery interna.
  - Arredonda o resultado para duas casas decimais.
  - Nomeia o resultado final como gasto_medio_cliente.

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

#### **Query 7:** Qual é a taxa de retorno de clientes (quantos clientes voltaram para fazer uma segunda compra)?
*Solução:*
1. Subquery:
  - Seleciona o user_id de cada cliente.
  - Filtra os clientes que têm mais de uma data de criação distinta (created_at) para pedidos com status ‘Complete’.
  - Agrupa os resultados por user_id.
Filtra os grupos que têm mais de uma data distinta de criação de pedidos.
2. Query Externa:
  - Conta o número de clientes distintos (user_id) que atendem aos critérios da subquery interna.
  - Nomeia o resultado final como total_clientes.

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

#### **Query 8:** Quais são os produtos mais vendidos?
*Solução:*
  - Conta o número total de vendas (COUNT(*)) e nomeia como vendas.
  - Seleciona o product_id e o name do produto.
  - Realiza um JOIN entre as tabelas order_items (itens de pedido) e products (produtos) com base no campo product_id.
  - Filtra os itens de pedido onde o status é ‘Complete’.
  - Agrupa os resultados por product_id e name.
  - Ordena os resultados pelo número de vendas (vendas) em ordem decrescente.
  - Limita os resultados aos 5 produtos mais vendidos.
  
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

#### **Query 9:** Quais produtos geraram a maior receita?
*Solução:*
- Calcula a soma dos preços de venda (sale_price) e arredonda o valor para duas casas decimais, nomeando como valor_vendido.
- Seleciona o product_id e o name do produto.
- Realiza um JOIN entre as tabelas order_items (itens de pedido) e products (produtos) com base no campo product_id.
- Filtra os itens de pedido onde o status é ‘Complete’.
- Agrupa os resultados por product_id e name.
- Ordena os resultados pelo valor vendido (valor_vendido) em ordem decrescente.
- Limita os resultados aos 5 produtos com maior valor vendido.
  
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

#### **Query 10:** Qual campanha teve o melhor desempenho em termos de aumento de vendas?
*Solução:*
  - Seleciona o tipo de evento (event_type) e a fonte de tráfego (traffic_source).
  - Conta o número de eventos (COUNT(*)) e nomeia como vendas.
  - Filtra os eventos onde o tipo de evento é ‘purchase’ (compra).
  - Agrupa os resultados por traffic_source e event_type.
  - Ordena os resultados pelo número de vendas (vendas) em ordem decrescente.

```sql
SELECT event_type, traffic_source, COUNT(*) as vendas

FROM bigquery-public-data.thelook_ecommerce.events

WHERE event_type = 'purchase'

GROUP BY traffic_source, event_type
ORDER BY 3 DESC;
```


### Análise Geográfica:

#### **Query 11:** Quais são os países com maior número de compras?
*Solução:*
  - Seleciona o país (country) dos usuários.
  - Conta o número de pedidos (COUNT(*)) e nomeia como vendas.
  - Realiza um JOIN entre as tabelas users (usuários) e orders (pedidos) com base no campo user_id.
  - Filtra os pedidos onde o status é ‘Complete’.
  - Agrupa os resultados por país (country).
  - Ordena os resultados pelo número de vendas (vendas) em ordem decrescente.

```sql
SELECT u.country, COUNT(*) as vendas

FROM bigquery-public-data.thelook_ecommerce.users AS u
JOIN bigquery-public-data.thelook_ecommerce.orders AS o
  ON u.id = o.user_id

WHERE o.status = 'Complete'

GROUP BY u.country
ORDER BY 2 DESC;
```

#### **Query 12:** Qual é a receita gerada por país?
*Solução:*
  - Seleciona o país (country) dos usuários.
  - Calcula a soma dos preços de venda (SUM(o.sale_price)) e arredonda para duas casas decimais, nomeando como valor_vendido.
  - Realiza um JOIN entre as tabelas users (usuários) e order_items (itens de pedido) com base no campo user_id.
  - Filtra os pedidos onde o status é ‘Complete’.
  - Filtra os pedidos criados entre ‘2023-08-01’ e ‘2024-08-01’.
  - Agrupa os resultados por país (country).
  - Ordena os resultados pelo valor vendido (valor_vendido) em ordem decrescente.
  
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


#### **Query 13:** Qual é a distribuição de vendas por categoria de produto?
*Solução:*
  - Seleciona a categoria (category) dos produtos.
  - Conta o número de pedidos (COUNT(*)) e nomeia como vendas.
  - Realiza um JOIN entre as tabelas products (produtos) e order_items (itens de pedido) com base no campo product_id.
  - Filtra os pedidos onde o status é ‘Complete’.
  - Agrupa os resultados por categoria (category).
  - Ordena os resultados pelo número de vendas (vendas) em ordem decrescente.
  
```sql
SELECT p.category, COUNT(*) as vendas

FROM bigquery-public-data.thelook_ecommerce.products AS p
JOIN bigquery-public-data.thelook_ecommerce.order_items AS o
  ON p.id = o.product_id

WHERE o.status = 'Complete'

GROUP BY p.category
ORDER BY 2 DESC;
```

#### **Query 14:** Quais categorias de produtos têm o melhor desempenho em termos de receita?
*Solução:*
  - Seleciona a categoria (category) dos produtos e renomeia como categoria.
  - Calcula a soma dos preços de venda (SUM(o.sale_price)) e arredonda para três casas decimais, nomeando como valor_vendas.
  - Realiza um JOIN entre as tabelas products (produtos) e order_items (itens de pedido) com base no campo product_id.
  - Filtra os pedidos onde o status é ‘Complete’.
  - Agrupa os resultados por categoria (category).
  - Ordena os resultados pelo valor das vendas (valor_vendas) em ordem decrescente.
  - Limita os resultados aos 5 primeiros registros.

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


