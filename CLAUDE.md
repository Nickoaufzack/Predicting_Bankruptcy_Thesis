# CLAUDE.md — Bankruptcy Prediction Thesis

This file is the persistent context for every Claude session on this project
(both Claude Code and project chats). Read it fully before making changes.
The "Locked Decisions" and "Non-Negotiables" sections override any inferred
best-practice — they reflect deliberate scientific positioning that protects
the thesis storyline.

**Status (July 9, 2026): Analysis complete. The project is in the WRITING phase.**
Code, results, and storyline are finished. What remains is writing the thesis.

---

## 1. Project Identity

- **Title:** *Predicting Bankruptcy with Machine Learning: An Undersampling and Feature Importance Approach using a Taiwanese Dataset*
- **Author:** Niklas Demmrich (TUM B.Sc. Management & Technology)
- **Supervisor:** Gabriel, Chair of Applied Econometrics (informal "du" — never formal address)
- **Official deadline:** August 15, 2026
- **Personal target (last resort):** July 31, 2026
- **Real working target:** ~July 17, 2026 — draft finished to send out for peer review
- **Milestone context:** Graduation celebration ~July 23 — ideal to be finished (feedback incorporated) by then, not necessarily submitted, just done and stress-free.
- **Work rhythm from July 9:** Full days (~9–19h), daily. No hard limits — pace is comfortable, buffer is deliberate to avoid end-stress.
- **6 supervisor meetings held.** All transcripts are in the project knowledge base.

---

## 2. The Scientific Storyline (NON-NEGOTIABLE)

This thesis positions itself as a **"Clean Pipeline"** against the current bankruptcy-prediction literature:

- Modern high-performance papers (Pham 2025, Suresh 2025, Sinap 2025) achieve 98–99% accuracy by using SMOTE / synthetic oversampling. This is treated as scientifically questionable because it fabricates bankruptcy events that never occurred in the Taiwanese market.
- This thesis trains **exclusively on real historical observations**. No synthetic minority data. No synthetic centroids. No SMOTE.
- The primary scientific contribution is **extending Wang & Liu (2021)** — same undersampling rigor and CV framework, but adding the **Feature Importance layer they lacked**.
- The primary metric is the **F2-score**, not accuracy. Missing a bankruptcy (False Negative) is economically much more costly than a false alarm.

**The core contribution is interpretability under a clean-undersampling premise** — "widening the black box" by comparing multiple feature-importance methods and explaining where and why they diverge. The value is not finding a "better" method; it is the *comparison* and the honest discussion of the divergence.

Any change that contradicts this storyline must be flagged, not silently implemented.

---

## 3. Locked Methodological Decisions (FINAL)

### Models — LR / RF / GB (three models, final)

The original 8-model / 3-tier plan (with SVC as lead) was **deliberately abandoned** in favor of a focused three-model comparison:

| Model | Role | Best F2 (representative) |
|-------|------|--------------------------|
| **Logistic Regression** | Interpretable, signed linear baseline (L1/L2) | ~0.48 |
| **Random Forest** | Tree ensemble — variance averaged away via bagging + feature subsampling | ~0.55 |
| **Gradient Boosting** | Tree ensemble — bias reduced sequentially with shallow stumps | ~0.55–0.56 (best) |

