# 10 dias de GCP: Self-service de Dados em 5 minutos

## Para quem sabe o básico de SQL, Excel e análise de dados.

<details>
  <summary>📊 BigQuery - Manipulação e Análise de Dados</summary>

#### Dia 1

- Criar uma tabela e carregar um CSV manualmente
- Executar uma consulta SQL simples (ex: SELECT * FROM tabela LIMIT 10;)
- Filtrar dados por data (ex: WHERE data >= '2024-01-01')

#### Dia 2

- Criar uma tabela baseada em outra usando SELECT INTO
- Criar uma tabela com dados únicos removendo duplicatas (DISTINCT)
- Criar uma tabela com contagem de registros por categoria (GROUP BY)

#### Dia 3

- Fazer uma soma de valores agrupados (SUM)
- Criar colunas calculadas (CASE WHEN, IF)
- Criar uma tabela temporária e testar consultas
- Exportar dados para um arquivo CSV no Cloud Storage

</details>

<details>
  <summary>📊 BigQuery - Análises Avançadas (Fáceis de entender para quem conhece Excel)</summary>

#### Dia 4

- Criar uma tabela com valores acumulados (SUM() OVER())
- Criar uma classificação (RANK() OVER()) de valores
- Fazer uma análise de médias móveis usando SQL (AVG() OVER())

#### Dia 5

- Criar tabelas com partições por data para melhorar desempenho
<!-- - Usar LEFT JOIN e INNER JOIN para combinar duas tabelas -->

</details>

<details>
  <summary>📈 Looker Studio - Visualização de Dados</summary>

#### Dia 6

- Criar um painel/dashboard conectado ao BigQuery
- Criar gráficos de barras e tabelas dinâmicas
- Criar um filtro de data interativo

#### Dia 7

- Adicionar métricas calculadas
- Criar um gráfico de tendências com dados de séries temporais

</details>

<details>
  <summary>🔄 Data Prep - Limpeza e Transformação de Dados</summary>

#### Dia 8

- Carregar um CSV no Data Prep e visualizar os dados
- Remover colunas desnecessárias no Data Prep
- Criar colunas calculadas no Data Prep (ex: concatenar nome + sobrenome)

#### Dia 9

- Filtrar registros com base em uma condição (ex: vendas acima de R$1000)
- Remover linhas duplicadas automaticamente

</details>

<details>
  <summary>🛠️ Integração e Automação Simples</summary>

#### Dia 10

- Agendar uma consulta SQL no BigQuery para rodar automaticamente
- Criar uma conexão entre uma planilha do Google Sheets e o BigQuery
- Exportar dados do BigQuery diretamente para o Google Sheets
- Criar uma tabela externa no BigQuery conectada a um Google Sheets
- Criar um alerta automático para quando um valor ultrapassa um limite

</details>


### Dia 1

#### 1. Criar uma tabela e carregar um CSV manualmente

- No console, adicionar fonte de dados em "Explorer.... + ADD" 
- Escolher "Arquivo local" ou "Google Cloud Storage" se csv tiver mais de 100mb
- Escolher "Criar novo dataset" e outras configurações de padrão

Utilizei um arquivo com mais de 100mb, então precisei fazer upload no Cloud Storage.

> Fonte de dados escolhida: https://www.kaggle.com/datasets/mkechinov/ecommerce-purchase-history-from-electronics-store?resource=download

#### 2. Executar uma consulta SQL simples

Na lateral esquerda, nos "três pontos" ao lado do dataset, selecionar: "Query" ou "Consulta"

```
SELECT *
FROM `<nome-do-projeto>.<base>.<tabela>`
LIMIT 100;
```

#### 3. Filtrar dados por data

```
SELECT *
FROM `<nome-do-projeto>.<base>.<tabela>`
WHERE event_time > '2020-11-19 08:13:00 UTC'
AND event_time < '2020-11-19 08:15:00 UTC'
LIMIT 1000;
```

### Dia 2

#### 1. Criar uma tabela baseada em outra

Para utilizar dados manipulados de uma tabela e inserir em outra (criar automaticamente outra tabela):

```
CREATE OR REPLACE TABLE `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.purchase_history_limited` AS
SELECT *
FROM `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store`
LIMIT 1000;
```
Neste exemplo simples só foi limitado o número de linhas em 1000, sem outra manipulação adicional.

#### 2. Criar uma tabela com dados únicos removendo duplicatas (DISTINCT)

Para criar uma tabela com valores únicos:

