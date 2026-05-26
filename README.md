
# 🔌 Análise da Matriz Energética Brasileira para Suporte à Expansão de Datacenters

> **Projeto de Conclusão de Curso**  
> **Disciplina:** Fundamentos de Dados e Analytics — Engenharia de Dados em Big Data  
> **Instituição:** Universidade Presbiteriana Mackenzie  
> **Professores:** Fabio Rossi Versolatto e Gustavo Moreira Calixto

---

## 📌 Contextualização

A explosão da Inteligência Artificial está provocando uma corrida global por **datacenters**, que são infraestruturas extremamente intensivas em energia elétrica. O Brasil disputa esse mercado, mas a decisão estratégica de localização depende de três fatores principais:

*   **Capacidade:** Energia disponível para grandes cargas.
*   **Sustentabilidade:** Percentual de fontes renováveis (Compromissos ESG).
*   **Estabilidade:** Confiabilidade do fornecimento regional.

Este projeto utiliza dados reais da **ANEEL (Agência Nacional de Energia Elétrica)** para construir um pipeline completo de engenharia de dados visando apoiar decisões de expansão tecnológica no país.

---

## 🎯 Problema a ser Resolvido

O problema consiste em prever, a partir das características de empreendimentos de geração de energia e de fatores relevantes para datacenters — como **geolocalização**, **condições meteorológicas** e **cobertura de fibra óptica** — em qual fase os projetos de energia se encontram (**operação, construção ou não iniciada**) e qual será sua potência fiscalizada esperada.

O objetivo é utilizar essas informações para identificar regiões com maior disponibilidade futura de energia no Brasil, apoiando decisões estratégicas sobre a localização de datacenters e a avaliação da viabilidade de cada região.

---

## 📊 Fonte de Dados

*   **Dataset:** SIGA – Sistema de Informações de Geração da ANEEL.
*   **URL:** [Dados Abertos ANEEL](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel)
*   **Atualização:** Mensal.
*   **Licença:** Open Data Commons Open Database License (ODbL).
*   **Volume:** ~7,5 MiB | 23 colunas | Dezenas de milhares de registros.

---

## 🛠️ Tecnologias Utilizadas

| Camada | Tecnologia |
| :--- | :--- |
| **Notebook / Execução** | Google Colab |
| **Linguagem** | Python 3.11 |
| **Ingestão** | `requests`, `pandas` |
| **Persistência** | MongoDB Atlas (Free Tier) |
| **Arquitetura** | Medallion (Bronze → Silver → Gold) |
| **EDA / Visualização** | `matplotlib`, `seaborn`, `plotly` |
| **Machine Learning** | `scikit-learn`, `xgboost` |
| **Versionamento** | Git / GitHub |

---

## 🗺️ Roadmap por Etapas

### 🔹 Etapa 1: Processamento e Ingestão (Entrega 05/05)
*   Download programático do CSV oficial da ANEEL.
*   Persistência **fiel à origem** na coleção `bronze_siga` do MongoDB.
*   Inclusão de metadados de auditoria (timestamp, fonte, URL).
*   Indexação básica para otimização de acesso.

### 🔹 Etapa 2: Análise Exploratória e Limpeza (Entrega 14/05)
*   **Bronze → Silver:** Tipagem de dados, padronização textual, deduplicação por `CodCEG` e validação de regras de negócio.
*   **EDA:** Análise de distribuição por tipo de geração, ranking de UFs e mapas interativos.
*   **Silver → Gold:** Criação de data mart por UF com o **Índice de Atratividade para Datacenters**.

### 🔹 Etapa 3: Aplicação de ML (Entrega 26/05)
*   **Classificação:** Uso de Random Forest vs XGBoost para prever a fase do empreendimento.
*   **Regressão:** Predição da potência fiscalizada esperada.
*   **Consumo:** Persistência das predições na camada Gold para dashboards.

---

## 📁 Estrutura do Repositório

```text
.
├── README.md
├── Projeto_ANEEL_Datacenters.ipynb    # Notebook principal (Colab)
├── data/                              # Dados intermediários (gitignored)
└── docs/                              # Slides da apresentação final
```

---

## 🚀 Como Executar

1.  Crie uma conta gratuita no [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/register).
2.  Configure o **Database Access** (usuário/senha) e o **Network Access** (IP `0.0.0.0/0`).
3.  Obtenha sua *Connection String* (`mongodb+srv://...`).
4.  Abra o notebook no **Google Colab**.
5.  Em **Configurações → Secrets**, adicione a chave `MONGO_URI` com o valor da sua string de conexão.
6.  Execute as células em sequência.

---

## 👥 Integrantes

| Nome | RA | GitHub |
| :--- | :--- | :--- |
| **Bruno de Souza Ribeiro** | 10731796 | [@yusukeyoko](https://github.com/yusukeyoko) |
| **Wender Carlos** | 10732285 | [@WenderCarlosPS](https://github.com/WenderCarlosPS) |
| **Tauã Matheus** | 10732344 | [@tauamat](https://github.com/tauamat) |
| **Leonardo Cuenca** | 10732437 | [@leonardocuenca98](https://github.com/leonardocuenca98) |

---

## 📅 Cronograma de Entregas

| Data | Marco |
| :--- | :--- |
| **23/04** | Planejamento completo e estruturação do README |
| **05/05** | Etapa 1 — Ingestão + Camada Bronze |
| **14/05** | Etapa 2 — Camadas Silver/Gold + EDA |
| **26/05** | Etapa 3 — Modelos de Machine Learning |
| **02/06** | Apresentação Final do Projeto |

---

## 📖 Dicionário de Dados (Principais Atributos)

| Coluna | Tipo | Descrição |
| :--- | :--- | :--- |
| `NomEmpreendimento` | string | Nome da usina/empreendimento |
| `CodCEG` | string | Código único do empreendimento (ANEEL) |
| `SigUFPrincipal` | string | Unidade federativa (UF) |
| `SigTipoGeracao` | string | Tipo de geração (ex: UHE, PCH, CGH) |
| `DscFaseUsina` | string | Fase atual (Operação, Construção, etc.) |
| `MdaPotenciaOutorgadaKw`| float | Potência outorgada em kW |
| `MdaPotenciaFiscalizadaKw`| float | Potência fiscalizada em kW |
| `NumCoordNEmpreendimento`| float | Latitude para georreferenciamento |
| `NumCoordEEmpreendimento`| float | Longitude para georreferenciamento |

---

## ⚖️ Critérios de Seleção

Com a ascensão da Inteligência Artificial (IA), a demanda por infraestrutura tecnológica tornou-se uma prioridade global. Diante desse cenário, selecionamos um conjunto de dados focado na relação entre o consumo de energia e a operação de **Datacenters**. O objetivo é analisar o posicionamento estratégico do Brasil na expansão desses centros em território nacional, abordando o pilar fundamental para o avanço dessa tecnologia: a matriz energética e a melhor localização necessária para sustentar tamanha carga computacional.

---

## 📜 Licença

Este projeto possui finalidade estritamente acadêmica. Os dados utilizados são de domínio público, regidos pela licença **ODbL** da ANEEL.
