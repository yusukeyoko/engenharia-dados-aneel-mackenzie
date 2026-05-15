# 🔌 Análise da Matriz Energética Brasileira para Suporte à Expansão de Datacenters

> **Projeto de Conclusão de Curso**  
> **Disciplina:** Fundamentos de Dados e Analytics — Engenharia de Dados em Big Data  
> **Instituição:** Universidade Presbiteriana Mackenzie  
> **Professores:** Fabio Rossi Versolatto e Gustavo Moreira Calixto

---

## 📌 Contextualização

A explosão da Inteligência Artificial está provocando uma corrida global por **datacenters**, infraestruturas extremamente intensivas em energia elétrica. O Brasil disputa esse mercado, mas a decisão estratégica de localização depende de três fatores principais:

- **Capacidade:** Energia disponível para grandes cargas.
- **Sustentabilidade:** Percentual de fontes renováveis (Compromissos ESG).
- **Estabilidade:** Confiabilidade do fornecimento regional.

Este projeto utiliza dados reais da **ANEEL (Agência Nacional de Energia Elétrica)** para construir um pipeline completo de engenharia de dados visando apoiar decisões de expansão tecnológica no país.

---

## 🎯 Problema a ser Resolvido

Prever, a partir das características de empreendimentos de geração de energia e de fatores relevantes para datacenters — **geolocalização**, **condições meteorológicas** e **cobertura de fibra óptica** — em qual fase os projetos se encontram (**operação, construção ou não iniciada**) e qual será sua potência fiscalizada esperada.

O objetivo é identificar regiões com maior disponibilidade futura de energia no Brasil, apoiando decisões estratégicas sobre a localização de datacenters.

---

## 📊 Fontes de Dados

> ⚠️ **Todas as fontes são lidas exclusivamente do MongoDB Atlas (camada Bronze).**  
> Nenhum download externo é feito durante a execução das Etapas 2 e 3.

| Fonte | Coleção Bronze | Descrição |
| :--- | :--- | :--- |
| **ANEEL/SIGA** | `bronze_siga` | ~24 mil empreendimentos de geração |
| **INMET** | `bronze_inmet_estacoes` | ~530 estações meteorológicas |
| **INMET obs.** | `bronze_inmet_obs` | Observações horárias 1T 2026 |
| **ANATEL** | `bronze_anatel_*` (5 coleções) | Contratos e empresas credenciadas |
| **Datacenters** | `bronze_datacenters` | Tabela pública de DCs mundiais |

---

## 🛠️ Tecnologias Utilizadas

| Camada | Tecnologia |
| :--- | :--- |
| **Notebook / Execução** | Google Colab |
| **Linguagem** | Python 3.11 |
| **Persistência** | MongoDB Atlas (Free Tier) — única fonte de dados |
| **Arquitetura** | Medallion: Bronze → Silver → Gold |
| **EDA / Visualização** | `matplotlib`, `seaborn`, `plotly` |
| **Join Geoespacial** | `scipy.spatial.cKDTree` |
| **Machine Learning** | `scikit-learn`, `xgboost` |
| **Versionamento** | Git / GitHub |

---

## 🗺️ Roadmap por Etapas

### 🔹 Etapa 1 — Processamento e Ingestão (Entrega 05/05) · `branch: main`
- Download programático dos CSVs oficiais (ANEEL, INMET, ANATEL, Datacenters).
- Persistência **fiel à origem** nas coleções `bronze_*` do MongoDB.
- Inclusão de metadados de auditoria (`_ingest_ts`, `_source`, `_source_url`).
- Indexação básica para otimização de acesso.