```
CREATE OR REPLACE TABLE `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.purchase_history_unique` AS
SELECT DISTINCT *
FROM `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store`;
```

Consulta adicional para checar linhas repetidas:

```
WITH duplicados AS (
  SELECT
    event_time,
    order_id,
    product_id,
    category_id,
    category_code,
    brand,
    price,
    user_id,
    COUNT(*) OVER (PARTITION BY event_time,
    order_id,
    product_id,
    category_id,
    category_code,
    brand,
    user_id) AS qtd_duplicadas
  FROM `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store`
)
SELECT *
FROM duplicados
WHERE qtd_duplicadas > 1;
```

#### 3. Criar uma tabela com contagem de registros por categoria (GROUP BY)

```
CREATE OR REPLACE TABLE `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.purchase_count_by_category` AS
SELECT
  category_code,
  COUNT(*) AS total_registros
FROM `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store`
GROUP BY category_code
ORDER BY total_registros DESC;
```

### Dia 3

#### 1. Criar consulta de soma de valores agrupados (SUM)


Consulta de GMV em milhões:

```
SELECT
  category_code,
  SUM(price)/1000000 AS GMV_MM
FROM `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store`
GROUP BY category_code
ORDER BY GMV_MM DESC;

```

#### 2. Criar colunas calculadas (CASE WHEN, IF)

Criar faixas de preço:

```
SELECT
  brand,
  price,
  category_code,
  CASE
    WHEN price > 1000 THEN 'Caro'
    WHEN price BETWEEN 501 AND 1000 THEN 'Médio'
    ELSE 'Barato'
  END AS faixa_preco
FROM `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store`
;
```

#### 3. Criar uma tabela temporária e testar consultas

```
-- Início do script
CREATE TEMP TABLE my_temp_table AS
SELECT
  event_time,
  order_id,
  product_id,
  category_id,
  category_code,
  brand,
  price,
  user_id
FROM `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store`
WHERE category_code IS NOT NULL
;

-- Agora você pode testar consultas usando 'my_temp_table'
-- Exemplo: mostrar as 10 maiores vendas
SELECT
  order_id,
  user_id,
  price
FROM my_temp_table
ORDER BY price DESC
LIMIT 10
;

-- Outra consulta: contar registros por categoria
SELECT
  category_code,
  COUNT(*) AS total_registros
FROM my_temp_table
GROUP BY category_code
ORDER BY total_registros DESC
;
-- Fim do script

```

#### 4. Exportar dados para um arquivo CSV no Cloud Storage

Para exportar dados de uma tabela do BigQuery para um arquivo CSV no Cloud Storage, existem algumas maneiras comuns:

1) Pela interface web do BigQuery (no Console GCP)

No Google Cloud Console, vá em BigQuery.
No painel de Explorer, encontre a tabela que deseja exportar.
Clique no nome da tabela para abrir os detalhes.
No canto superior direito, clique em Export.
Escolha Export to GCS.
Na janela de configuração:
Selecione o formato de arquivo (CSV).
Defina o caminho no Cloud Storage (por exemplo, gs://nome-do-seu-bucket/pasta/arquivo.csv).
Ajuste outras opções, como compressão (GZIP, se quiser) ou cabeçalho de colunas no CSV.
Clique em Export e aguarde o processo terminar.
Assim que terminar, o arquivo CSV estará no local especificado no seu bucket do GCS.

2) Usando a linha de comando (bq CLI)

Se você preferir usar o terminal e tiver instalado e configurado o gcloud e o bq CLI, pode executar um comando bq extract. Suponha que você tenha um projeto meu_projeto, dataset meu_dataset e tabela minha_tabela, e queira exportar para gs://meu-bucket/saida.csv:

```
bq extract \
  --destination_format=CSV \
  --field_delimiter="," \
  meu_projeto:meu_dataset.minha_tabela \
  gs://meu-bucket/saida.csv
```

Observações:

--destination_format=CSV: especifica que o formato de saída é CSV.
--field_delimiter: define o separador de campos. Se quiser “;” ou outro caractere, você pode alterar.
Se você quiser o arquivo comprimido (por exemplo, GZIP), pode usar a flag --compression=GZIP. Nesse caso, o nome do arquivo costuma terminar com .csv.gz.
Ao executar, o BigQuery iniciará o job de exportação. Após a conclusão, o arquivo estará em seu bucket do GCS.

