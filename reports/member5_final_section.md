# Final Report — Member 5 Section: Screening Model Selection, Threshold Tuning, and Prevalence Analysis

**Author / Section Lead:** Fatih Osman Darılmaz (220201083, `@darilmaz-cmd`)  
**Scope:** SE309-50 · Group of 5 — Part 5 (*Choosing a model and a threshold for screening*)

---

## 1. Objective and Evaluation Protocol

In a memory-clinic screening setting, the primary clinical priority is identifying older adults who may have dementia so they can be referred for a full specialist neuropsychological and clinical work-up. Missing a truly demented patient (**False Negative, FN**) delays care and support, whereas flagging a healthy individual (**False Positive, FP**) triggers a confirmatory evaluation. Consequently, overall accuracy alone is an inadequate selection criterion.

Using the exact same leakage-free preparation as Part 1 on **OASIS-2** (`373` MRI visits across `150` unique subjects aged `60–98`; binary target `demented = 1` if `CDR > 0`, with `CDR`, `Group`, `Subject ID`, `MRI ID`, `Visit`, `MR Delay`, and constant `Hand` strictly excluded from inputs):
- Subjects were split **subject-wise** (`75% / 25%`, stratified at the subject level, `random_state=42`) into `112` training subjects (`279` visits, `45.5%` demented) and `38` locked held-out test subjects (`94` visits, `42.6%` demented), with **zero subject overlap**.
- Missing values (`SES`: `19` visits across `8` subjects; `MMSE`: `2` visits across `1` subject) were imputed using **training-fold medians** (`SES = 2.0`, `MMSE = 29.0` on the full training set) inside `scikit-learn` `Pipeline` objects, and numeric features were z-score standardized (`StandardScaler`) inside the pipeline for scale-sensitive models.
- All model comparisons and threshold decisions were performed strictly on the **training set** via **10-fold subject-wise stratified cross-validation (`StratifiedGroupKFold`, `n_splits=10, shuffle=True, random_state=42`)** so no subject appeared in both training and validation folds.

---

## 2. 10-Fold Subject-Wise Cross-Validation Comparison

We compared four classifiers alongside the **ZeroR** majority-class baseline (`DummyClassifier(strategy="most_frequent")`):

| Classifier (OASIS-2 Train CV, n=279 / 112 subjects) | Accuracy | Sensitivity | Specificity | Precision | F1 | ROC AUC |
|---|---:|---:|---:|---:|---:|---:|
| ZeroR (Baseline) | 0.545 | 0.000 | 1.000 | 0.000 | 0.000 | 0.500 |
| Decision Tree (`max_depth=3`) | 0.767 | 0.701 | 0.822 | 0.767 | 0.733 | 0.766 |
| Naive Bayes (`GaussianNB`) | 0.796 | 0.661 | 0.908 | 0.857 | 0.747 | 0.858 |
| **Logistic Regression (Standardized)** | **0.828** | **0.756** | **0.888** | **0.850** | **0.800** | **0.908** |
| Random Forest (`100` trees) | 0.778 | 0.740 | 0.809 | 0.764 | 0.752 | 0.852 |

**Key findings:**
1. **ZeroR baseline** achieves `54.5%` accuracy by predicting *nondemented* for every visit, yielding `0.000` sensitivity and `0.500` ROC AUC—clinically useless for screening.
2. **Logistic Regression** outperforms every alternative across **Accuracy (`0.828`)**, **Sensitivity (`0.756`)**, **F1 (`0.800`)**, and **ROC AUC (`0.908`)**, while providing interpretable log-odds coefficients and well-behaved probability estimates.
3. **Why subject-wise CV is essential:** Repeating the same 10-fold evaluation with naive ungrouped `StratifiedKFold` artificially inflated Random Forest ROC AUC from `0.852` to `0.940` (`+0.087`) because multiple visits of the same subject leaked across folds.

---

## 3. ROC Curve and Screening Threshold Selection (Training Data Only)

*(Insert Figure: `images/roc_threshold_cv.png`)*

