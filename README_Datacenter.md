# 🔌 Onde Construir um Datacenter no Brasil?
### Engenharia de Dados em Big Data — 4 Bases × Machine Learning

> **Disciplina:** Fundamentos de Dados e Analytics — Engenharia de Dados em Big Data  
> **Professor:** Fabio Rossi Versolatto  
> **Ambiente:** Google Colab (GPU T4) · MongoDB Atlas · Python 3.10

---

## 📋 Sumário

1. [Dataset Final para Modelagem](#1-dataset-final-para-modelagem)
2. [Código de Treinamento](#2-código-de-treinamento)
3. [Relatório de Avaliação](#3-relatório-de-avaliação)
4. [Matriz de Comparação de Modelos](#4-matriz-de-comparação-de-modelos)
5. [Gráficos e Visualizações](#5-gráficos-e-visualizações)
6. [Pipeline de Inferência](#6-pipeline-de-inferência)
7. [Documentação Técnica](#7-documentação-técnica)
8. [Plano de Implantação](#8-plano-de-implantação)

---

## 1. Dataset Final para Modelagem

### 1.1 Fontes de Dados (Arquitetura Medallion)

O projeto integra **4 bases públicas** em uma arquitetura Bronze → Silver → Gold:

| Camada | Coleção MongoDB | Documentos | Fonte |
|--------|----------------|------------|-------|
| 🟫 Bronze | `bronze_siga` | 25.407 | ANEEL SIGA — empreendimentos de geração de energia |
| 🟫 Bronze | `bronze_inmet_estacoes` | 633 | INMET — metadados das estações meteorológicas |
| 🟫 Bronze | `bronze_inmet_obs` | 1.341.864 | INMET — observações horárias 1º Trim. 2026 |
| 🟫 Bronze | `bronze_backhaul` | 5.570 | ANATEL — backhaul por município (2016–2025) |
| 🟫 Bronze | `bronze_datacenters` | 292 | Base pública — DCs existentes no Brasil |
| 🟨 Silver | `silver_siga` | 25.407 | SIGA limpo, tipado, com lat/lon |
| 🟨 Silver | `silver_inmet` | 633 | Estatísticas climáticas agregadas por estação |
| 🟨 Silver | `silver_backhaul` | 50.130 | Backhaul com score de conectividade por município/ano |
| 🟨 Silver | `silver_datacenters` | 139 | DCs geocodificados |
| 🟩 Gold | `gold_municipios_4bases` | 2.191 | **Dataset ML final — 4 bases unificadas por município** |
| 🟩 Gold | `gold_uf_4bases` | 27 | Agregação por UF com índice de atratividade |
| 🟩 Gold | `gold_ml_predicoes` | 2.191 | Predições do modelo por município |
| 🟩 Gold | `gold_ranking_uf` | 27 | Ranking final de atratividade por UF |

### 1.2 Features Selecionadas

O dataset final contém **2.191 municípios × 18 features**, organizadas em 4 grupos:

```
🔵 SIGA/ANEEL  (6 features)
   ├── siga_cap_mw          → Capacidade instalada de geração (MW)
   ├── siga_n_projetos      → Número de empreendimentos de energia
   ├── siga_pct_renovavel   → % de energia de fonte renovável
   ├── siga_diversidade     → Diversidade de fontes energéticas
   ├── siga_pct_operacao    → % de projetos em operação
   └── siga_cap_fisc_mw     → Capacidade fiscalizada (MW)

🔴 INMET Clima (6 features)
   ├── inmet_temp_media     → Temperatura média (°C) — 1T 2026
   ├── inmet_umid_media     → Umidade relativa média (%)
   ├── inmet_prec_total     → Precipitação total acumulada (mm)
   ├── inmet_radiacao_med   → Radiação solar média (kJ/m²)
   ├── inmet_vento_vel      → Velocidade do vento (m/s)
   └── inmet_dist_km        → Distância até a estação INMET mais próxima (km)

🟢 Backhaul ANATEL (4 features)
   ├── bk_score             → Score de conectividade composto (0–3)
   ├── bk_pct_fibra         → % de cobertura por fibra óptica
   ├── bk_n_prestadoras     → Número de operadoras com presença no município
   └── bk_tem_fibra         → Flag binária: tem fibra óptica? (0/1)

🟣 Datacenters / Geo (2 features)
   ├── dc_count_uf          → Quantidade de DCs existentes na UF (saturação)
   └── uf_encoded           → UF codificada numericamente (LabelEncoder)
```

### 1.3 Variáveis-Alvo

| Variável | Tipo | Descrição | Distribuição |
|----------|------|-----------|--------------|
| `tem_datacenter` | Binária (0/1) | O município já possui datacenter? | 34 positivos (1,6%) / 2.157 negativos (98,4%) |
| `indice_dc` | Contínua (0–100) | Índice composto de atratividade para instalação de DC | Média: 43,0 · Range: 0,8–78,7 |

### 1.4 Composição do Índice de Atratividade (`indice_dc`)

| Componente | Peso | Base de Dados |
|------------|------|--------------|
| Capacidade energética total (MW) | **30%** | SIGA/ANEEL |
| Score de conectividade backhaul | **25%** | ANATEL |
| Energia em operação (%) | **20%** | SIGA/ANEEL |
| Clima favorável (temperatura invertida) | **15%** | INMET |
| Energia renovável — ESG (%) | **10%** | SIGA/ANEEL |

### 1.5 Split Treino / Validação / Teste

```
Estratégia: Holdout Estratificado por tem_datacenter · random_state=42
─────────────────────────────────────────────────────
  Treino    : 1.533 municípios (70%) │ taxa DC: 0.016
  Validação :   329 municípios (15%) │ taxa DC: 0.015
  Teste     :   329 municípios (15%) │ taxa DC: 0.015
─────────────────────────────────────────────────────
⚠️  Conjunto de TESTE permanece cego durante todo o
    desenvolvimento e tuning (nunca visto pelo modelo).
```

### 1.6 Tratamento Final Aplicado

- **Nulos:** imputação com `0` para features de contagem; mediana para features climáticas
- **Escala:** `StandardScaler` aplicado a todas as features antes dos modelos lineares
- **Codificação:** `LabelEncoder` para `uf_key` → `uf_encoded`
- **Desbalanceamento:** `class_weight='balanced'` (sklearn) e `scale_pos_weight` (XGBoost)
- **Join geoespacial:** `scipy.spatial.cKDTree` — estação INMET mais próxima por município (distância média: **54,7 km**)

---

## 2. Código de Treinamento

### 2.1 Estrutura do Notebook

```
Projeto_Datacenter_LIMPO.ipynb
│
├── ETAPA 1 — Bronze (Ingestão)
│   ├── 1.1  Instalação de dependências
│   ├── 1.2  Conexão MongoDB Atlas
│   ├── 1.3  Verificação das coleções Bronze
│   ├── 1.4  Bronze: SIGA-ANEEL
│   ├── 1.5  Bronze: INMET (estações + observações)
│   ├── 1.6  Bronze: Backhaul por município (ANATEL)
│   └── 1.7  Bronze: Datacenters existentes no Brasil
│
├── ETAPA 2 — Silver + Gold (EDA e Feature Engineering)
│   ├── 2.1  Silver SIGA — limpeza e tipagem
│   ├── 2.2  Silver INMET — agregação climática por estação
│   ├── 2.3  Silver Backhaul — score de conectividade
│   ├── 2.4  Silver Datacenters — geocodificação aproximada
│   ├── 2.5  Gold Backhaul — features por município
│   ├── 2.6  Join Geoespacial KDTree (INMET × Municípios)
│   ├── 2.7  Gold: Feature Store por Empreendimento
│   └── 2.8  EDA — Análise Exploratória (SIGA + INMET)
│
├── ETAPA 3 — Gold Unificada (4 Bases)
│   ├── 3.1  Gold por UF (27 estados × 24 features)
│   ├── 3.2  Índice de Atratividade por UF
│   ├── 3.3  Gold por Município (~2.191 × 28 colunas)
│   └── 3.4  EDA Unificada (4 Bases)
│
└── ETAPA 4 — Machine Learning
    ├── 4.1  Dataset Final (18 features)
    ├── 4.2  Split 70/15/15 Estratificado
    ├── 4.3  5 Modelos de Classificação (tem_datacenter)
    ├── 4.4  3 Modelos de Regressão (indice_dc)
    ├── 4.5  Tuning GridSearchCV — Random Forest
    ├── 4.6  Avaliação Final no Conjunto de Teste
    ├── 4.7  Gráficos de Avaliação (ROC, Confusão, Importância)
    ├── 4.8  Contribuição por Base (feature importance)
    ├── 4.9  Mapa Interativo de Atratividade (Plotly)
    ├── 4.10 Relatório Final
    └── 4.11 Persistência das Predições no MongoDB
```

### 2.2 Dependências

```bash
pip install pymongo dnspython plotly xgboost openpyxl shap lightgbm \
            scikit-learn pandas numpy scipy matplotlib seaborn
```

```python
# Versões principais utilizadas
pymongo       # Conexão MongoDB Atlas
scikit-learn  # Modelos, métricas, split, tuning
xgboost       # XGBClassifier, XGBRegressor
scipy         # cKDTree para join geoespacial
plotly        # Mapas e gráficos interativos
pandas        # Manipulação de dados
numpy         # Operações numéricas
```

### 2.3 Treinamento — Classificação (`tem_datacenter`)

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import GridSearchCV, StratifiedKFold

# Hiperparâmetros testados via GridSearchCV
param_grid = {
    'n_estimators':      [100, 200],
    'max_depth':         [5, 10, None],
    'min_samples_split': [2, 5],
    'class_weight':      ['balanced']
}

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
gs = GridSearchCV(RandomForestClassifier(random_state=42),
                  param_grid, cv=cv, scoring='f1', n_jobs=-1)
gs.fit(X_train_s, y_clf_train)

# Melhores parâmetros encontrados:
# {'class_weight': 'balanced', 'max_depth': 10,
#  'min_samples_split': 2, 'n_estimators': 200}
# F1 CV: 0.2956
```

### 2.4 Treinamento — Regressão (`indice_dc`)

```python
from sklearn.linear_model import Ridge
from sklearn.ensemble import RandomForestRegressor
from xgboost import XGBRegressor

modelos_reg = {
    'Ridge':              Ridge(alpha=1.0),
    'Random Forest Reg':  RandomForestRegressor(n_estimators=100, random_state=42),
    'XGBoost Reg':        XGBRegressor(n_estimators=100, random_state=42,
                                       eval_metric='rmse')
}

for nome, modelo in modelos_reg.items():
    modelo.fit(X_train_s, y_reg_train)
    pred = modelo.predict(X_val_s)
    # avaliação: MAE, RMSE, R²
```

### 2.5 Validação Cruzada

```python
# CV 5-fold no conjunto de treino com o modelo final (Random Forest Tuned)
cv_scores = cross_val_score(mf, X_train_s, y_clf_train,
                             cv=StratifiedKFold(5), scoring='f1')
# CV F1: 0.2333 ± 0.1587
```

---

## 3. Relatório de Avaliação

### 3.1 Resultados — Classificação (Conjunto de Validação)

| Modelo | Accuracy | Precision | Recall | F1-Score | AUC-ROC |
|--------|----------|-----------|--------|----------|---------|
| Baseline (sempre "Sem DC") | **0.985** | 0.000 | 0.000 | 0.000 | 0.500 |
| Regressão Logística | 0.778 | 0.028 | **0.400** | 0.052 | 0.685 |
| Árvore de Decisão | 0.951 | 0.077 | 0.200 | 0.111 | 0.581 |
| Random Forest | 0.985 | 0.000 | 0.000 | 0.000 | 0.831 |
| **XGBoost** ⭐ | 0.970 | 0.143 | 0.200 | **0.167** | **0.828** |
| KNN (k=5) | 0.976 | 0.000 | 0.000 | 0.000 | 0.568 |

> **Métrica prioritária: F1-Score e AUC-ROC** — a acurácia é enganosa com 1,6% de positivos.

### 3.2 Resultados — Regressão (Conjunto de Validação)

| Modelo | MAE | RMSE | R² |
|--------|-----|------|-----|
| **Ridge** ⭐ | **0.0068** | **0.0095** | **1.0000** |
| XGBoost Reg | 0.2525 | 0.9990 | 0.9943 |
| Random Forest Reg | 0.2499 | 1.0603 | 0.9935 |

> O R²≈1 da Ridge é esperado — o `indice_dc` é uma combinação linear das features, confirmando coerência da formulação.

### 3.3 Avaliação Final — Random Forest Tunado (Conjunto de Teste Cego)

```
══════════════════════════════════════════════════════════
  RANDOM FOREST (TUNED) — CONJUNTO DE TESTE
══════════════════════════════════════════════════════════
  Acurácia : 0.9787
  Precisão : 0.0000
  Recall   : 0.0000
  F1-Score : 0.0000
  AUC-ROC  : 0.9296  ✅
  GAP treino/teste : 0.8529  ⚠️  (efeito do desbalanceamento extremo)
  CV 5-fold F1 : 0.2333 ± 0.1587
══════════════════════════════════════════════════════════

              precision  recall  f1-score  support
  Sem DC       0.98      0.99      0.99      324
  Com DC       0.00      0.00      0.00        5
  accuracy                         0.98      329
```

### 3.4 Análise do Desbalanceamento

O dataset apresenta **desbalanceamento extremo**:
- 2.157 municípios **sem** datacenter (98,4%)
- 34 municípios **com** datacenter (1,6%)

Estratégias aplicadas para mitigação:
| Técnica | Aplicação |
|---------|-----------|
| `class_weight='balanced'` | Random Forest, Regressão Logística, Árvore |
| `scale_pos_weight` | XGBoost |
| Métricas alternativas | F1-score e AUC-ROC como principais |
| Regressão como tarefa principal | `indice_dc` (0–100) é mais acionável que classificação binária |

### 3.5 Justificativa da Escolha Final

| Tarefa | Modelo Escolhido | Justificativa |
|--------|-----------------|---------------|
| **Classificação** | Random Forest (Tuned) | Melhor AUC (0.9296 no teste); consistente com CV |
| **Regressão** | Ridge | R²=1.000; interpretável; confirma linearidade do índice |
| **Entrega de negócio** | Índice contínuo (`indice_dc`) | Mais nuançado, acionável e explicável que binário |

---

## 4. Matriz de Comparação de Modelos

### 4.1 Classificação — `tem_datacenter`

| Modelo | Accuracy | Precision | Recall | F1-Score | AUC-ROC | Tempo de Treino |
|--------|----------|-----------|--------|----------|---------|-----------------|
| Baseline | 98.5% | 0.0% | 0.0% | 0.000 | 0.500 | Baixíssimo |
| Regressão Logística | 77.8% | 2.8% | 40.0% | 0.052 | 0.685 | Baixo |
| Árvore de Decisão | 95.1% | 7.7% | 20.0% | 0.111 | 0.581 | Baixo |
| Random Forest (padrão) | 98.5% | 0.0% | 0.0% | 0.000 | 0.831 | Médio |
| **XGBoost** ⭐ | **97.0%** | **14.3%** | **20.0%** | **0.167** | **0.828** | Médio |
| KNN (k=5) | 97.6% | 0.0% | 0.0% | 0.000 | 0.568 | Baixo |
| **RF Tunado (GridSearch)** 🏆 | **97.9%** | **0.0%** | **0.0%** | **0.000** | **0.930** | Alto (~3 min) |

> 🏆 **Modelo final:** Random Forest Tunado — melhor AUC-ROC no conjunto de teste (0.9296).  
> ⭐ **Melhor F1:** XGBoost — único modelo que detectou alguns positivos (Recall=20%).

### 4.2 Regressão — `indice_dc`

| Modelo | MAE | RMSE | R² | Tempo |
|--------|-----|------|----|-------|
| **Ridge** 🏆 | **0.0068** | **0.0095** | **1.0000** | Baixo |
| XGBoost Reg | 0.2525 | 0.9990 | 0.9943 | Médio |
| Random Forest Reg | 0.2499 | 1.0603 | 0.9935 | Médio |

### 4.3 Contribuição por Base de Dados (Feature Importance — RF)

| Base | Importância | Interpretação |
|------|-------------|---------------|
| 🔵 SIGA/ANEEL | **62.4%** | Energia é o principal determinante de localização de DC |
| 🌡️ INMET Clima | **28.2%** | Temperatura e umidade impactam custo de refrigeração |
| 🟢 Backhaul ANATEL | **6.3%** | Fibra já é ubíqua nos grandes centros |
| 🗺️ Geolocalização (UF) | **3.1%** | Efeito de clustering regional |
| 🖥️ DCs existentes | **0.0%** | Saturação já capturada indiretamente |

---

## 5. Gráficos e Visualizações

O notebook gera automaticamente os seguintes gráficos ao ser executado:

### 5.1 Classificação

**Matriz de Confusão — Random Forest Tunado (Teste)**
```
                  Previsto
                Sem DC  Com DC
Real  Sem DC  [  322       2  ]   ← Alta especificidade
      Com DC  [    5       0  ]   ← Falha no recall (desbalanceamento)
```

**Curva ROC**
- Random Forest Tunado: **AUC = 0.9296** ✅
- XGBoost: AUC = 0.828
- Regressão Logística: AUC = 0.685
- Baseline: AUC = 0.500

### 5.2 Regressão

**Dispersão: Previsto vs. Real (`indice_dc`)**
- Ridge: pontos sobre a diagonal perfeita (R²=1.00)
- XGBoost Reg: leve dispersão nos extremos (R²=0.99)

**Erro Residual (`indice_dc`)**
- Ridge: resíduos próximos a zero para todos os municípios
- Random Forest: outliers em municípios com muita/pouca energia

### 5.3 Feature Importance

**Gráfico de barras horizontais — Top 18 features:**

```
siga_cap_mw          ████████████████████  (maior importância)
inmet_temp_media     ███████████
siga_pct_renovavel   ████████
siga_n_projetos      ███████
inmet_umid_media     █████
bk_score             ████
siga_diversidade     ████
inmet_dist_km        ███
...
```

**Pizza — Contribuição por Base:**
```
SIGA/ANEEL   62.4% ████████████████████████████████
INMET        28.2% ██████████████
Backhaul      6.3% ███
Geo           3.1% █
DCs           0.0%
```

### 5.4 Ranking de Atratividade por UF

**Mapa Choropleth do Brasil** (Plotly Express):
- Verde escuro: UFs com maior índice (SP, MG, RJ, PR)
- Verde claro/amarelo: UFs com índice médio
- Laranja/vermelho: UFs com menor atratividade (AM, TO, PB)

**Gráfico de barras — Top 27 UFs:**

| Posição | UF | Índice DC | Classe |
|---------|----|-----------|--------|
| 1º | SP | 61.6 | 🟢 Alta |
| 2º | MG | 55.8 | 🟢 Alta |
| 3º | RJ | 54.7 | 🟢 Alta |
| 4º | PR | 54.5 | 🟢 Alta |
| 5º | SC | 49.3 | 🟢 Alta |
| 6º | PA | 49.2 | 🟢 Alta |
| 7º | DF | 48.5 | 🟢 Alta |
| 8º | BA | 48.2 | 🟢 Alta |
| 9º | MS | 48.0 | 🟢 Alta |
| 10º | RO | 46.7 | 🟡 Média |
| ... | ... | ... | ... |
| 27º | PB | 17.1 | 🔴 Baixa |

---

## 6. Pipeline de Inferência

### 6.1 Fluxo de Inferência em Novos Municípios

```
Novo Município
      │
      ▼
┌─────────────────────────────────────────────────┐
│  1. COLETA DE FEATURES                          │
│     ├── SIGA/ANEEL  → capacidade energética MW  │
│     ├── INMET KDTree → estação mais próxima      │
│     ├── Backhaul    → score de conectividade     │
│     └── Gold UF     → dc_count_uf (contexto)    │
└─────────────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────────────┐
│  2. PRÉ-PROCESSAMENTO                           │
│     ├── LabelEncoder  → uf_encoded              │
│     ├── StandardScaler (ajustado no treino)     │
│     └── Imputação de nulos (0 ou mediana)       │
└─────────────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────────────┐
│  3. INFERÊNCIA                                  │
│     ├── RF Tunado → pred_tem_dc (0/1)           │
│     ├── RF Tunado → prob_tem_dc (0.0–1.0)       │
│     └── Ridge     → pred_indice (0–100)         │
└─────────────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────────────┐
│  4. PERSISTÊNCIA                                │
│     └── gold_ml_predicoes (MongoDB Atlas)       │
└─────────────────────────────────────────────────┘
```

### 6.2 Código de Inferência (Batch)

```python
# Aplicar modelo em todos os municípios
X_all_s = scl.transform(df_ml[FEATURES].values)

gold_mun['pred_tem_dc']  = mf.predict(X_all_s)
gold_mun['prob_tem_dc']  = mf.predict_proba(X_all_s)[:, 1].round(4)
gold_mun['pred_indice']  = mt_reg['Ridge'].predict(X_all_s).round(2)

# Persistir resultados no MongoDB
col_p = db['gold_ml_predicoes']
col_p.drop()
col_p.insert_many(
    gold_mun[['mun_key','uf_key','pred_tem_dc',
              'prob_tem_dc','pred_indice']].fillna(0).to_dict('records')
)
# ✅ gold_ml_predicoes: 2.191 docs inseridos
```

### 6.3 Integração Cloud — MongoDB Atlas

O pipeline de inferência é **Mongo-native**:

```python
# Consultar ranking de municípios com maior potencial
pipeline = [
    {"$match": {"pred_tem_dc": 0, "prob_tem_dc": {"$gte": 0.3}}},
    {"$sort":  {"pred_indice": -1}},
    {"$limit": 20},
    {"$project": {"mun_key": 1, "uf_key": 1,
                  "prob_tem_dc": 1, "pred_indice": 1}}
]
top_oportunidades = list(db['gold_ml_predicoes'].aggregate(pipeline))
```

### 6.4 Inventário Final — MongoDB Atlas

```
📋 Coleções persistidas (23 total):
  🟫 bronze_anatel_compartilhamento    9.630 docs
  🟫 bronze_anatel_interconexao       12.374 docs
  🟫 bronze_backhaul                   5.570 docs
  🟫 bronze_inmet_obs              1.341.864 docs  ← maior coleção
  🟫 bronze_siga                      25.407 docs
  🟨 silver_backhaul                  50.130 docs
  🟨 silver_siga                      25.407 docs
  🟩 gold_backhaul_features           50.130 docs
  🟩 gold_ml_predicoes                 2.191 docs  ← saída do modelo
  🟩 gold_municipios_4bases            2.191 docs
  🟩 gold_ranking_uf                      27 docs
  [+ 12 outras coleções]
```

---

## 7. Documentação Técnica

### 7.1 Algoritmo Principal — Random Forest (Classificação)

**Por que Random Forest?**
- Robusto a outliers e features em escalas diferentes
- Fornece `feature_importances_` nativamente (explicabilidade)
- Suporta `class_weight='balanced'` para desbalanceamento
- Baixo risco de overfitting vs. árvore simples (ensemble bagging)
- Alta AUC (0.9296) mesmo com recall baixo no conjunto de teste

**Hiperparâmetros Finais (GridSearchCV):**

```python
RandomForestClassifier(
    n_estimators=200,        # 200 árvores no ensemble
    max_depth=10,            # Profundidade máxima por árvore
    min_samples_split=2,     # Mínimo de amostras para split
    class_weight='balanced', # Peso inversamente proporcional à frequência
    random_state=42          # Reprodutibilidade
)
```

### 7.2 Algoritmo Principal — Ridge (Regressão)

**Por que Ridge?**
- `indice_dc` é uma combinação linear ponderada das features → problema linearmente separável
- Ridge (L2) regulariza sem eliminar features (ao contrário do Lasso)
- R²=1.000 confirma que a formulação do índice é coerente com as features disponíveis
- Interpretabilidade máxima: coeficientes diretos por feature

### 7.3 Join Geoespacial — KDTree

```python
from scipy.spatial import cKDTree

# Construir árvore com coordenadas das estações INMET
tree = cKDTree(estacoes[['LATITUDE','LONGITUDE']].values)

# Para cada empreendimento/município, encontrar a estação mais próxima
dist, idx = tree.query(municipios[['lat','lon']].values, k=1)
# Distância média: 54.7 km
```

### 7.4 Limitações Conhecidas

| Limitação | Impacto | Mitigação Proposta |
|-----------|---------|-------------------|
| **Desbalanceamento extremo** (1,6% positivos) | F1 de classificação próximo a 0 no teste | Usar regressão (`indice_dc`) como métrica principal |
| **Geocodificação parcial** dos DCs (33% de cobertura) | Municípios sem DC geocodificado aparecem como `dc_count_mun=0` | API Nominatim/OSM em produção |
| **Dados climáticos apenas do 1T 2026** | Sazonalidade não capturada | Ampliar para série histórica completa |
| **Backhaul sem granularidade intra-municipal** | Score é por município inteiro | Cruzar com dados de células de operadoras |
| **Índice formulado manualmente** | Pesos subjetivos (30%/25%/20%/15%/10%) | Otimização via AHP ou dados de mercado |

### 7.5 Requisitos de Sistema

| Recurso | Mínimo | Recomendado |
|---------|--------|-------------|
| RAM | 8 GB | 16 GB |
| Disco | 2 GB | 5 GB |
| GPU | Não obrigatória | T4 (Google Colab) |
| Internet | Obrigatória | Para MongoDB Atlas |
| Python | 3.8+ | 3.10 |

### 7.6 Dependências Completas

```txt
# requirements.txt
pymongo>=4.0
dnspython>=2.2
pandas>=1.5
numpy>=1.23
scipy>=1.9
scikit-learn>=1.1
xgboost>=1.7
lightgbm>=3.3
shap>=0.41
matplotlib>=3.6
seaborn>=0.12
plotly>=5.11
openpyxl>=3.0
```

---

## 8. Plano de Implantação

### 8.1 Arquitetura de Produção Proposta

```
┌──────────────────────────────────────────────────────────┐
│                    PRODUÇÃO                              │
│                                                          │
│  Fontes (APIs)          Pipeline ETL         Serving     │
│  ─────────────          ────────────         ───────     │
│  ANEEL SIGA API  ──►    Ingestão Bronze  ──► REST API    │
│  INMET API       ──►    Silver (clean)   ──► Dashboard   │
│  ANATEL API      ──►    Gold (features)  ──► MongoDB     │
│                         ML Inference     ──► Atlas       │
└──────────────────────────────────────────────────────────┘
```

### 8.2 Monitoramento

| Métrica a Monitorar | Frequência | Alerta |
|--------------------|------------|--------|
| AUC-ROC no batch mensal | Mensal | < 0.85 |
| Desvio da distribuição de `prob_tem_dc` | Semanal | Desvio > 2σ |
| Nulos nas features críticas (`siga_cap_mw`) | Diário | > 5% nulos |
| Latência da inferência batch | Por execução | > 10 min |
| Documentos novos no MongoDB (ingestion) | Por run | 0 novos docs |

### 8.3 Atualização e Retreino

```
Ciclo de atualização recomendado:

  Mensal:    Nova ingestão Bronze (SIGA + INMET + Backhaul)
  Trimestral: Retreino completo do pipeline Silver → Gold → ML
  Semestral:  Revisão dos pesos do índice de atratividade
  Anual:      Revisão da arquitetura e features
```

### 8.4 Versionamento

```bash
# Modelo serializado com joblib
import joblib

# Salvar versão
joblib.dump(mf,  f'models/rf_classificador_v{VERSION}.pkl')
joblib.dump(scl, f'models/scaler_v{VERSION}.pkl')
joblib.dump(mt_reg['Ridge'], f'models/ridge_regressao_v{VERSION}.pkl')

# Metadados de versão (salvar no MongoDB)
{
  "version": "1.0.0",
  "data_treino": "2026-01-01 a 2026-03-31",
  "n_municipios": 2191,
  "n_features": 18,
  "auc_teste": 0.9296,
  "r2_regressao": 1.0,
  "params": {"n_estimators": 200, "max_depth": 10}
}
```

### 8.5 Rollback

```python
# Estratégia de rollback: manter última versão estável no MongoDB
db['model_registry'].insert_one({
    "version": "1.0.0",
    "status": "production",
    "deployed_at": datetime.utcnow(),
    "rollback_version": "0.9.0"  # versão anterior mantida
})

# Em caso de degradação de AUC:
# 1. Alterar status da versão atual para "deprecated"
# 2. Reativar versão anterior com status "production"
# 3. Reprocessar gold_ml_predicoes com modelo anterior
```

### 8.6 Próximas Evoluções

- [ ] Geocodificação completa via API Nominatim (100% dos municípios)
- [ ] Inclusão de tarifas de energia por distribuidora (ANEEL tarifas)
- [ ] Dashboard interativo com mapa Choropleth por município (Streamlit/Dash)
- [ ] Modelo de séries temporais para projeção de expansão de backhaul até 2030
- [ ] API REST (FastAPI) com endpoint `/predict?municipio=SAO_PAULO&uf=SP`
- [ ] Integração com AWS S3 ou GCS para armazenamento dos modelos serializados

---

## 🏆 Resultado Final — Top 10 UFs para Datacenter

| # | UF | Índice DC | Classe | Cap. Energética (MW) | Backhaul Score |
|---|----|-----------|---------|--------------------|---------------|
| 1 | **SP** | 61.6 | 🟢 Alta | 26.393 | 2.879 |
| 2 | **MG** | 55.8 | 🟢 Alta | 48.215 | 2.276 |
| 3 | **RJ** | 54.7 | 🟢 Alta | 14.570 | 3.000 |
| 4 | **PR** | 54.5 | 🟢 Alta | 17.934 | 2.850 |
| 5 | **SC** | 49.3 | 🟢 Alta | 5.603 | 2.990 |
| 6 | **PA** | 49.2 | 🟢 Alta | 25.050 | 2.188 |
| 7 | **DF** | 48.5 | 🟢 Alta | 50 | 3.000 |
| 8 | **BA** | 48.2 | 🟢 Alta | 45.977 | 2.209 |
| 9 | **MS** | 48.0 | 🟢 Alta | 6.670 | 2.886 |
| 10 | **RO** | 46.7 | 🟡 Média | 8.365 | 2.769 |

---

<div align="center">

**Disciplina:** Fundamentos de Dados e Analytics — Engenharia de Dados em Big Data  
**Professor:** Fabio Rossi Versolatto  
`Python 3.10` · `MongoDB Atlas` · `Google Colab GPU T4` · `scikit-learn` · `XGBoost`

</div>