3) Usando API ou SDKs
Se você estiver automatizando via código (Python, Java, etc.), também pode usar a BigQuery Storage Write API ou as bibliotecas específicas para jobs de exportação do BigQuery. A lógica é semelhante ao CLI: você cria um job de extract apontando para seu bucket, definindo formato CSV e parâmetros de exportação.

4) Dicas gerais
Tamanho do CSV: Se sua tabela for muito grande, BigQuery pode gerar vários arquivos com sufixos numéricos (sharding) para lidar com grandes volumes.
Controle de schema: Se quiser, por exemplo, excluir colunas ou modificar valores antes da exportação, você pode primeiro criar uma tabela temporária ou fazer uma view no BigQuery com a transformação desejada e depois exportar essa view/tabela.
Permissões: A conta de serviço usada para o job precisa ter permissões de escrita (write) no bucket do Cloud Storage (geralmente a permissão roles/storage.objectAdmin ou roles/storage.objectCreator, etc.). Se der erro de acesso, verifique essas permissões.
Exemplo completo via bq CLI


- Exemplo: Exportar a tabela 'minha_tabela' do dataset 'meu_dataset'
- no projeto 'meu_projeto' para um arquivo CSV no bucket 'meu-bucket'

```
bq extract \
  --destination_format=CSV \
  --field_delimiter="," \
  --compression=GZIP \
  meu_projeto:meu_dataset.minha_tabela \
  gs://meu-bucket/minha_tabela_export_*.csv.gz
```

Observe o uso de * no nome do arquivo — caso a exportação gere vários “shards”. Se for um volume gerenciável e couber em um único arquivo, pode escrever um nome fixo (por exemplo, saida.csv.gz).

### Dia 4

#### 1. Criar uma tabela com valores acumulados (SUM() OVER())

```
CREATE OR REPLACE TABLE `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store-acc` AS
SELECT
  order_id,
  price,
  -- Soma acumulada de 'price' ao longo do tempo
  SUM(price) OVER (
    ORDER BY event_time
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS acumulado_price
FROM `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store`
ORDER BY event_time;
```

#### 2. Criar uma classificação (RANK() OVER()) de valores

```
CREATE OR REPLACE TABLE `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-_ranked` AS
SELECT
  *,
  RANK() OVER (
    ORDER BY price DESC
  ) AS ranking_por_preco
FROM `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store`;
```

#### 3. Fazer uma análise de médias móveis usando SQL (AVG() OVER())

```

CREATE OR REPLACE TABLE 
  `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-cumulative` 
AS
WITH daily_agg AS (
  SELECT 
    DATE(event_time) AS dt,          -- Convertendo para data
    SUM(price) AS daily_revenue      -- Soma do 'price' por dia
  FROM `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store`
  GROUP BY DATE(event_time)
)
SELECT
  dt,
  daily_revenue,
  -- Soma acumulada ao longo dos dias (running total)
  SUM(daily_revenue) OVER (
    ORDER BY dt
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS cumulative_revenue
FROM daily_agg
ORDER BY dt;

```

### Dia 5

#### 1. Criar tabelas com partições por data para melhorar desempenho

```
CREATE OR REPLACE TABLE
`gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store_particionamento`
PARTITION BY DATE(event_time)
AS
SELECT
event_time,
order_id,
product_id,
category_id,
category_code,
brand,
price,
user_id
FROM `gcp-study-448422.e_commerce_purchase_history_from_electronics_store.e-commerce-purchase-history-from-electronics-store`
WHERE event_time >= '2020-08-01' AND event_time <= '2020-08-08';
```

### Dia 6

#### 1. Criar um painel/dashboard conectado ao BigQuery

- Conectar a Fonte de Dados:

    - No Looker Studio (antigo Data Studio), clique em "Criar" > "Fonte de dados" e selecione BigQuery.

    - Escolha o projeto, dataset e tabela desejados (por exemplo, a tabela particionada que você criou).

- Configurar o Conector:

    - Revise as colunas importadas e ajuste nomes, tipos e agregações se necessário.

- Criar o Dashboard:

    - Após conectar a fonte de dados, crie um novo relatório e adicione a fonte.

#### 2. Criar gráficos de barras e tabelas dinâmicas

- Gráficos de Barras:
    - Insira um gráfico de barras para visualizar, por exemplo, o GMV (soma de price) por category_code.
    - Configure os eixos e formate os rótulos conforme a necessidade.

- Tabelas Dinâmicas:
    - Adicione uma tabela dinâmica que permita explorar os dados (ex.: detalhar vendas por marca, categoria e data).