### 🔹 Etapa 2 — Análise Exploratória e Limpeza (Entrega 14/05) · `branch: etapa-2`
- **Bronze → Silver SIGA:** tipagem de dados, coordenadas WGS-84, flag renovável.
- **Bronze → Silver INMET:** agregação climática por estação (temp, umidade, chuva, radiação, vento).
- **Bronze → Silver ANATEL:** proxy de cobertura de telecom/fibra por UF.
- **Bronze → Silver Datacenters:** geocodificação por cidade.
- **Join geoespacial (KDTree):** enriquecimento dos empreendimentos com clima + saturação de DCs.
- **Silver → Gold:** feature store `gold_empreendimentos_features`.
- **EDA completa:** mapas interativos, heatmap climático, scatter ANATEL×ANEEL.

### 🔹 Etapa 3 — Aplicação de ML (Entrega 26/05) · `branch: etapa-3`
- Leitura exclusiva de `gold_empreendimentos_features` (MongoDB).
- **Classificação:** Random Forest vs XGBoost para prever fase do empreendimento.
- **Regressão:** Random Forest para potência fiscalizada esperada (R² > 0,9).
- **Índice de Atratividade para Datacenter:** score composto [0–1] por UF.
- Persistência de `gold_predicoes_potencia` e `gold_ranking_atratividade_dc`.

---

## 📁 Estrutura do Repositório

```text
.
├── README.md
├── Etapa2_EDA_Limpeza.ipynb          # Bronze → Silver → Gold + EDA (MongoDB-only)
├── Etapa3_ML_Modelos.ipynb           # ML: Classificação + Regressão + Índice DC (MongoDB-only)
├── Projeto_ANEEL_Datacenter_v2-Etapa2.ipynb  # Notebook legado (referência)
├── data/                             # Dados intermediários (gitignored)
└── docs/                             # Slides da apresentação final
```

---

## 🚀 Como Executar

### Pré-requisito: MongoDB Atlas com Bronze populado (Etapa 1)

1. Crie uma conta gratuita em [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/register).
2. Configure **Database Access** (usuário/senha) e **Network Access** (IP `0.0.0.0/0`).
3. Obtenha a *Connection String* (`mongodb+srv://...`).
4. Abra o notebook no **Google Colab**.
5. Em **Configurações → Secrets**, adicione a chave `MONGO_URI`.
6. Execute as células em sequência — a célula `0.4 — verificar_bronze()` confirma que os dados estão prontos.

### Fluxo de execução

```
Etapa 1 (ingestão) → MongoDB Bronze populado
       ↓
Etapa2_EDA_Limpeza.ipynb → Silver + Gold + EDA
       ↓
Etapa3_ML_Modelos.ipynb  → Modelos + Ranking + Predições
```

---

## 🏗️ Arquitetura Medallion (MongoDB)

```
bronze_siga                   ─┐
bronze_inmet_estacoes          │  Etapa 2
bronze_inmet_obs               ├─────────────► silver_siga
bronze_anatel_* (×5)           │               silver_inmet
bronze_datacenters            ─┘               silver_anatel_uf
                                               silver_datacenters
                                                      │
                                                      ▼
                                       gold_empreendimentos_features
                                                      │
                                               Etapa 3 (ML)
                                                      │
                                    ┌─────────────────┴──────────────────┐
                                    ▼                                    ▼
                        gold_predicoes_potencia     gold_ranking_atratividade_dc
```

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

| Data | Marco | Status |
| :--- | :--- | :--- |
| **23/04** | Planejamento completo e estruturação do README | ✅ |
| **05/05** | Etapa 1 — Ingestão + Camada Bronze | ✅ |
| **14/05** | Etapa 2 — Silver/Gold + EDA (MongoDB-only) | ✅ |
| **26/05** | Etapa 3 — Modelos de Machine Learning | ✅ |
| **02/06** | Apresentação Final do Projeto | 🔜 |

---

## 📖 Dicionário de Dados

> Organizado por fonte e camada Medallion. Colunas prefixadas com `inmet_` na Gold são cópias das homônimas da Silver INMET, juntadas via KDTree ao dataset de empreendimentos.

### 🔌 ANEEL / SIGA — `bronze_siga` · `silver_siga`

