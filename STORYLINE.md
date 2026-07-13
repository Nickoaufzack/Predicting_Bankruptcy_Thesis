# STORYLINE — the spine of the thesis

The one thread that runs through every chapter. Keep this open while writing.

## The through-line (5 steps)

1. **Clean pipeline, real data.** No synthetic firms — undersampling, not SMOTE.
   Every training example is a real, historically observed firm.
2. **Keep ALL features (integrity).** Unlike papers that filter/reduce up front
   (Pham → drop + BPSO; Chaudhary → PCA; Sinap → PCA), we feed in all 95 ratios —
   the same choice our base paper Wang & Liu made ("to preserve the integrity of
   the input"). No upfront feature selection. (LR's L1 is embedded selection
   *inside* the model, not a pre-filter on the data.)
3. **That choice leaves the redundancy in.** The dataset is globally low-correlated
   (mean |r| = 0.08) but has local near-duplicate pockets: ~31 of 93 features have a
   twin at |r| ≥ 0.9 (ROA(A/B/C); Debt ratio % ↔ Net worth/Assets = 1.00; etc.).
4. **Retained redundancy makes feature importance non-trivial.** Permutation
   importance collapses (a twin substitutes for the shuffled feature), native Gini
   is biased/split, so the three importance methods **disagree**.
5. **That disagreement IS the finding.** We compare native / permutation / SHAP and
   show which survives the redundancy (SHAP) and which breaks (permutation). Honest
   interpretability *under retained redundancy* — the layer Wang & Liu lacked.

## Where we stand vs the field

- **Pham / Sinap / Chaudhary:** buy accuracy by removing/transforming features
  (and Pham/Sinap by synthesising data via SMOTE). Can't then interpret what they
  dropped.
- **Wang & Liu (2021):** keep all features, real-data undersampling — but stop at
  prediction, no importance analysis.
- **We = Wang & Liu + the feature-importance layer**, done honestly on real,
  full-feature data. Keep AND interpret.

## Non-negotiable framing (Gabriel)

- Methodological focus. **Nothing causal** is inferred — feature importance is
  *associative* (what the model relies on), not economic cause.
- Primary metric **F2** (missing a bankruptcy costs more than a false alarm).
  Accuracy/ROC-AUC misleading under 96.8/3.2 imbalance.

## Source tiering (who to lean on — see also Lit Review journal-quality note)

- **Anchor / most authoritative:** Liang et al. 2016 — **EJOR (ABDC A\*)**, and the
  dataset's origin. Veganzones & Séverin 2018 — **DSS (ABDC A\*)**. Ansah-Narh 2024 —
  ESWA (high-impact Elsevier). Amirshahi & Lahmiri 2024 — Expert Systems (Wiley).
- **Methodological predecessor (extend this):** Wang & Liu 2021 — PLOS ONE
  (legit peer-reviewed megajournal, mid-tier). Value = closest same-dataset
  undersampling+F2+CV study, NOT prestige.
- **Use as "argue-against" examples, not authority:** Pham 2025 (IJTech / RMIT),
  Sinap 2025 (DUJE), Ko & Lee 2024 (JKSIAM), Liashenko 2023 (Ekonomika) — regional
  journals; Suresh 2025 (IEEE conference, 99.6% claim); Chaudhary 2023 (MSc student
  thesis, not peer-reviewed).
- **Method references:** Lundberg & Lee (SHAP), Strobl et al. 2007 (Gini bias),
  Probst et al. 2019 (RF tuning).

## Potential cuts (if space runs tight, ~20 pages)

- Dataset: the commented "integrity / 2008 financial crisis / representativeness"
  block → move to Discussion (critical lens).
- §4.1: the sentence on the two 0/1 flags (Liability-Assets, Net Income) —
  nice-to-have context, cuttable.

## Key numbers (from Final_Code_Output)

- Data: 6,819 firms, 95 features, 3.23% bankrupt.
- Best F2: GB 0.550 > RF 0.546 > LR 0.482.
- Divergence: LR → Debt Ratio #1 (all 3 agree); RF/GB native → Persistent EPS #1
  (Gini artifact); GB SHAP → Borrowing Dependency #1 (0.356); permutation → tiny
  (correlation masking).