#### 3. Criar um filtro de data interativo

- Adicionar Controle de Data:
    - No menu, selecione "Adicionar um controle" > "Intervalo de data" e posicione-o no relatório.

- Vincular ao Relatório:
    - Configure o controle para que filtre todos os gráficos e tabelas conectadas à mesma fonte de dados.

### Dia 7

#### 1. Adicionar métricas calculadas

- Criar Campos Calculados:

    - No painel de dados do Looker Studio, clique em "Adicionar um campo".

    - Exemplo: Crie uma métrica para calcular o crescimento percentual ou a taxa de conversão.

    - Exemplo de Campo:

```
(SUM(price) - LAG(SUM(price)) OVER(PARTITION BY category_code
ORDER BY DATE(event_time))) / LAG(SUM(price))
OVER(PARTITION BY category_code
ORDER BY DATE(event_time))
```

(Ajuste conforme sua necessidade, lembrando que campos calculados no Looker Studio são definidos usando a interface e funções disponíveis.)

#### 2. Criar um gráfico de tendências com dados de séries temporais

- Gráfico de Linhas:

    - Adicione um gráfico de linhas para exibir a evolução de uma métrica (por exemplo, receita diária) ao longo do tempo.

    - Configuração:
    - Configure o eixo X com a data (event_time ou coluna de data) e o eixo Y com a métrica desejada (ex.: SUM(price)).


### Dia 8

#### 1. Carregar um CSV no Data Prep e visualizar os dados

- Importar Dados:
    - No Data Prep (ex.: Cloud Dataprep by Trifacta), carregue seu arquivo CSV.
    - Visualize o dataset e identifique inconsistências ou colunas irrelevantes.

#### 2. Remover colunas desnecessárias no Data Prep

- Selecionar Colunas:
    - Use as ferramentas de transformação para remover as colunas que não serão utilizadas na análise.

#### 3. Criar colunas calculadas no Data Prep (ex: concatenar nome + sobrenome)

Exemplo – Concatenar Nome e Sobrenome:

- Crie uma nova coluna usando uma fórmula (ex.: CONCAT(nome, ' ', sobrenome)) para combinar informações de duas colunas.
    - Aplicar Transformações:
    - Utilize as funções disponíveis no Data Prep para limpar e padronizar os dados.

### Dia 9

#### 1. Filtrar registros com base em uma condição (ex: vendas acima de R$1000)

- Aplicar Filtro:
    - No Data Prep, defina um filtro para selecionar apenas registros que atendam a um critério, como vendas acima de R$1000.

#### 2. Remover linhas duplicadas automaticamente

- Deduplicação:
    - Utilize a função de deduplicação do Data Prep para eliminar registros repetidos, mantendo apenas a primeira ocorrência ou conforme sua lógica de negócio.

### Dia 10

#### 1. Agendar uma consulta SQL no BigQuery para rodar automaticamente

- Configurar Agendamento:
    - No Console do BigQuery, após criar sua consulta, clique em "Agendar consulta" e defina o cronograma (ex.: diariamente às 02:00 AM).

#### 2. Criar uma conexão entre uma planilha do Google Sheets e o BigQuery

- Usar o Conector do Google Sheets:
    - No Google Sheets, acesse "Dados" > "Conectar a BigQuery" e selecione sua consulta ou tabela.

#### 3. Exportar dados do BigQuery diretamente para o Google Sheets

- Exportação Direta:
    - Configure a exportação para que os dados sejam atualizados automaticamente em um intervalo definido.

#### 4. Criar uma tabela externa no BigQuery conectada a um Google Sheets

- Tabela Externa:
    - Use o comando SQL para criar uma tabela externa que aponte para um arquivo do Google Sheets:

```
CREATE OR REPLACE EXTERNAL TABLE `seu_projeto.seu_dataset.tabela_externa_sheets`
OPTIONS (
  format = 'CSV',
  uris = ['https://docs.google.com/spreadsheets/d/SEU_ID/export?format=csv']
);
```

#### 5. Criar um alerta automático para quando um valor ultrapassa um limite

- Alertas:
    - Utilize o recurso de Alertas do BigQuery ou configure uma automação com Cloud Functions e Pub/Sub para monitorar uma métrica.
    - Exemplo: Crie uma consulta que verifique se o GMV ultrapassa um determinado valor e, se sim, acione uma notificação via e-mail ou Slack.
