# ⚡ GridInsight Analytics

> **Plataforma de Monitoramento de Ações do Setor Elétrico Brasileiro**
> Projeto Acadêmico — Engenharia de Dados | Parte 2: Implementação

[![Python](https://img.shields.io/badge/Python-3.11+-blue)](https://python.org)
[![Streamlit](https://img.shields.io/badge/Dashboard-Streamlit-FF4B4B)](https://streamlit.io)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## 📋 Sumário

1. [Visão Geral](#visão-geral)
2. [Universo de Ativos](#universo-de-ativos)
3. [Arquitetura As-Built](#arquitetura-as-built)
4. [Relatório de Mudanças em Relação à Parte 1](#relatório-de-mudanças)
5. [Como Executar](#como-executar)
6. [Estrutura do Repositório](#estrutura-do-repositório)
7. [Pipeline de Dados](#pipeline-de-dados)
8. [Dashboard](#dashboard)
9. [Testes](#testes)
10. [Qualidade e Governança](#qualidade-e-governança)

---

## Visão Geral

O **GridInsight Analytics** é uma plataforma de engenharia de dados que monitora as **11 ações ordinárias do setor elétrico brasileiro** listadas na B3, classificadas por segmento (geração, transmissão, distribuição e integradas).

A plataforma implementa um pipeline batch completo:

```
Yahoo Finance API → Bronze (JSON) → Silver (Parquet) → Gold (Parquet) → Streamlit Dashboard
```

Com camadas de **qualidade de dados**, **orquestração agendada**, **armazenamento opcional em PostgreSQL** e **testes unitários**.

---

## Universo de Ativos

| Ticker | Empresa | Segmento |
|---|---|---|
| ELET3 | Eletrobras | ⚡ Integrada |
| CMIG3 | Cemig | 🏭 Integrada |
| CPLE3 | Copel | 🏭 Integrada |
| EGIE3 | Engie Brasil | ⚡ Geração |
| ENEV3 | Eneva | ⚡ Geração |
| TAEE3 | Taesa | 🔌 Transmissão |
| ALUP3 | Alupar | 🔌 Transmissão |
| CPFE3 | CPFL Energia | 🏘️ Distribuição |
| EQTL3 | Equatorial | 🏘️ Distribuição |
| NEOE3 | Neoenergia | 🏘️ Distribuição |
| ENGI3 | Energisa | 🏘️ Distribuição |

---

## Arquitetura As-Built

> Diagrama atualizado refletindo o que foi **efetivamente implementado**.

```mermaid
flowchart TD
    subgraph FONTES["🌐 Fontes Externas"]
        YF[Yahoo Finance\nyfinance lib]
        META[Metadados\nconfig.py]
    end

    subgraph INGESTAO["⚙️ Ingestão — Bronze"]
        PC[price_collector.py\nDiário / Sob demanda]
        FC[fundamentals_collector.py\nSemanal]
        BR1[(data/bronze/prices/\nJSON por ticker/data)]
        BR2[(data/bronze/fundamentals/\nJSON por ticker)]
    end

    subgraph QUALIDADE["🔍 Qualidade"]
        DV[data_validator.py\n8 regras de negócio]
        QL[(logs/quality_log.json)]
    end

    subgraph SILVER["🥈 Silver — Parquet Limpo"]
        SP[silver_processor.py\nLimpeza + Enriquecimento]
        SV1[(data/silver/ohlcv/\nParquet por ticker)]
        SV2[(data/silver/ohlcv_completo.parquet\nUnificado)]
    end

    subgraph GOLD["🥇 Gold — Analítico"]
        GP[gold_processor.py\nIndicadores + Agregações]
        GO1[(data/gold/indicators.parquet\nRetornos, Vol, MM, RSI)]
        GO2[(data/gold/rankings.parquet\nRanking dos 11 ativos)]
        GO3[(data/gold/sector_view.parquet\nVisão por segmento)]
        GO4[(data/gold/fundamentals_latest.parquet)]
    end

    subgraph ORQUESTRACAO["🎛️ Orquestração"]
        PR[pipeline_runner.py\nschedule lib]
        LOG[(logs/pipeline_runs.json)]
    end

    subgraph STORAGE_OPC["🐘 Storage Opcional"]
        PG[(PostgreSQL\nvia Docker Compose)]
        SQ[(SQLite\npadrão local)]
    end

    subgraph CONSUMO["🖥️ Consumo"]
        ST[Streamlit Dashboard\n5 abas interativas]
        EX[Exportação CSV/Excel]
    end

    subgraph TESTES["🧪 Testes"]
        PY[pytest\ntests/test_indicators.py]
    end

    YF --> PC --> BR1
    YF --> FC --> BR2
    META --> SP

    BR1 --> DV --> QL
    DV --> SP

    SP --> SV1
    SP --> SV2
    BR2 --> GP

    SV2 --> GP
    GP --> GO1 & GO2 & GO3 & GO4

    PR -.->|Agenda e encadeia| PC & FC & SP & GP
    PR --> LOG

    GO1 & GO2 & GO3 & GO4 --> ST
    GO1 --> EX
    SV1 --> PG
    GO1 --> PG
    SV1 --> SQ

    GP --> TESTES

    style FONTES fill:#2c3e50,color:#fff
    style INGESTAO fill:#7B3F00,color:#fff
    style QUALIDADE fill:#5c0a0a,color:#fff
    style SILVER fill:#0a3a5c,color:#fff
    style GOLD fill:#1a3a1a,color:#fff
    style ORQUESTRACAO fill:#3d3d3d,color:#fff
    style STORAGE_OPC fill:#2a2a4a,color:#fff
    style CONSUMO fill:#3a0a5c,color:#fff
    style TESTES fill:#1a1a1a,color:#fff
```

---

## Relatório de Mudanças

> **Seção obrigatória** — documenta o que mudou em relação ao plano da Parte 1 e as justificativas técnicas.

### Mudanças em relação à Parte 1

| Componente | Planejado (Parte 1) | Implementado (Parte 2) | Justificativa |
|---|---|---|---|
| **Orquestração** | Apache Airflow | `pipeline_runner.py` + biblioteca `schedule` | Airflow exige JVM, banco de dados próprio e configuração de executor. Para o escopo acadêmico local, um script Python com `schedule` entrega o mesmo resultado (agendamento, encadeamento de tarefas, log de execuções) sem overhead de infraestrutura. |
| **Consumo/API** | FastAPI + Streamlit separados | Streamlit unificado | O Streamlit já expõe dados de forma interativa e suficiente para demonstração. A FastAPI seria necessária para integração com sistemas externos — escopo fora desta fase. |
| **Transformação** | dbt (SQL) | Scripts Python/pandas | dbt exige Node.js e configuração de perfil de conexão. Para volume de dados < 50 MB, pandas oferece o mesmo resultado com zero dependência extra. |
| **Armazenamento primário** | MinIO / S3 | Sistema de arquivos local (Parquet) | MinIO requer Docker e adiciona complexidade operacional desnecessária para ambiente local. A estrutura de pastas Bronze/Silver/Gold é idêntica e migrável para S3 sem mudança de código. |
| **Banco de dados** | PostgreSQL obrigatório | SQLite padrão + PostgreSQL opcional (Docker) | SQLite funciona sem Docker e sem configuração, tornando o projeto executável em qualquer máquina com Python. O schema PostgreSQL foi mantido para ambientes que desejam configuração mais robusta. |
| **Governança** | DataHub | `quality_log.json` + `pipeline_runs.json` | DataHub requer cluster Kubernetes e é desproporcional para 11 ativos. Os logs JSON cumprem o requisito de rastreabilidade e auditoria com solução leve. |
| **Fonte de dados** | Yahoo Finance + Fundamentus (scraping) | Yahoo Finance principal via `yfinance` | O scraping do Fundamentus exige manutenção frequente de seletores CSS. O `yfinance` fornece dados suficientes incluindo fundamentalistas básicos (DY, P/L, ROE). |

### O que foi mantido exatamente como planejado

- ✅ Estrutura Medallion Architecture: Bronze → Silver → Gold
- ✅ Classificação por segmento (geração, transmissão, distribuição, integradas)
- ✅ 11 ativos exatos: ELET3, CPFE3, EQTL3, NEOE3, EGIE3, ENEV3, ENGI3, CMIG3, CPLE3, TAEE3, ALUP3
- ✅ Indicadores técnicos: RSI 14, MM20, MM50, retornos em múltiplas janelas, volatilidade
- ✅ Validação de qualidade com 8 regras de negócio
- ✅ Formato Parquet para armazenamento analítico
- ✅ Pipeline exclusivamente batch (sem streaming)
- ✅ Testes unitários com pytest

---

## Como Executar

### Pré-requisitos

- Python 3.11+
- pip
- (Opcional) Docker Desktop — para PostgreSQL

### Passo 1 — Clonar e instalar

```bash
git clone https://github.com/seu-usuario/gridinsight-analytics.git
cd gridinsight-analytics

# Criar ambiente virtual (recomendado)
python -m venv .venv
source .venv/bin/activate      # Linux/Mac
.venv\Scripts\activate         # Windows

# Instalar dependências
pip install -r requirements.txt

# Configurar variáveis de ambiente
cp .env.example .env
```

### Passo 2 — Executar o pipeline inicial

```bash
# Carga histórica completa (3 anos de dados — ~2 min)
python -m src.orquestracao.pipeline_runner --init
```

Saída esperada:
```
2025-04-24 19:30:01 [INFO] 🚀 GRIDINSIGHT PIPELINE — 24/04/2025 19:30:01
2025-04-24 19:30:01 [INFO] 🔵 ETAPA 1 — INGESTÃO
2025-04-24 19:30:05 [INFO]   ✅ ELET3.SA: 756 registros → 2025-04-24.json
...
2025-04-24 19:32:10 [INFO] 🏁 PIPELINE CONCLUÍDO COM SUCESSO em 129.4s
```

### Passo 3 — Abrir o dashboard

```bash
streamlit run src/serving/app.py
```

Acesse: **http://localhost:8501**

### Passo 4 (opcional) — PostgreSQL via Docker

```bash
docker compose up -d
# Adicione ao .env:
# DATABASE_URL=postgresql://gridinsight:gridinsight@localhost:5432/gridinsight

# Rodar pipeline novamente para popular o banco
python -m src.orquestracao.pipeline_runner --run
```

### Executar testes

```bash
pytest tests/ -v
```

### Comandos rápidos via Makefile

```bash
make install     # instala dependências
make init        # pipeline inicial completo
make run         # pipeline incremental (dados do dia)
make dashboard   # abre o Streamlit
make test        # roda os testes
make docker-up   # sobe o PostgreSQL
```

---

## Estrutura do Repositório

```
gridinsight-analytics/
│
├── src/                            # Código-fonte principal
│   ├── config.py                   # Configuração central (ativos, paths, segmentos)
│   │
│   ├── ingestao/
│   │   ├── price_collector.py      # Coleta OHLCV via yfinance → Bronze JSON
│   │   └── fundamentals_collector.py # Coleta P/L, DY, ROE → Bronze JSON
│   │
│   ├── qualidade/
│   │   └── data_validator.py       # 8 regras de qualidade + quality_log.json
│   │
│   ├── transformacao/
│   │   ├── silver_processor.py     # Bronze → Silver (Parquet limpo)
│   │   └── gold_processor.py       # Silver → Gold (indicadores técnicos + rankings)
│   │
│   ├── orquestracao/
│   │   └── pipeline_runner.py      # Encadeia etapas + agendamento (schedule)
│   │
│   └── serving/
│       └── app.py                  # Dashboard Streamlit (5 abas)
│
├── data/                           # Dados gerados (não versionados)
│   ├── bronze/                     # Dados brutos (JSON)
│   ├── silver/                     # Dados limpos (Parquet)
│   └── gold/                       # Dados analíticos (Parquet)
│
├── logs/                           # Logs de qualidade e execução
│   ├── quality_log.json
│   └── pipeline_runs.json
│
├── sql/
│   └── schema.sql                  # Schema PostgreSQL completo
│
├── tests/
│   └── test_indicators.py          # 14 testes unitários (pytest)
│
├── .streamlit/
│   └── config.toml                 # Tema escuro para o dashboard
│
├── docker-compose.yml              # PostgreSQL 15 + pgAdmin
├── requirements.txt
├── Makefile
├── .env.example
└── README.md
```

---

## Pipeline de Dados

### Fluxo de execução

```
pipeline_runner.py --init
│
├─ ETAPA 1: INGESTÃO
│   ├─ price_collector.py        → data/bronze/prices/{TICKER}/{data}.json
│   └─ fundamentals_collector.py → data/bronze/fundamentals/{TICKER}/{data}.json
│
├─ ETAPA 2: QUALIDADE
│   └─ data_validator.py         → valida 8 regras, gera logs/quality_log.json
│
├─ ETAPA 3: SILVER
│   └─ silver_processor.py       → data/silver/ohlcv_completo.parquet
│
└─ ETAPA 4: GOLD
    └─ gold_processor.py         → data/gold/indicators.parquet
                                 → data/gold/rankings.parquet
                                 → data/gold/sector_view.parquet
```

### Indicadores calculados (Gold)

| Indicador | Descrição |
|---|---|
| `retorno_diario` | Variação percentual do fechamento em relação ao dia anterior |
| `retorno_5d` | Retorno acumulado em 5 dias úteis |
| `retorno_21d` | Retorno acumulado em 21 dias úteis (~1 mês) |
| `retorno_252d` | Retorno acumulado em 252 dias úteis (~1 ano) |
| `volatilidade_21d` | Desvio-padrão dos retornos diários × √252 (anualizada) |
| `mm20` / `mm50` | Médias Móveis Simples de 20 e 50 dias |
| `rsi_14` | Relative Strength Index de 14 dias (0–100) |
| `dist_mm20_pct` | Distância percentual do preço em relação à MM20 |
| `maxima_52s` / `minima_52s` | Máxima e mínima dos últimos 252 pregões |
| `sinal_rsi` | `sobrecomprado` (>70), `sobrevendido` (<30) ou `neutro` |
| `tendencia_mm` | `alta` (MM20 > MM50) ou `baixa` |

---

## Dashboard

O dashboard Streamlit expõe os dados Gold em **5 abas interativas**:

| Aba | Conteúdo |
|---|---|
| 📊 **Visão Geral** | KPIs do dia, tabela completa dos 11 ativos com retornos e indicadores |
| 🏭 **Por Segmento** | Comparativo de retorno e volatilidade por segmento setorial, gráficos de barras e pizza |
| 📈 **Análise Individual** | Candlestick + MM20/MM50 overlay, gráfico RSI, métricas do ativo selecionado |
| 🏆 **Rankings** | Ranking dos 11 ativos por retorno (diário / 21d / 252d), gráfico de barras colorido |
| 🔍 **Qualidade & Pipeline** | Status da última execução, logs de qualidade por ticker, histórico de runs |

**Filtros sidebar:** segmento setorial, ticker específico, intervalo de datas.

---

## Testes

```bash
pytest tests/ -v
```

**14 testes unitários** cobrindo:

- `TestCalcRSI` — 4 testes: range válido (0-100), mercado em alta, queda, tipo de retorno
- `TestCalcIndicadores` — 7 testes: retorno diário NaN, MM positiva, colunas obrigatórias, máxima ≥ mínima, volatilidade não-negativa, sinal RSI
- `TestDataValidator` — 5 testes: registros válidos passam, close negativo detectado, high < low detectado, variação extrema, stats calculados

---

## Qualidade e Governança

### Regras de qualidade (`data_validator.py`)

| Regra | Descrição |
|---|---|
| `preco_close_positivo` | Close > 0 |
| `preco_open_positivo` | Open > 0 |
| `volume_positivo` | Volume ≥ 0 |
| `high_maior_igual_low` | High ≥ Low |
| `close_dentro_high_low` | Low ≤ Close ≤ High |
| `open_dentro_high_low` | Low ≤ Open ≤ High |
| `close_nao_nulo` | Close não é nulo |
| `variacao_extrema` | \|Close/Open - 1\| < 30% |

### Rastreabilidade

- **`logs/quality_log.json`** — anomalias por ticker, data e tipo, com timestamp de detecção
- **`logs/pipeline_runs.json`** — histórico das últimas 50 execuções com status por etapa e duração
- **`logs/pipeline_YYYY-MM-DD.log`** — log completo em arquivo por dia de execução
- **Camada Bronze preservada** — dados brutos nunca são sobrescritos, permitindo reprocessamento

### Segurança

- Credenciais de API em `.env` (excluído do Git via `.gitignore`)
- Dados financeiros brutos excluídos do versionamento (`data/` no `.gitignore`)
- Schema SQL com constraints e índices para integridade dos dados no PostgreSQL

---

## Tecnologias — Justificativas

| Tecnologia | Função | Por quê |
|---|---|---|
| **yfinance** | Ingestão de preços e fundamentais | Gratuito, sem API key, suporta todos os tickers B3 (.SA), retorna DataFrame pandas nativo |
| **pandas + NumPy** | Processamento e cálculo de indicadores | Volume de dados < 50 MB não justifica Spark; pandas é suficiente e tem integração nativa com Parquet |
| **Apache Parquet** | Formato de armazenamento Silver e Gold | Orientado a colunas, compressão Snappy, leitura analítica 10x mais rápida que CSV |
| **schedule** | Orquestração e agendamento | Leve, sem dependências, funciona no mesmo processo Python — adequado para escala acadêmica |
| **Streamlit** | Dashboard interativo | Código Python puro, deploy em 1 comando, suporte nativo a Plotly e DataFrames |
| **Plotly** | Visualizações interativas | Candlestick, RSI e heatmaps interativos; tema escuro compatível com o GridInsight |
| **SQLite** | Banco de dados padrão local | Zero configuração, sem Docker necessário, suficiente para ambiente de desenvolvimento |
| **PostgreSQL + Docker** | Banco de dados opcional produção | Schema completo definido, ativado via variável de ambiente — pronto para deploy em nuvem |
| **pytest** | Testes unitários | Padrão da indústria Python, integração simples, saída clara |
| **python-dotenv** | Gestão de credenciais | Mantém segredos fora do código-fonte e do repositório Git |

---

## Referências

- [yfinance Documentation](https://github.com/ranaroussi/yfinance)
- [Apache Parquet Format](https://parquet.apache.org/docs/)
- [Streamlit Documentation](https://docs.streamlit.io/)
- [Plotly Python](https://plotly.com/python/)
- [B3 — Dados históricos](https://www.b3.com.br/pt_br/market-data-e-indices/)
- REIS, J.; HOUSLEY, M. *Fundamentals of Data Engineering*. O'Reilly, 2022.
- KLEPPMANN, M. *Designing Data-Intensive Applications*. O'Reilly, 2017.

---

*GridInsight Analytics · Engenharia de Dados · Setor Elétrico Brasileiro · B3*
