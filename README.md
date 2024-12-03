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

#### ⚪ **Query 1:** Qual é a receita total gerada nos últimos 12 meses?
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

#### *Resultado:*
receita_ano |
-- |
925096.45 |

#### *Insights:*
  - The Look Ecommerce apresentou um crescimento de aproximadamente 72.5% na receita em relação ao mesmo período no ano anterior.



#### ⚪ **Query 2:** Qual é a média mensal de vendas?
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

#### *Resultado:*
media_vendas_mes |
-- |
55055.04 |

#### *Insights:*
   - A média de vendas mensais para o e-commerce The Look é de $55,055.04.


  
#### ⚪ **Query 3:** Quais são os meses com o maior e o menor faturamento?
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

#### *Resultado:*
maior_venda |	mes
-- | --
76899.95	| 12
64801.61	| 10
60702.1 |	11
60316.39 |	7
58445.66	| 9
57771.37	| 8
55083.71	| 6
48748.75	| 5
47046.82	| 3
46005.18	| 4
45833.3 |	2
39005.59 |	1

#### *Insights:*
  - Os melhores 3 meses, em termos de vendas, foram os meses de dezembro, com $76,899.95, seguido por outubro ($64,801.61) e novembro ($60,702.10).
  - No ultimo trimestre do ano (outubro, novembro e dezembro) o faturamento é superior.
  - Janeiro é o mês com a menor venda, com $39,005.59.



#### ⚪ **Query 4:** Quais são os meses com o maior e o menor volume de vendas?
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

#### *Resultado:*
mes |	volume_vendas
-- | --
1	| 479
2 |	507
4 |	535
5	| 568
6	| 573
3	| 579
9 |	643
7	| 651
8	| 651
11	| 721
10	| 746
12	| 818

#### *Insights:*
  - Em conformidade com a análise anterior (referente ao faturamento), dezembro também teve o maior volume de vendas, com 818 unidades, e janeiro teve o menor volume de vendas, com 479 unidades.



### Análise de Clientes:

#### ⚪ **Query 5:** Quantos clientes únicos realizaram compras nos últimos 12 meses?
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

#### *Resultado:*
total_clientes |
-- |
9945 |

#### *Insights:*
  - Houve um crescimento de aproximadamente 68.05% no número de clientes em relação ao mesmo perído do ano anterior.


#### ⚪ **Query 6:** Qual é o valor médio gasto por cliente?
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

#### *Resultado:*
gasto_medio_cliente |
-- |
99.61 |

#### *Insights:*
  - Cada cliente gastou, em média, $99.61 no ecommerce The Look.



#### ⚪ **Query 7:** Qual é a taxa de retorno de clientes (quantos clientes voltaram para fazer uma segunda compra)?
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

#### *Resultado:*
total_clientes |
-- |
3342 |

#### *Insights:*
  - 3342 clientes fizeram mais de uma compra.
  - Aproximadamente um terço dos clientes está satisfeito e voltando para fazer mais compras no The Look.


### Análise de Produtos:

#### ⚪ **Query 8:** Quais são os produtos mais vendidos?
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

#### *Resultado:*
vendas | product_id	| name
-- | -- | --
8	| 20753 |	Cinch Jeans Mens Carter Medium Stone
8	| 12370	| Bra Garter Thong Set
8	| 16064	| VOLCOM Think Mens Thermal
8	| 4368	| Silver Jeans Juniors Frances 18 Bootcut Jean
8	| 18391	| PUMA Men's Knitted Tricot Jacket

#### *Insights:*
  - Os produtos mais vendidos são itens de vestuário variados, desde jeans e jaquetas até conjuntos de lingerie.


#### ⚪ **Query 9:** Quais produtos geraram a maior receita?
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

#### *Resultado:*
valor_vendido |	product_id	| name
-- | -- | --
3612.0	| 2793	| adidas Women's adiFIT Slim Pant
3612.0	| 2796	| ASCIS Cushion Low Socks (Pack of 3)
3612.0	| 8429	| The North Face Women's S-XL Oso Jacket
3000.0	| 8319	| Canada Goose Women's Mystique
2997.0	| 23546	| Alpha Industries Rip Stop Short

#### *Insights:*
  - Produtos das marcas Adidas, ASCIS e The North Face geraram a maior receita.
  - Os produtos mais vendidos não necessariamente geraram a maior receita. A lista de produtos mais vendidos inclui uma variedade de itens de vestuário, enquanto os produtos de maior receita incluem itens de marcas premium, sugerindo que produtos de alta qualidade e preço mais elevado contribuem significativamente para a receita total.



