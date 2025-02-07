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
- Usar LEFT JOIN e INNER JOIN para combinar duas tabelas

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

Utilizei um arquivo com mais de 100mb, então precisei fazer upload no Cloud Storage. A referência do dataset (que encontrei no kaggle) vai estar no passo a passo no GitHub.

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




