# Linear vs. GENIE3-Style Perturbation Predictors  
**Reproducing and extending benchmarks of single-cell perturbation models**

---

## 📘 Overview
This repository reproduces the linear baseline benchmark from  
> *“Foundation models struggle to outperform simple linear predictors for gene perturbation prediction”* (2024)

and extends it with a **GENIE3-inspired Random-Forest (RF) model** for predicting gene-expression changes after perturbations.  
We compare both models to existing deep-learning approaches (GEARS, scGPT) using the **Replogle et al. (2022) K562 CRISPRi** single-perturbation dataset.

---

## 🧠 Project goals
1. **Reproduce** the paper’s linear low-rank model.  
2. **Implement** a tree-based, per-gene Random-Forest perturbation predictor (GENIE3 style).  
3. **Evaluate** performance on unseen single-gene perturbations.  
4. **Explain** model behavior using SHAP and LIME.  
5. **Compare** interpretability and accuracy between linear and tree-based models.

---


---

## ⚙️ Data inputs
Both models expect the same inputs:

### `Y_pseudobulk.csv`
| gene | ctrl | GENEA_KD | GENEB_KD | GENEC_KD |
|------|------|-----------|-----------|-----------|
| GENE1 | 4.23 | 3.98 | 4.35 | 4.10 |
| GENE2 | 5.11 | 4.70 | 5.40 | 5.15 |

*rows = genes, columns = perturbation conditions, values = log-normalized mean expression*

### `perturbations.csv`
| condition | perturbed_gene |
|------------|----------------|
| ctrl | NONE |
| GENEA_KD | GENEA |
| GENEB_KD | GENEB |

### (optional) `scgpt_gene_emb.npy`
Pre-computed scGPT gene embeddings (n_genes × emb_dim) aligned to `Y_pseudobulk.csv`.

---

## 🧩 Models implemented
### 1. **Linear baseline**
\[
\hat{Y} = GWP^T + b\mathbf{1}^T
\]
- \(G\): PCA gene embedding (K=10)  
- \(P\): PCA perturbation embedding (L=10)  
- \(W\): ridge-fit weight matrix (λ = 0.1)  
- Captures additive effects between genes and perturbations.

### 2. **GENIE3-style Random-Forest**
- One RF per target gene: predicts gene expression from perturbation features.  
- Supports binary one-hot or scGPT embedding inputs.  
- Captures nonlinear and conditional perturbation effects.  
- Aggregates feature importances into an inferred GRN.

---

## 📈 Metrics
| Metric | Description |
|---------|--------------|
| **RMSE** | L2 distance between predicted & observed expression (top 1000 genes) |
| **Pearson δ** | correlation of predicted vs observed expression changes |
| **Bootstrap 95 % CI** | confidence intervals across random splits |
| **Calibration slope/intercept** | checks prediction bias & scaling |
| **Top-k stability (Jaccard)** | overlap of most important features across repeats |
| **Permutation null** | baseline after shuffling perturbation labels |

---

## 🧪 Explainability
- **Linear model:** coefficient inspection, SHAP (LinearExplainer-like via `G@W`), contribution heatmaps.  
- **RF model:** TreeExplainer SHAP values, feature-importance stability, aggregated GRN edges.  
- **Optional:** LIME or permutation importance for confirmatory checks.

---