### Análise de Campanhas:

#### ⚪ **Query 10:** Qual campanha teve o melhor desempenho em termos de aumento de vendas?
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

#### *Resultado:*
event_type	| traffic_source	| vendas
-- | -- | --
purchase	| Email	| 82517
purchase	| Adwords	| 54747
purchase	| YouTube	| 18301
purchase	| Facebook	| 18266
purchase	| Organic	| 9074

#### *Insights:*
  - A campanha de Email foi a mais eficaz, gerando o maior número de vendas. Dado o sucesso, é válido para o The Look investir mais recursos em campanhas de email marketing.
  - As campanhas de Facebook e com tráfego orgânico tiveram o menor desempenho em termos de vendas.



### Análise Geográfica:

#### ⚪ **Query 11:** Quais são os países com maior número de compras?
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

#### *Resultados:*
country	| vendas
-- | --
China	| 10664
United States	| 7248
Brasil	| 4529
South Korea	| 1679
France	| 1487
United Kingdom	| 1466
Germany	| 1263
Spain	| 1202
Japan	| 768
Australia	| 672
Belgium	| 441
Poland	| 64
Colombia	| 2

#### *Insights:*
  - A China é o maior mercado, com 10,664 vendas, seguida pelos Estados Unidos, com 7,248 vendas, e Brasil, com 4,529 vendas.
  - A Colômbia teve o menor volume de vendas, com apenas 2 unidades.
    


#### ⚪ **Query 12:** Qual é a receita gerada por país?
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

#### *Resultado:*
country |	valor_vendido
-- | --
China	| 310879.54
United States	| 217926.2
Brasil	| 135408.77
South Korea	| 51415.24
United Kingdom	| 43009.1
France	| 42392.29
Germany	| 34501.89
Spain	| 33583.39
Japan	| 22726.76
Australia	| 20434.16
Belgium	| 11114.38
Poland	| 1704.73

#### *Insights:*
  - A China gerou a maior receita, com $310,879.54, correspondendo também ao maior volume de vendas (10,664 unidades).
  - A Coreia do Sul e o Reino Unido, embora tenham um volume de vendas menor, ainda geraram receitas significativas ($51,415.24 e $43,009.10, respectivamente).


### Análise de Categorias:


#### ⚪ **Query 13:** Qual é a distribuição de vendas por categoria de produto?
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
#### *Resultados:*
category | vendas
-- | --
Intimates	| 3458
Jeans	| 3278
Tops & Tees |	2973
Fashion Hoodies & Sweatshirts	| 2932
Swim	| 2905
Sweaters	| 2845
Shorts	| 2844
Sleep & Lounge	| 2785
Accessories	| 2489
Active	| 2331
Outerwear & Coats |	2276
Underwear	| 1887
Pants	| 1829
Socks |	1513
Maternity	| 1332
Dresses	| 1306
Suits & Sport Coats	| 1302
Plus	| 1066
Socks & Hosiery |	947
Pants & Capris	| 848
Blazers & Jackets |	799
Leggings	| 730
Skirts	| 527
Suits	| 247
Jumpsuits & Rompers	| 214
Clothing Sets |	59

#### *Insights:*
  -  Roupas Íntimas, Jeans e Camisetas e Blusas são as categorias mais populares entre os clientes.
  -  Ternos, Macacões e Conjuntos são as categorias com o menor volume de vendas.



#### ⚪ **Query 14:** Quais categorias de produtos têm o melhor desempenho em termos de receita?
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

#### *Resultado:*
categoria	| valor_vendas
-- | --
Outerwear & Coats	| 338060.89
Jeans |	326753.06
Sweaters | 215585.27
Swim	| 163713.05
Suits & Sport Coats	| 161992.28

#### *Insights:*
  - Casacos e Jaquetas, Jeans e Suéteres são as categorias que geraram a maior receita.
  - Jeans são populares tanto em volume de vendas quanto em receita gerada, já as Roupas Íntimas e Camisetas e Blusas, embora sejam as mais vendidas, não aparecem entre as categorias com maior receita, sugerindo que esses itens têm um preço médio mais baixo.
  - Casacos e Jaquetas e Suéteres geram alta receita, mas não estão entre as categorias mais vendidas, indicando que esses itens têm um preço médio mais alto.