| Coluna | Tipo | Descrição |
| :--- | :--- | :--- |
| `CodCEG` | string | Código de Empreendimento de Geração (ANEEL) — identificador único |
| `IdeNucleoCEG` | string | Identificador do Núcleo do Código de Empreendimento de Geração |
| `NomEmpreendimento` | string | Nome do empreendimento de geração de energia |
| `SigTipoGeracao` | string | Tipo de geração: UFV · UTE · EOL · UHE · PCH · CGH… |
| `SigUFPrincipal` | string | Unidade federativa (estado) onde o empreendimento está localizado |
| `DscFaseUsina` | string | Fase atual: **Operação · Construção · Construção não iniciada** _(target da classificação)_ |
| `MdaPotenciaOutorgadaKw` | float | Potência outorgada (autorizada pela ANEEL) em kW |
| `MdaPotenciaFiscalizadaKw` | float | Potência fiscalizada em kW _(target da regressão)_ |
| `MdaGarantiaFisicaKw` | float | Garantia física do empreendimento em kW |
| `DscOrigemCombustivel` | string | Origem do combustível: Hídrica · Solar · Eólica · Fóssil · Biomassa… |
| `NomeCombustivel` | string | Nome detalhado do combustível utilizado (quando aplicável) |
| `DscTipoOutorga` | string | Tipo de outorga concedida: Autorização · Concessão… |
| `DscSubBacia` | string | Sub-bacia hidrográfica (empreendimentos hídricos) |
| `IdcGeracaoQualificada` | string | Indicador de geração qualificada conforme critérios ANEEL |
| `DatEntradaOperacao` | date | Data de entrada em operação comercial |
| `DatGeracaoConjuntoDados` | date | Data de geração do conjunto de dados pela ANEEL |
| `DatInicioVigencia` | date | Data de início da vigência da outorga |
| `DatFimVigencia` | date | Data de fim da vigência da outorga |
| `DatPublicacao` | date | Data de publicação dos dados do empreendimento |
| `NumCoordNEmpreendimento` | string | Coordenada Norte (latitude) **original** da base Bronze — texto bruto da ANEEL |
| `NumCoordEEmpreendimento` | string | Coordenada Leste (longitude) **original** da base Bronze — texto bruto da ANEEL |
| `lat` | float | Latitude padronizada WGS-84 — derivada de `NumCoordN` na Silver |
| `lon` | float | Longitude padronizada WGS-84 — derivada de `NumCoordE` na Silver |
| `eh_renovavel` | bool | Flag: fonte renovável (solar, eólica, hídrica, biomassa) |

### 🌦️ INMET — `bronze_inmet_estacoes` · `silver_inmet`

| Coluna | Tipo | Descrição |
| :--- | :--- | :--- |
| `REGIAO` | string | Região do Brasil onde a estação meteorológica está localizada |
| `ESTACAO` | string | Nome da estação meteorológica |
| `CODIGO (WMO)` | string | Código da estação conforme Organização Meteorológica Mundial (OMM) |
| `LATITUDE` | float | Latitude da estação meteorológica |
| `LONGITUDE` | float | Longitude da estação meteorológica |
| `ALTITUDE` | float | Altitude da estação em metros |
| `DATA_INICIO` | date | Início do período de observação |
| `DATA_FIM` | date | Fim do período de observação |
| `temp_media` | float | Temperatura média do ar (°C) — agregada 1T 2026 |
| `temp_std` | float | Desvio padrão da temperatura do ar (°C) |
| `temp_max` | float | Temperatura máxima registrada no período (°C) |
| `temp_min` | float | Temperatura mínima registrada no período (°C) |
| `umid_media` | float | Umidade relativa média do ar (%) |
| `prec_total` | float | Precipitação total acumulada no período (mm) |
| `prec_max_h` | float | Precipitação máxima horária registrada (mm) |
| `radiacao_med` | float | Radiação global média (kJ/m²) |
| `vento_med` | float | Velocidade média do vento (m/s) |
| `rajada_max` | float | Velocidade máxima da rajada de vento (m/s) |
| `n_obs` | int | Número de observações horárias válidas usadas na agregação |