- The storyline is **simple → complex**: LR as an intuitive, understandable baseline, then RF and GB which earn their place through higher F2. Every model has its "Daseinsberechtigung" (Gabriel, Meeting 5): even non-top models give insight. This is an *application* of the algorithms, not a race for the single best model.
- **SVC, k-NN, LDA, XGBoost, ANN are out.** SVC (RBF) only tied the linear LR, confirming the relevant structure is threshold-based interactions (trees' strength), not smooth nonlinearity.
- **Naive Bayes is deliberately excluded but held in reserve.** It is implemented in `models.py` but not in `MODELS_TO_RUN`. Under ClusterCentroids it collapses (F2 ~0.16 — the opposite of Wang & Liu, where ENN+NB was best). This is a *strong potential discussion point* (performance = model × sampling interaction) but is NOT in scope unless the Discussion needs more weight. Do not add it silently.

### Sampling — ClusterCentroids only (final)

- **Approach:** Undersampling only. No SMOTE, no oversampling, no hybrid. The multi-sampler comparison (Tomek/ENN/OSS/NCR) from the old plan is dropped — the thesis uses **ClusterCentroids exclusively**.
- **Mechanism (correct — clarified in Meeting 5):** ClusterCentroids runs **K-Means** over the majority class. It forms exactly as many clusters as firms to keep (660), and each centroid becomes a new representative firm. It does **NOT** form a few clusters and delete outliers — the centroids *are* the new majority set. With `voting='hard'`, the real firm nearest each centroid is kept (not a synthetic average). So 5,939 healthy firms are *condensed* into 660 real representative firms.
- **`voting='hard'` is non-negotiable** (keeps real firms — see Non-Negotiables §5).
- **`sampling_strategy=0.3` is FIXED** for comparability with Wang & Liu (their Fig. 12 shows the LDA optimum near 30%). Not swept. 198 bankrupt ÷ 0.3 = 660 healthy per training fold.

### Validation (final)

- **Cross-validation:** RepeatedStratifiedKFold, 10 folds × 3 repeats = 30 fits per model.
- **Scaling + sampling happen INSIDE each CV training fold** via `imblearn.pipeline.Pipeline` (MinMaxScaler → ClusterCentroids → model). Test fold: scaler transforms only, sampler skipped. Leakage-free.
- **GridSearch** is nested in the CV: each hyperparameter combination gets the full 30 fits; the combination with the highest fold-averaged F2 wins.

### Primary Metric (final)

- **F2-score** (`fbeta_score(beta=2)`) as a custom scorer — recall weighted 2× over precision.
- Secondary: Recall, Precision, F1, ROC-AUC, Confusion Matrix.
- Accuracy is reported but never used for selection (meaningless on a 96.8/3.2 split — an active point to make to Gabriel; ROC-AUC is likewise misleading under imbalance).

### Feature Importance — three methods (final, IMPLEMENTED)

This is the thesis's central contribution. Three importance perspectives per model:

1. **Native importance:** LR signed coefficients (log-odds, direction-bearing); RF/GB Gini-impurity importance (unsigned, biased toward continuous/high-cardinality features per Strobl et al. 2007).
2. **Permutation Importance:** model-agnostic, F2 scorer, 10 repeats, all firms. Measures performance-drop when a feature is shuffled. The comparison anchor across models.
3. **SHAP:** LinearExplainer for LR (exact), TreeExplainer for RF/GB (exact) — both circumvent the 2^94 combinatorial impossibility of brute-force Shapley. Class 1 = bankruptcy, ranked by mean(|SHAP|).

**The central finding = systematic three-way divergence:**
- LR: all three methods agree — Debt Ratio #1 (economically plausible, signed).
- RF/GB native Gini: Persistent EPS #1 (Gini bias artifact).
- SHAP partially corrects the Gini bias: GB SHAP puts **Borrowing Dependency #1** (0.356) above EPS — economically sensible.
- Permutation on RF/GB shows characteristically **tiny values** (correlation masking: with 95 correlated features, permuting one barely hurts because a correlated backup substitutes). Textbook illustration of the permutation weakness under correlation.

**Key SHAP framing point:** SHAP is the only method that produces *signed* importance for *all three* models — it is therefore the only method that places all models on the same comparable axis (LR coefficients are signed but linear-only; Gini is unsigned). This motivates SHAP as the primary interpretive tool.

**Deliberately rejected (mention as known-but-not-used, do NOT implement):**
- **LIME** — local-only, unstable (random perturbations → varying explanations), answers the same question as SHAP with weaker guarantees. Per Meeting 6: mention with a short *content* justification is optional, not required.
- **Pearson correlation** (Chaudhary) — "too simple, only linear", and it's feature *selection* upfront, not feature *importance* on the fitted model. Belongs in the Literature Review, NOT in the FI chapter. No Pearson computation exists or is needed.
- **BPSO** (Pham) — feature selection metaheuristic, different purpose, out of scope.

### Causal caution (cross-cutting, NON-NEGOTIABLE in writing)

Without hypothesis tests or a causal identification design, all feature-importance statements are **associative, not causal**. FI measures what the model relies on for prediction, not which economic mechanisms actually cause bankruptcy. This caution must be audible throughout Feature Importance, Results, and Discussion. It is scientifically protective and Gabriel values the nuance.

---

## 4. Reference Benchmark — Wang & Liu (2021)

The closest existing paper; the base of the thesis's methodology.

- **Dataset:** Same Taiwanese Bankruptcy Dataset, 6,819 firms, 95 features, ~3.23% bankrupt.
- **Their best result:** ENN (k=3) + Naive Bayes → F2 = 0.423. Their CMUT(30%) for LDA = 0.3994.
- **Shared methodology:** repeated stratified 10-fold CV (×3), leakage-prevention pipeline, F2 as primary metric.
- **Their CMUT vs. our ClusterCentroids:** same *philosophy* (keep representative real majority firms), different *algorithm* (they delete farthest points; imblearn keeps nearest to centroid). **Do NOT claim to replicate CMUT exactly or to "beat Wang & Liu under identical conditions."** Honest framing: our best setup (RF/GB + ClusterCentroids ~0.55) vs. their best reported (ENN+NB 0.423); RF/GB + undersampling is a combination they left unexplored.
- **Gap filled:** Wang & Liu has no feature importance / no economic interpretation. This thesis adds that layer.

---

## 5. Non-Negotiables / Critical Pitfalls

1. **`ClusterCentroids` MUST use `voting='hard'`.** The default resolves to synthetic centroids (mathematical averages, not real firms), which breaks the "no synthetic data" claim.
   ```python
   ClusterCentroids(sampling_strategy=0.3, voting='hard', random_state=69)
   ```
2. **Scaling + sampling inside CV folds only**, via `imblearn.pipeline.Pipeline` (never `sklearn.pipeline.Pipeline` for samplers).
3. **No SMOTE, no oversampling, no synthetic minority generation.** Ever. For comparison, cite the literature instead.
4. **Accuracy is never the selection criterion.** F2 only. ROC-AUC also misleading under imbalance.
5. **`Liability-Assets Flag` and `Net Income Flag` are categorical (0/1)** — don't scale/transform them like continuous ratios.
6. **No forced 50:50 balancing** (contra Liang et al. 2016). The natural post-undersampling ratio is the truth.

---

## 6. Thesis Structure — the "Equator" (FLEXIBLE working map)

This is the agreed chapter structure with a rough page budget (~20 pages of text; embedded plots are extra). **It is deliberately flexible** — page counts and content will shift as we reach each chapter, things may be added or cut, and Gabriel may give further feedback. Treat it as the orienting map, not a contract.

| # | Chapter | Pages | What it does |
|---|---------|-------|--------------|
| 1 | Abstract | 0.25–0.5 | Summarizes the whole thesis; research question + short answer. |
| 2 | Introduction | 1 | Motivation, positioning against the literature, overview of the approach. |
| 3 | Literature Review | 1.5 | Situates the work; carves out the interpretability research gap. (Pearson/Altman may be cut to save space.) |
| 4 | Dataset | 0.5 | Brief description of the Taiwanese dataset + first critical questions (raised, left open). |
| 5 | Methodology | 7–8 | Methods backbone (sub-chapters below). |
| 5.1 | Data Preprocessing | 0.5 | MinMax normalization; leakage avoidance. |
| 5.2 | Undersampling | 1.5–2 | ClusterCentroids + K-Means mechanism; voting='hard'; argument against SMOTE. |
| 5.3 | Prediction Models | 2–2.5 | LR / RF / GB function + bias-variance contrast; why these three. |
| 5.4 | Cross-Validation | 1 | RepeatedStratifiedKFold; leakage-free pipeline; embedded GridSearch. |
| 5.5 | Performance Metrics | 1 | Confusion matrix, P/R/F1/Fβ formulas, F2 justification. |
| 5.6 | Hyperparameter Optimization | 1 | GridSearch results; overfitting check (complexity curves → appendix, referenced in text). |
| 6 | Feature Importance | 2.5–3 | Explains the three methods (NOT results). Causal caution up front. Weight on 6.3 SHAP (Shapley formula, why 2^94 → Linear/TreeSHAP). |
| 7 | Experimental Results | 2 | Model scores + FI results; the divergence at the center. Strictly predictive relevance, no causal claims. Plots embedded. |
| 8 | Discussion | 1.5 | Interpretive core; the two "avenues" (Gini artifact vs. real economic result); economic reading under association caveat; dataset limitations revisited. |
| 9 | Conclusion | 0.5–1 | What was solved, the methodological contribution, outlook (2nd dataset, deeper economic interpretation). |

**Gabriel's concrete writing directives (Meeting 6):**
- FI complex ≈ 4 pages total (method in ch. 6 + FI results in ch. 7), for a Bachelor reader who doesn't know SHAP.
- **Pro/Contra comparison** of Permutation vs. SHAP — first general (textbook advantages/disadvantages), then specific to this correlated dataset. This is the red thread of the FI treatment.
- **The complex Shapley formula (with coalitions/factorials) must be explained** — not just the simplified version. Start from the normal Shapley value, show why exact computation is infeasible (2^94), then Linear/TreeSHAP as the heuristics used. Reference: Lundberg & Lee.
- Complexity/overfitting curves → **appendix**, one sentence in the main text. Note that `max_depth` means something different for RF vs. GB (can't compare RF=10 to GB=2 directly).
- LIME non-use: optional short content-based justification, not required.

---

## 7. Code Conventions

- **Language:** English code, variables, comments, docstrings.
- **Random seed:** `RANDOM_STATE = 69` (centralized in `config.py`). Deliberately non-standard, kept consistent for reproducibility.
- **Style:** PEP8; type hints on function signatures; Google-style docstrings for public functions.
- **Git:** **Do NOT auto-commit.** Niklas commits manually at milestones. Suggest messages, never run `git commit`/`push`.
- **No magic numbers in module code** — grids, paths, seeds live in `config.py`.
- **Path handling:** `pathlib.Path` relative to project root, never absolute paths.

---

## 8. Code Structure (final — analysis complete)

```
src/
├── config.py            # RANDOM_STATE=69, paths, hyperparameter grids
├── data_loader.py       # load_data()
├── sampling.py          # ClusterCentroids (voting='hard', sampling_strategy=0.3)
├── models.py            # LR, RF, GB factories (NB present but not in MODELS_TO_RUN)
├── pipelines.py         # imblearn Pipeline, CV wrapper, GridSearch
├── evaluate.py          # F2 scorer, confusion matrix, secondary metrics
├── features.py          # native + SHAP (Linear/Tree) + permutation importance
└── main.py              # orchestration; MODELS_TO_RUN = ['LogisticRegression','RandomForest','GradientBoosting']
```

The canonical final run (all metrics + FI rankings) is saved as **`Final_Code_Output`** in the project knowledge base — use those exact numbers when writing, for consistency.

---

## 9. Writing Workflow (current phase)

Three separate spaces, each with one job:

1. **Main strategy chat** — the full project-journey archive. Used for big-picture strategy, chapter structure, meeting-debrief extraction. Heavy context; use sparingly for high-level questions.
2. **Writing workshop (Cowork / project chat linked to the LaTeX repo)** — where the actual writing and LaTeX editing happens, with direct file access. Fresh, light context; carries full domain context via this knowledge base.
3. **Overleaf (TUM ShareLaTeX, Premium ~1 month)** — compile + preview + share for peer review. Fed from the same GitHub repo.

- The LaTeX repo is on GitHub; the TUM Overleaf template is the required format (do not rebuild it).
- Thesis is written in **English**; conversations with Niklas are in **German**.
- Meeting debriefs are authoritative inputs — extract concrete writing directives from them.
- When writing chapters, pull exact numbers from `Final_Code_Output`, not from memory.

---

## 10. Gabriel's Expectations & Style

- Informal "du" relationship; strong personal rapport (was Niklas's econometrics tutor).
- Values **analysis depth and sound discussion over polished writing or raw accuracy.** His judgment depends "0.0%" on what the FI list happens to show — only on technical quality and the quality of the discussion.
- Wants methods **explainable to a fellow Bachelor student**; "you understand it if you could hand-code it."
- Pro-honesty: **"report findings even if not fully understood"** — the opposite of p-hacking hyperparameters until the FI list looks nice. Surprising results (e.g. EPS) are fine and are discussion material, via two legitimate avenues (method artifact vs. real economic result).
- Key word: **"wasserdicht"** (watertight) — methodology must withstand scrutiny.
- Likes the scientific-integrity angle ("no synthetic firms") as the unique contribution.

---

## 11. Key Literature Reference Points

| Paper | What it gives the thesis |
|-------|--------------------------|
| Wang & Liu (2021) | Base methodology, F2 justification, sampling + CV framework; the paper the thesis extends |
| Liang et al. (2016) | Feature-category groundwork; the "50:50 stratified" approach to argue against |
| Chaudhary (2023) | The "Pearson is too simple" critique (Literature Review only) |
| Pham et al. (2025) | The "Black Box" benchmark to argue against (98.6% via SMOTE + ANN) |
| Suresh et al. (2025) | High-accuracy SMOTE example; integrity > 0.5% F1 gain |
| Sinap (2025) | SMOTE vs. Tomek comparison; also uses SHAP — context for the FI discussion |
| Ansah-Narh et al. (2024) | Uses ClusterCentroids; RF feature-importance example |
| Lundberg & Lee | The SHAP reference for the Shapley-value / additive-attribution formulas |
| Strobl et al. (2007) | The Gini-bias citation (impurity importance favors continuous/high-cardinality features) |
| Probst, Wright & Boulesteix (2019) | RF hyperparameter justification (max_features as key parameter) |
