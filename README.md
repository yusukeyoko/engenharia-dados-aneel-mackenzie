# engenharia-dados-aneel-mackenzie
Projeto para a conclusão do curso de engenharia de dados Mackenzie- Aneel  
# 🔌 Análise da Matriz Energética Brasileira para Suporte à Expansão de Datacenters

> Projeto da disciplina **Fundamentos de Dados e Analytics — Engenharia de Dados em Big Data**  
> Universidade Presbiteriana Mackenzie  
> Profs. Fabio Rossi Versolatto e Gustavo Moreira Calixto

---

## 📌 Contextualização

A explosão da Inteligência Artificial está provocando uma corrida global por **datacenters**, que são infraestruturas extremamente intensivas em energia elétrica. O Brasil disputa esse mercado, mas a decisão de **onde** instalar esses datacenters depende de três fatores principais: **capacidade de energia disponível**, **percentual de fontes renováveis** (compromissos ESG) e **estabilidade do fornecimento**.

Este projeto utiliza dados governamentais reais da **ANEEL (Agência Nacional de Energia Elétrica)** para construir um pipeline completo de engenharia de dados que apoia decisões de localização de datacenters no Brasil.

## 🎯 Problema a ser resolvido

Como prever, a partir das características de um empreendimento de geração de energia, em qual fase ele se encontra (Operação, Construção, Construção não iniciada) e qual sua potência fiscalizada esperada — fornecendo subsídio para decisões de localização de datacenters?

## 📊 Fonte de dados

- **Dataset:** SIGA – Sistema de Informações de Geração da ANEEL
- **URL:** https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel
- **Atualização:** mensal
- **Licença:** Open Data Commons Open Database License (ODbL) — dados abertos governamentais brasileiros, gratuitos
- **Volume:** ~7,5 MiB, com 23 colunas e dezenas de milhares de empreendimentos

## 🛠️ Tecnologias utilizadas

| Camada | Tecnologia |
|---|---|
| Notebook / Execução | Google Colab |
| Linguagem | Python 3.11 |
| Ingestão | `requests`, `pandas` |
| Persistência | **MongoDB Atlas** (Free Tier) |
| Arquitetura de dados | **Medallion** (Bronze → Silver → Gold) |
| EDA / Visualização | `matplotlib`, `seaborn`, `plotly` |
| Machine Learning | `scikit-learn`, `xgboost` |
| Versionamento | Git / GitHub |

## 🗺️ Roadmap por etapa

### Etapa 1 — Processamento e Ingestão (entrega 05/05)
- Download programático do CSV oficial da ANEEL
- Persistência **fiel à origem** na coleção `bronze_siga` do MongoDB
- Metadados de auditoria (timestamp, fonte, URL)
- Indexação básica para acesso

### Etapa 2 — Análise Exploratória e Limpeza (entrega 14/05)
- Bronze → **Silver**: tipagem (numérico/data/coordenadas), padronização textual, deduplicação por `CodCEG`, validação de regras de negócio
- EDA: distribuição por tipo de geração, ranking de UFs por capacidade, evolução temporal por fonte, mapa interativo
- Silver → **Gold**: data mart por UF com **Índice de Atratividade para Datacenters** (capacidade × % renovável × pipeline)

### Etapa 3 — Aplicação de ML (entrega 26/05)
- **Classificação** (Random Forest vs XGBoost): prever a fase do empreendimento
- **Regressão** (Random Forest Regressor): prever a potência fiscalizada
- Persistência das predições na camada Gold para consumo por dashboards

## 📁 Estrutura do repositório

```
.
├── README.md
├── Projeto_ANEEL_Datacenters.ipynb   # Notebook principal (Colab)
├── data/                              # Dados intermediários (gitignored)
└── docs/                              # Slides da apresentação final
```

## 🚀 Como executar

1. Crie uma conta gratuita em [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/register) (cluster M0 — 512 MB grátis)
2. Em **Database Access**, crie um usuário/senha
3. Em **Network Access**, libere `0.0.0.0/0` (apenas para uso acadêmico)
4. Copie a connection string (`mongodb+srv://...`)
5. Abra o notebook no Google Colab
6. Em **Configurações → Secrets**, adicione `MONGO_URI` com a connection string
7. Execute todas as células

## 👥 Integrantes

| Nome | RA | GitHub |
|---|---|---|
|Bruno de Souza Ribeiro|10731796 | https://github.com/yusukeyoko |
| Wender Carlos| 10732285 |https://github.com/WenderCarlosPS |
| Tauã Matheus| 10732344 | https://github.com/tauamat |
| Leonardo Cuenca|10732437 | https://github.com/leonardocuenca98 |

## 📅 Cronograma

| Data | Entrega |
|---|---|
| 23/04 | Planejamento completo (este README) |
| 05/05 | Etapa 1 — Ingestão + Bronze |
| 14/05 | Etapa 2 — Silver, Gold + EDA |
| 26/05 | Etapa 3 — Modelos de ML |
| 02/06 | Apresentação Final |

## 📜 Licença

Este projeto é acadêmico. Os dados utilizados estão sob licença **ODbL** da ANEEL.