### 📡 ANATEL — `silver_anatel_uf`

| Coluna | Tipo | Descrição |
| :--- | :--- | :--- |
| `anatel_compartilhamento` | int | Menções à UF em contratos de compartilhamento de infraestrutura (postes, dutos, torres) |
| `anatel_interconexao` | int | Menções à UF em contratos de interconexão entre prestadoras |
| `anatel_mvno` | int | Menções à UF em contratos de operadoras virtuais (MVNO) |
| `anatel_ran_sharing` | int | Menções à UF em contratos de compartilhamento de rede (RAN Sharing 4G/5G) |
| `anatel_credenciadas` | int | Menções à UF em registros de empresas credenciadas vigentes |
| `anatel_total` | int | Soma de todas as menções ANATEL por UF — **proxy de cobertura de telecomunicações** |

### 🏢 Datacenters — `bronze_datacenters` · `silver_datacenters`

| Coluna | Tipo | Descrição |
| :--- | :--- | :--- |
| `name` | string | Nome do datacenter |
| `company` | string | Empresa operadora do datacenter |
| `city` | string | Cidade de localização |
| `state` | string | Estado de localização |
| `country` | string | País de localização |
| `address` | string | Endereço completo do datacenter |
| `LATITUDE` | float | Latitude geocodificada (dicionário de cidades brasileiras — Silver) |
| `LONGITUDE` | float | Longitude geocodificada (dicionário de cidades brasileiras — Silver) |

### 🥇 Features computadas — `gold_empreendimentos_features` · `gold_ranking_atratividade_dc`

> Colunas geradas pela Etapa 2 (join geoespacial) e Etapa 3 (ML). As prefixadas com `inmet_` são as variáveis do Silver INMET associadas à estação mais próxima de cada empreendimento.

| Coluna | Tipo | Descrição |
| :--- | :--- | :--- |
| `inmet_dist_km` | float | Distância em km até a estação INMET mais próxima (cKDTree) |
| `inmet_temp_media` | float | Temperatura média (°C) da estação INMET mais próxima |
| `inmet_temp_std` | float | Desvio padrão da temperatura (°C) da estação mais próxima |
| `inmet_umid_media` | float | Umidade relativa média (%) da estação mais próxima |
| `inmet_prec_total` | float | Precipitação total (mm) da estação mais próxima |
| `inmet_prec_max_h` | float | Precipitação máxima horária (mm) da estação mais próxima |
| `inmet_radiacao_med` | float | Radiação global média (kJ/m²) da estação mais próxima |
| `inmet_vento_med` | float | Velocidade média do vento (m/s) da estação mais próxima |
| `inmet_rajada_max` | float | Rajada máxima de vento (m/s) da estação mais próxima |
| `dc_proximos` | int | Nº de datacenters existentes a ≤ 100 km do empreendimento |
| `potencia_fiscalizada_predita_kw` | float | Potência fiscalizada predita pelo modelo Random Forest de regressão |
| `prob_operacao` | float | Probabilidade de fase "Operação" predita pelo classificador XGBoost |
| `indice_atratividade_dc` | float | Score composto [0–1] de atratividade para localização de datacenter por UF |

---

## ⚖️ Critérios de Seleção

Com a ascensão da IA, a demanda por infraestrutura tecnológica tornou-se prioridade global. Selecionamos datasets que correlacionam **energia**, **clima** e **telecom** para gerar um índice de atratividade para datacenters, respondendo: *"Qual o local ideal para construir um datacenter no Brasil?"*

---

## 📜 Licença

Finalidade estritamente acadêmica. Dados de domínio público regidos pela licença **ODbL** (ANEEL/INMET/ANATEL).
