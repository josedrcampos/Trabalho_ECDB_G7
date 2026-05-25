# Trabalho Prático 2 — Espetroscopia Raman
### Extração de Conhecimento de Dados · Grupo 7

---

## Descrição

Análise de espetros Raman de 10 compostos orgânicos, com o objetivo de explorar padrões espetrais, comparar compostos e construir modelos de classificação. O trabalho cobre todo o pipeline — desde o pré-processamento até à interpretação espetroscópica dos modelos.

---

## Compostos Analisados

| # | Composto | Família |
|---|---|---|
| 1 | 1,3-Dimethyl-2-imidazolidinone (1,3-DMI) | Heterociclo |
| 2 | 2-Propanol | Álcool |
| 3 | 2,2-Dimethoxypropane (2,2-DMP) | Acetal |
| 4 | 4-Methyl-2-pentanone (MIBK) | Cetona |
| 5 | Acetic acid | Ácido |
| 6 | Acetone | Cetona |
| 7 | Acetonitrile | Nitrilo |
| 8 | Benzaldehyde | Aromático |
| 9 | Benzyl bromide | Aromático |
| 10 | Butyl acetate | Éster |

---

## Ficheiros

```
raman_grupo7.ipynb   # Notebook principal com toda a análise
README.md            # Este ficheiro
```

---

## Pipeline de Análise

### 1 · Carregamento e Descrição dos Dados
Carregamento do CSV, filtragem dos 10 compostos do Grupo 7, identificação das variáveis de Raman Shift e descrição da estrutura dos dados.

### 2 · Pré-processamento
- Verificação e interpolação de valores em falta
- Suavização Savitzky-Golay (janela = 11, ordem = 3)
- Correção de baseline: **rubber-band** (referência) e **ALS** — *Asymmetric Least Squares* (λ = 10⁶, p = 0.001) — com comparação visual
- Normalização: **min-max** por espetro e **SNV** (*Standard Normal Variate*), com discussão das diferenças

### 3 · Estatística Descritiva
Estatísticas por composto (média, desvio-padrão, percentis) e exploração gráfica dos espetros individuais e médios.

### 4 · Comparação de Espetros e Bandas de Diagnóstico
- Sobreposição dos espetros médios dos 10 compostos
- Matriz de correlação de Pearson entre espetros médios
- Identificação automática de picos e tabela de bandas cruzada com literatura (SDBS / NIST)

### 5 · Análise Multivariada Não Supervisionada
- **PCA** — scores, loadings e variância explicada acumulada
- **t-SNE**
- **UMAP**

### 6 · Clustering
- **K-Means** com seleção de *k* por método do cotovelo e *silhouette score*
- **Clustering hierárquico Ward** com dendrograma

### 7 · Classificação Supervisionada (5-fold CV estratificado)
Dois alvos de classificação:
- **Por composto** (10 classes)
- **Por família química** (7 famílias) — problema mais exigente, mais informativo sobre generalização

Modelos comparados: KNN, SVM (RBF), Random Forest, Gradient Boosting, Logistic Regression.

### 8 · Importância de Variáveis e Interpretação Espetral
- Importância Gini (Random Forest) no espaço espetral completo
- **SHAP values** — comparação com Gini e heatmap de importância por classe
- **Curvas de aprendizagem** (SVM e RF) para diagnóstico de overfitting

### 9 · Discussão dos Resultados
Análise crítica cobrindo: impacto das escolhas de pré-processamento, estrutura de correlação entre compostos, interpretação química dos clusters, desempenho e limitações dos modelos, e correspondência entre importância computacional (SHAP) e atribuição espetroscópica da literatura.

---

## Requisitos

```bash
pip install numpy pandas matplotlib seaborn scipy scikit-learn umap-learn pybaselines shap
```

> Testado com Python 3.10+. O SHAP é compatível com versões antigas (output em lista) e novas (output em array 3D) — o notebook normaliza o formato automaticamente.

---

## Reprodutibilidade

Todas as operações com componente aleatória usam `random_state = 42`. Para reproduzir integralmente basta executar as células por ordem (`Kernel → Restart & Run All`).

---

## Referências

- SDBS — Spectral Database for Organic Compounds: https://sdbs.db.aist.go.jp
- NIST Chemistry WebBook: https://webbook.nist.gov
- Savitzky & Golay (1964) — *Smoothing and Differentiation of Data by Simplified Least Squares Procedures*
- Eilers & Boelens (2005) — *Baseline Correction with Asymmetric Least Squares Smoothing*
- Lundberg & Lee (2017) — *A Unified Approach to Interpreting Model Predictions* (SHAP)
