# Visualizações e Arquitetura — B3 Energy Analytics

Este documento pode ser adicionado ao repositório na pasta `docs/` ou utilizado para enriquecer o README principal.

---

# Arquitetura da Solução

```mermaid
flowchart TB

    A[yFinance API]

    A --> B[Bronze Layer]
    B --> C[Silver Layer]
    C --> D[Gold Layer]
    D --> E[Data Mart]
    E --> F[Dashboard Streamlit]

    G[Pipeline Logs]
    H[Data Quality]
    I[Pipeline Audit]

    B --> G
    B --> I
    C --> H
```

## Fluxo de Dados

```mermaid
flowchart LR

    A[Coleta Yahoo Finance]
    B[Bronze]
    C[Silver]
    D[Gold]
    E[Data Mart]
    F[Consumo]

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
```

---

# Arquitetura Medalhão

## Bronze Layer

```mermaid
flowchart LR

    A[yFinance]

    A --> B[stock_fundamentals_raw]
    A --> C[stock_price_history_raw]

    B --> D[pipeline_logs]
    C --> E[pipeline_audit]
```

### Responsabilidades

- Ingestão dos dados brutos
- Armazenamento dos snapshots
- Registro de logs
- Auditoria das execuções

---

## Silver Layer

```mermaid
flowchart LR

    A[Dados Brutos]

    A --> B[Validação]
    B --> C[Padronização]
    C --> D[Qualidade]

    D --> E[stock_fundamentals]
    D --> F[stock_price_history]

    D --> G[rejected_records]
    D --> H[data_quality_report]
```

### Responsabilidades

- Limpeza dos dados
- Validação de regras
- Tratamento de nulos
- Controle de qualidade

---

## Gold Layer

```mermaid
flowchart LR

    A[Silver]

    A --> B[Valuation Metrics]
    A --> C[Risk Metrics]
    A --> D[Performance Metrics]

    B --> E[Data Mart]
    C --> E
    D --> E
```

### Indicadores

#### Valuation

- P/L
- P/VP
- EV/EBITDA
- Dividend Yield

#### Risk

- Beta
- Dívida / EBITDA
- Volatilidade

#### Performance

- Retorno 30 dias
- Retorno 90 dias
- Retorno 180 dias
- Retorno 365 dias

---

# Dashboard Analítico

## Página 1 — Overview

### KPIs

- Ativos Monitorados
- Preço Médio
- Dividend Yield Médio
- Beta Médio
- Volatilidade Média

### Visualização

```text
+---------------------------+
| KPI CARDS                 |
+---------------------------+

+---------------------------+
| Risco x Retorno           |
+---------------------------+
```

---

## Página 2 — Valuation

### Gráficos

- Ranking de Dividend Yield
- Ranking de P/L
- Ranking de EV/EBITDA
- Ranking de P/VP

### Exemplo

```text
Dividend Yield

TAEE11  ████████████
EGIE3   █████████
TRPL4   ████████
```

---

## Página 3 — Risk

### Gráficos

- Beta por ativo
- Dívida/EBITDA
- Volatilidade

### Exemplo

```text
Beta

TAEE11  ██
EGIE3   ███
EQTL3   ███████
```

---

## Página 4 — Performance

### Gráficos

- Retorno 30 dias
- Retorno 90 dias
- Retorno 180 dias
- Retorno 365 dias

### Exemplo

```text
Retorno 12 Meses

EQTL3   ███████████████
ELET3   ████████████
TAEE11  ████████
```

---

## Página 5 — Governança

### Monitoramento

- Histórico de Execuções
- Data Quality
- Pipeline Logs
- Auditoria

### Exemplo

```text
Execução     Status

2026-06-12   SUCCESS
2026-06-11   SUCCESS
2026-06-10   SUCCESS
```

---

# Modelo de Dados

```mermaid
erDiagram

    stock_fundamentals_raw ||--o{ stock_fundamentals : transforms
    stock_price_history_raw ||--o{ stock_price_history : transforms

    stock_fundamentals ||--o{ valuation_metrics : generates
    stock_fundamentals ||--o{ risk_metrics : generates

    stock_price_history ||--o{ risk_metrics : generates
    stock_price_history ||--o{ performance_metrics : generates

    valuation_metrics ||--|| dashboard_dataset : feeds
    risk_metrics ||--|| dashboard_dataset : feeds
    performance_metrics ||--|| dashboard_dataset : feeds
```

---

# Entrega de Valor

A plataforma transforma dados brutos do mercado financeiro em informações analíticas prontas para consumo, permitindo:

- Comparação entre empresas do setor elétrico
- Análise de valuation
- Monitoramento de risco
- Avaliação de performance histórica
- Controle de qualidade dos dados
- Observabilidade do pipeline

O resultado final é uma solução completa de Engenharia de Dados com arquitetura Medalhão, Data Mart analítico e camada de visualização para apoio à tomada de decisão.
