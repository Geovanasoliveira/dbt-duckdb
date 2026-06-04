# Pipeline de dados construído com **DBT + DuckDB**

O projeto transforma 14 tabelas brutas de ERP's em modelos analíticos prontos para consumo, cobrindo faturamento, churn, segmentação de clientes, performance de vendas e logística.

---

## 📁 Estrutura do Projeto

```
northwind/
├── models/
│   ├── staging/          # Limpeza e padronização das fontes brutas
│   │   ├── sources.yml
│   │   ├── schema.yml
│   │   ├── stg_orders.sql
│   │   ├── stg_order_details.sql
│   │   ├── stg_customers.sql
│   │   ├── stg_products.sql
│   │   ├── stg_categories.sql
│   │   ├── stg_employees.sql
│   │   ├── stg_suppliers.sql
│   │   ├── stg_shippers.sql
│   │   ├── stg_territories.sql
│   │   ├── stg_region.sql
│   │   └── stg_employee_territories.sql
│   ├── intermediate/     # Joins e enriquecimento de contexto
│   │   ├── int_orders_enriched.sql
│   │   ├── int_products_enriched.sql
│   │   └── int_customer_orders_history.sql
│   └── marts/            # KPIs e modelos analíticos finais
│       ├── schema.yml
│       ├── fct_revenue.sql
│       ├── fct_revenue_by_category.sql
│       ├── fct_top_customers.sql
│       ├── fct_churn_analysis.sql
│       ├── fct_top_products.sql
│       ├── fct_employee_performance.sql
│       ├── fct_rfm.sql
│       ├── fct_cohort_retention.sql
│       ├── fct_delivery_sla.sql
│       ├── fct_market_basket.sql
│       ├── fct_seasonality.sql
│       ├── dim_customers.sql
│       └── dim_products.sql
├── seeds/                # CSVs das 14 tabelas brutas do ERP
├── macros/
├── tests/
├── dbt_project.yml
└── profiles.yml          # Configurado localmente (não versionado)
```

---

## 🏗️ Arquitetura

O pipeline segue a arquitetura em **3 camadas**:

```
Seeds (CSVs) → Staging → Intermediate → Marts
```

| Camada | Materialização | Responsabilidade |
|---|---|---|
| **Staging** | View | Uma model por tabela fonte. Renomeia colunas, padroniza tipos, trata nulos. Fonte única da verdade por entidade. |
| **Intermediate** | View | Joins entre entidades para construir contexto de negócio. Não exposto diretamente para consumo. |
| **Marts** | Table | Modelos analíticos finais com KPIs e métricas de negócio. Consumidos pelo relatório e ferramentas de BI. |

---

## 📊 Modelos Analíticos

### KPIs de Negócio
| Modelo | Descrição |
|---|---|
| `fct_revenue` | Faturamento mensal agregado por país, com ticket médio |
| `fct_revenue_by_category` | Receita e unidades vendidas por categoria de produto |
| `fct_top_customers` | Ranking de clientes com participação e receita acumulada (Pareto) |
| `fct_top_products` | Produtos rankeados por receita e unidades vendidas |
| `fct_employee_performance` | Receita e pedidos por funcionário |

### Análises Avançadas
| Modelo | Descrição |
|---|---|
| `fct_rfm` | Segmentação RFM com scores NTILE(4) e classificação em Champion, Leal, Potencial, Em risco e Perdido |
| `fct_cohort_retention` | Taxa de retenção mensal por cohort de primeira compra |
| `fct_churn_analysis` | Status de churn por cliente: Ativo (≤45 dias), Em risco (46–90 dias), Churned (>90 dias) |
| `fct_delivery_sla` | Performance de entrega por transportadora — taxa de atraso e dias médios de envio |
| `fct_market_basket` | Pares de produtos comprados juntos com suporte e confiança |
| `fct_seasonality` | Receita média e pedidos por mês do ano (padrão histórico) |

### Dimensões
| Modelo | Descrição |
|---|---|
| `dim_customers` | Clientes com segmentação por valor (Alto, Médio, Baixo) via QUANTILE_CONT |
| `dim_products` | Produtos com categoria e fornecedor enriquecidos |

---

## 🛠️ Stack Técnica

| Ferramenta | Versão | Uso |
|---|---|---|
| [dbt-core](https://docs.getdbt.com) | 1.11+ | Transformação e modelagem |
| [dbt-duckdb](https://github.com/duckdb/dbt-duckdb) | 1.10+ | Adaptador DBT para DuckDB |
| [DuckDB](https://duckdb.org) | 1.0+ | Banco de dados analítico local |
| Python | 3.10+ | Orquestração e visualizações |
| pandas | — | Ingestão dos CSVs |
| matplotlib / seaborn | — | Gráficos exploratórios |
| Google Colab | — | Ambiente de execução |

---

## ▶️ Como Reproduzir

### Pré-requisitos
```bash
pip install dbt-core dbt-duckdb duckdb pandas seaborn matplotlib
```

### 1. Configurar o profiles.yml
Crie o arquivo `~/.dbt/profiles.yml`:
```yaml
northwind:
  target: dev
  outputs:
    dev:
      type: duckdb
      path: /caminho/para/northwind.duckdb
      threads: 1
```

> ⚠️ Use um arquivo `.duckdb` dedicado para o DBT. Não abra conexões Python nesse arquivo enquanto o DBT estiver rodando — o DuckDB não suporta múltiplas conexões de escrita simultâneas.

### 2. Carregar os dados
Coloque os 14 CSVs na pasta `seeds/` (separador `;`, encoding UTF-8) e rode:
```bash
dbt seed
```

### 3. Rodar o pipeline
```bash
dbt run
```

### 4. Executar os testes de qualidade
```bash
dbt test
```

### 5. Gerar documentação
```bash
dbt docs generate
dbt docs serve
```

---

## 🧪 Testes de Qualidade

O projeto inclui testes automáticos via `schema.yml`:

| Teste | Modelos |
|---|---|
| `not_null` | `order_id`, `customer_id`, `product_id`, `employee_id` em todos os stagings |
| `unique` | PKs de todas as tabelas de staging |
| `accepted_values` | `churn_status` → `['Ativo', 'Em risco', 'Churned']` |

---

## 📈 Principais Resultados

| Indicador | Valor |
|---|---|
| Receita total (Jul/96 – Mai/98) | **$1.27M** |
| Crescimento no período | **+344%** |
| Ticket médio por pedido | **$1.530** |
| Clientes ativos | **70,8%** (63/89) |
| Clientes em risco ou perdidos | **29%** (26/89) |
| Taxa média de atraso nas entregas | **4,2%** |
| Produto mais rentável | **Côte de Blaye** ($141K — 11% da receita) |
| Melhor vendedor | **Margaret Peacock** ($232K) |
| Pico sazonal | **Abril** ($88K médio/ano) |

---

## 📂 Entregáveis

```
├── Case_AE_DBT.ipynb               # Notebook completo — setup, pipeline, visualizações
└── Northwind_Relatorio_Executivo.html  # Relatório executivo com insights e recomendações
```

---

## 🗂️ Fonte dos Dados

14 tabelas do ERP da Northwind Traders (sistema PostgreSQL fictício), disponibilizadas em CSV:

`categories` · `customers` · `customer_demographics` · `customer_customer_demo` · `employees` · `employee_territories` · `order_details` · `orders` · `products` · `region` · `shippers` · `suppliers` · `territories` · `us_states`

---

## 👩‍💻 Autora

**Geovana Oliveira** — Data Engineer | Analytics Engineer  
[LinkedIn](https://www.linkedin.com/in/geovanasilvaoliveira/)