At the default decision threshold (`t = 0.50`), Logistic Regression misses `31` of `127` demented training visits (`Sensitivity = 0.756`, `Specificity = 0.888`, `FP = 17`). To align the classifier with memory-clinic screening priorities, we apply the cost-based threshold rule (`t = C_FP / (C_FP + C_FN)`):
- Judging a missed dementia case (**FN**) as **3× as costly** as a false alarm (**FP**) yields **`t = 1 / (1 + 3) = 0.25`**.
- On the training 10-fold CV out-of-fold probabilities, lowering the threshold from `0.50` to **`0.25`** increases Sensitivity from `0.756` to **`0.913`** (reducing missed demented visits from `31` to `11`), while maintaining Specificity at **`0.658`** (`Precision = 0.690`, `F1 = 0.786`).
- Lowering further to `t = 0.20` would drop Specificity sharply to `0.572` (`FP = 65`) for only 6 fewer missed cases. Therefore, **`t = 0.25`** was locked strictly on training data before inspecting the held-out test set.

---

## 4. Step-by-Step Precision at Two Prevalences

Sensitivity (`0.913`) and Specificity (`0.658`) are intrinsic operating properties of the classifier at `t = 0.25`, whereas **Precision (Positive Predictive Value, PPV)** depends strongly on disease prevalence via Bayes' theorem:

$$\text{PPV} = \frac{\text{Sensitivity} \times \text{Prevalence}}{(\text{Sensitivity} \times \text{Prevalence}) + (1 - \text{Specificity}) \times (1 - \text{Prevalence})}$$

Evaluating a hypothetical cohort of **1,000 screened older adults** step by step:

1. **At OASIS-2 Dataset Prevalence (`167 / 373 = 44.77%`):**
   - Truly demented: `1,000 × 0.4477 = 447.7` → True Positives (`TP`) = `447.7 × 0.9134 = 408.9`
   - Truly nondemented: `1,000 × 0.5523 = 552.3` → False Positives (`FP`) = `552.3 × (1 − 0.6579) = 188.9`
   - Total flagged positive: `408.9 + 188.9 = 597.9`
   - **Precision (PPV)** = `408.9 / 597.9 = 68.4%`

2. **At General Older Population Prevalence (`10.0%`):**
   - Truly demented: `1,000 × 0.10 = 100.0` → True Positives (`TP`) = `100.0 × 0.9134 = 91.3`
   - Truly nondemented: `1,000 × 0.90 = 900.0` → False Positives (`FP`) = `900.0 × (1 − 0.6579) = 307.9`
   - Total flagged positive: `91.3 + 307.9 = 399.2`
   - **Precision (PPV)** = `91.3 / 399.2 = 22.9%`

**Clinical Meaning:** Even with identical sensitivity (`91.3%`) and specificity (`65.8%`), moving the model from an enriched research cohort (`44.8%` prevalence) to general community screening (`10%` prevalence) causes Precision to fall from **`68.4%` to `22.9%`**. In community screening, **~77% of positive flags (`308` of `399` per 1,000 people) would be false alarms**. In real-world use, this model could only serve as a **first-stage triage prompt** prior to specialist cognitive and clinical assessment—never as a standalone diagnostic tool—otherwise it would cause severe patient anxiety and clinic overload (alarm fatigue).

---

## 5. Final One-Time Held-Out Test Evaluation

*(Insert Figure: `images/test_confusion_matrix.png`)*

Fitting the locked Logistic Regression pipeline on all `279` training visits (`112` subjects) and evaluating **once** on the `94` held-out test visits (`38` unseen subjects: `40` demented, `54` nondemented) yielded **ROC AUC = `0.873`**:

| Held-Out Test Setting (n=94 visits, 38 unseen subjects) | Accuracy | Sensitivity | Specificity | Precision | F1 | TP | FN | FP | TN |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| ZeroR (Always Nondemented) | 0.574 | 0.000 | 1.000 | 0.000 | 0.000 | 0 | 40 | 0 | 54 |
| Logistic Regression (default `t = 0.50`) | 0.755 | 0.725 | 0.778 | 0.707 | 0.716 | 29 | 11 | 12 | 42 |
| **Logistic Regression (screening `t = 0.25`)** | **0.723** | **0.900** | **0.593** | **0.621** | **0.735** | **36** | **4** | **22** | **32** |

At the locked screening threshold `t = 0.25`, the model detects **`90.0%` (`36 / 40`)** of demented visits on unseen subjects—cutting missed cases from `11` down to `4`—at the cost of `22` false alarms (`Specificity = 0.593`, `F1 = 0.735`), closely matching the training cross-validation behavior and confirming no threshold overfitting.
