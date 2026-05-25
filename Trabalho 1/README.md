# Trabalho Prático 1 — Extração de Conhecimento de Dados Biológicos
**Grupo 7 | 2025/2026**  
Diogo Barreira · Gonçalo Gomes · José Campos

---

## Descrição

Análise de dados de expressão génica de carcinoma da tiróide (THCA) provenientes do projeto **TCGA-THCA**, acedidos via [Genomic Data Commons (GDC)](https://portal.gdc.cancer.gov/) através do package `TCGAbiolinks`. O dataset contém medições de RNA-seq para **564 amostras** (tumoral e normal) e **60 660 genes**, com 242 variáveis clínicas associadas.

O objetivo principal é a exploração e análise destes dados — desde o pré-processamento até à construção de modelos preditivos capazes de classificar amostras tumorais vs. normais com base nos perfis de expressão génica.

---

## Estrutura do Repositório

```
Trabalho_ECDB_G7/
│
├── fase1/
│   └── trabalhoECDB_1a_fase.Rmd      # Fase 1 — versão final entregue
│
├── fase2/
│   └── trabalhoECDB_2a_fase.Rmd      # Fase 2 — versão final entregue
│
├── fase_final/
│   └── trabalho_fase_final.Rmd       # Fase final — versão final entregue
│
├── versoes_intermédias/
│   ├── trabalhoextrac.Rmd            # Rascunho inicial (v0)
│   └── trabalhoecdb_corr.Rmd         # Versão corrigida pré-fase 1
│
└── README.md
```

> Os relatórios em formato HTML são gerados a partir dos ficheiros `.Rmd` via RStudio/knitr.  
> Os dados são descarregados automaticamente pelo script — ver secção [Como reproduzir](#como-reproduzir).

---

## Pipeline de Análise

### Fase 1 — Pré-processamento e Análise Univariada
`trabalhoECDB_1a_fase.Rmd`

- Descarregamento dos dados via `TCGAbiolinks` (STAR - Counts)
- Filtragem de genes pouco expressos (`rowSums > 10` em pelo menos 10 amostras)
- Transformação log₂ para estabilização da variância
- Estatística descritiva: `summary`, histogramas, boxplots, heatmap dos 50 genes mais variáveis
- Classificação das amostras em **Tumor** vs. **Normal** pelo barcode TCGA (posições 14–15)
- PCA exploratória das amostras
- Análise de expressão diferencial por **teste t** com correção FDR (`p.adjust`, método `"fdr"`)
- Análise de enriquecimento funcional GO (Biological Process) via `clusterProfiler` + `org.Hs.eg.db`
- Guarda do objeto `SummarizedExperiment` em `dados_thca.rds` para reutilização nas fases seguintes

### Fase 2 — Clustering e Redução de Dimensionalidade
`trabalhoECDB_2a_fase.Rmd`

- Carregamento dos dados guardados na Fase 1 (`readRDS`)
- Substituição do teste t pelo **DESeq2** para análise de expressão diferencial mais robusta (distribuição binomial negativa)
- Transformação VSD (`varianceStabilizingTransformation`) para análises downstream
- MA plot e Volcano plot com base nos resultados DESeq2
- **Clustering hierárquico** das amostras (distância euclidiana, método `ward.D2`) com correspondência aos grupos biológicos
- **Heatmap anotado** dos 50 genes mais variáveis (VSD) com separação visual Tumor/Normal
- **MDS** (Multidimensional Scaling / `cmdscale`) das amostras
- Interpretação integrada das três abordagens não supervisionadas: PCA, MDS e clustering convergem para uma separação clara entre grupos

### Fase Final — Modelos Preditivos e Seleção de Genes
`trabalho_fase_final.Rmd`

- Utilização da matriz **FPKM-UQ** (`fpkm_uq_unstrand`) para análise exploratória e modelos preditivos
- Análise de componentes principais com tabela de variância explicada (PC1–PC10)
- Análise de expressão diferencial com **DESeq2** usando diretamente os metadados (`sample_type`)
- Seleção dos 50 genes mais significativos (menor `padj`) como features para ML
- Divisão treino/teste (70/30, `set.seed(123)`)
- **Modelos treinados:**
  - Regressão logística manual (`glm`)
  - Regressão logística com validação cruzada 5-fold (`caret`)
  - k-Nearest Neighbors (`caret`)
- Avaliação com matriz de confusão, accuracy, sensibilidade e especificidade
- Comparação entre genes DE top-50 e genes mais importantes no modelo preditivo (`varImp`) — coincidência total observada

---

## Evolução entre Versões

| Ficheiro | Fase | Alterações relevantes face à versão anterior |
|---|---|---|
| `trabalhoextrac.Rmd` | v0 (rascunho) | Primeira versão: sem nomes nos chunks, `summary(exp_log)` na matriz completa, `barplot` para GO, sem `table(group)`, sem saveRDS |
| `trabalhoecdb_corr.Rmd` | v1 (corrigida) | Adição de `table(group)`, substituição de `barplot` por `dotplot` no GO, tabela top DEGs com ordenação por `p_adj`, nota sobre ~24.76% de genes não convertidos para ENTREZ |
| `trabalhoECDB_1a_fase.Rmd` | Fase 1 final | Chunks com nomes descritivos, histogramas lado a lado (`par(mfrow)`), boxplot limitado a 30 amostras, heatmap com título, `group` como `factor`, `saveRDS` no final |
| `trabalhoECDB_2a_fase.Rmd` | Fase 2 final | Substituição do teste t pelo **DESeq2**, transformação VSD, MA plot, clustering hierárquico com `cutree`, heatmap com anotação de grupos, MDS, interpretação global das análises não supervisionadas |
| `trabalho_fase_final.Rmd` | Fase final | Reescrita completa da introdução, uso de FPKM-UQ, DESeq2 com metadados diretos, modelos ML (regressão logística manual + caret + kNN), métricas detalhadas, comparação DE vs. importância de variáveis |

---

## Packages Utilizados

| Package | Utilização |
|---|---|
| `TCGAbiolinks` | Descarregamento dos dados do GDC |
| `SummarizedExperiment` | Estrutura de dados principal |
| `DESeq2` | Expressão diferencial, VSD |
| `clusterProfiler` | Enriquecimento funcional GO |
| `org.Hs.eg.db` | Conversão ENSEMBL → ENTREZID |
| `pheatmap` | Heatmaps anotados |
| `caret` | Modelos ML com validação cruzada |
| `BiocManager` | Gestão de packages Bioconductor |

---

## Como Reproduzir

1. Clonar o repositório:
   ```bash
   git clone https://github.com/josedrcampos/Trabalho_ECDB_G7.git
   ```

2. Abrir o ficheiro `.Rmd` da fase pretendida no **RStudio**.

3. Fazer *knit* para gerar o relatório HTML.  
   > Na **Fase 1**, os dados são descarregados automaticamente do GDC (requer ligação à internet). O ficheiro `dados_thca.rds` é criado no final e deve estar na mesma pasta para as fases seguintes.  
   > Na **Fase 2** e **Fase Final**, os dados são lidos a partir do `dados_thca.rds` gerado na Fase 1.

4. Os packages Bioconductor necessários são instalados automaticamente no início do script da Fase 1.

---

## Dataset

- **Fonte:** [TCGA via GDC Portal](https://portal.gdc.cancer.gov/)
- **Projeto:** TCGA-THCA (Thyroid Carcinoma)
- **Tipo de dados:** RNA-Seq (STAR - Counts + FPKM-UQ)
- **Amostras:** 564 (Primary Tumor + Solid Tissue Normal)
- **Genes:** 60 660 (antes de filtragem)
- **Metadados:** 242 variáveis clínicas e experimentais

---

## Referências

- The Cancer Genome Atlas Research Network (2014). Integrated genomic characterization of papillary thyroid carcinoma. *Cell*, 159(3), 676–690.
- Wang, Y. et al. (2017). *Thyroid Cancer Epidemiology and Aetiology.*
- Du, L. et al. (2021). *Molecular subtypes and prognosis in thyroid carcinoma.*
