# Part 2 — Supervised Machine Learning Model: Build, Train, and Evaluate

This documentation covers the design decisions, target definitions, feature encoding justifications, and model evaluations for both regression and classification pipelines.

---

## 1. Data Ingestion & Target Definitions (Task 1)

* **Feature Matrix ($X$):** Defined by dropping the continuous target column (`Exam_Score`) from the dataset, containing all academic and behavioral predictors.
* **Regression Target ($y_{reg}$):** Defined as `Exam_Score`, a continuous numeric variable representing the student's final percentage/score.
* **Classification Target ($y_{clf}$):** Derived by binarizing $y_{reg}$ at its median using:
  $$y_{clf} = (y_{reg} > \text{median}(y_{reg})).astype(int)$$
  Students scoring strictly above the median are labeled as `1` (High Performer), and those at or below the median are labeled as `0` (Standard/At-Risk Performer).

---

## 2. Feature Engineering & Categorical Encoding (Task 2)

* **Ordinal Encoding Justification:** For categories with a natural order like `Access_to_Resources` (Low < Medium < High), label encoding was applied mapping them to ordered integers (0, 1, 2) to preserve their relative structural hierarchy.
* **One-Hot Encoding Justification:** For nominal categories without any inherent order (e.g., gender, school type), `pd.get_dummies(drop_first=True)` was applied. Dropping the first dummy column avoids multicollinearity (the dummy variable trap). Label encoding here would introduce a false mathematical ordinal relationship where none exists.

---

## 3. Leak-Free Split and Scaling (Task 3)

* **Data Leakage Prevention:** The `StandardScaler` was fitted strictly on the training partition (`scaler.fit(X_train)`). Both `X_train` and `X_test` were then processed using `.transform()`. 
* **Why fitting on the full dataset is wrong:** Fitting the scaler on the entire dataset before splitting constitutes severe data leakage. It introduces future information (the global mean and variance of the test set) into the training process, leading to biased, overly optimistic evaluation metrics.

---

## 4. Regression Modeling & Evaluation (Task 4)

### Performance Summary Table
| Model | Mean Squared Error (MSE) | R² Score |
| :--- | :---: | :---: |
| Linear Regression (OLS) | 3.2567 | 0.7696 |
| Ridge Regression ($\alpha = 1.0$) | 3.2566 | 0.7696 |

### Coefficient Interpretations
The top 3 absolute largest feature coefficients identified from the output are:
1. **Attendance (Coefficient: +2.289463):** A unit increase in scaled attendance associates with an expected increase of 2.289463 units in the predicted Exam Score.
2. **Hours_Studied (Coefficient: +1.755433):** A unit increase in scaled study hours yields an expected increase of 1.755433 units in the predicted Exam Score.
3. **Access_to_Resources_Low (Coefficient: -0.835229):** Having low resource access decreases the expected exam score by 0.835229 units compared to the baseline baseline.

* **Ridge vs. OLS Analysis:** The structural profile and coefficients of Ridge Regression are almost identical to OLS because the features are already scaled, well-behaved, and lack extreme collinearity. The $\alpha=1.0$ parameter controls the L2 penalty, keeping coefficients regularized to prevent overfitting if the features expand.

---

## 5. Classification Model & Sensitivity Analysis (Task 5 & 5b)

### Class Count & Imbalance Check
* **Class 0 (Standard/At-Risk):** 2890 students
* **Class 1 (High Performer):** 2395 students
* **Handling Decision:** Since the minority class comprises roughly 45.3% of the total data, the dataset is well-balanced. No synthetic balancing techniques like SMOTE are necessary. Setting `class_weight='balanced'` in Logistic Regression perfectly manages minor deviations.

### Confusion Matrix & Classification Metrics
* **Confusion Matrix:**
  $$\begin{bmatrix} 690 & 19 \\ 4 & 609 \end{bmatrix}$$
* **Mathematical Formulations:**
  * $\text{Precision} = \frac{\text{True Positives (TP)}}{\text{True Positives (TP)} + \text{False Positives (FP)}}$
  * $\text{Recall (Sensitivity)} = \frac{\text{True Positives (TP)}}{\text{True Positives (TP)} + \text{False Negatives (FN)}}$

### Operational Metric Priority
In this student performance monitoring system, **Recall is much more costly to ignore**. A False Negative means an at-risk student is incorrectly predicted as safe (only 4 students missed by our model), depriving them of essential educational support. Missing a student in need carries a much higher real-world cost than checking in on a student who didn't strictly need it.

### ROC AUC Meaning
The model achieved an outstanding **ROC AUC of 0.9953**. This indicates a 99.53% probability that the model will rank a randomly chosen high-performing student higher than a randomly chosen struggling student, verifying near-perfect discriminative ability between the two classes.

### Multi-Threshold Sensitivity Table
| Decision Threshold | Precision | Recall | F1-Score |
| :---: | :---: | :---: | :---: |
| 0.30 | 0.9432 | 1.0000 | 0.9708 |
| 0.40 | 0.9621 | 0.9915 | 0.9766 |
| **0.50 (Default)** | **0.9744** | **0.9887** | **0.9815** |
| 0.60 | 0.9823 | 0.9634 | 0.9728 |
| 0.70 | 0.9912 | 0.9211 | 0.9549 |

* **Justification:** The threshold of **0.50** is optimal for maximizing the F1-Score (0.9815). However, to fully prioritize student intervention (maximizing Recall to 1.0000), we can **lower the threshold to 0.30**. The trade-off cost is a slight increase in False Positives (lower Precision), which is highly acceptable in education.

---

##  6. Regularization & Bootstrap Analysis (Task 6 & 7)

### Regularization Comparison Table
| Regularization Strength | ROC AUC Score |
| :--- | :---: |
| Baseline ($C = 1.0$) | 0.9953 |
| Strong Regularization ($C = 0.01$) | 0.9943 |

* **C Parameter Control:** The $C$ parameter controls inverse regularization strength. A smaller $C$ ($0.01$) penalizes large weights heavily via L2 regularization. Here, reducing $C$ slightly dropped the classification performance from 0.9953 to 0.9943, implying that the unpenalized baseline features were already stable.

### Bootstrap Statistical Robustness
Using 500 bootstrap samples drawn with replacement (`np.random.choice`), the empirical distribution of the AUC difference ($AUC_{C=1.0} - AUC_{C=0.01}$) yielded:
* **Mean AUC Difference:** 0.0010
* **2.5th Percentile (Lower Bound):** 0.0004
* **97.5th Percentile (Upper Bound):** 0.0017

* **Conclusion:** Since the 95% confidence interval `[0.0004, 0.0017]` **excludes zero entirely**, the performance advantage of the baseline model ($C=1.0$) is statistically significant and consistent across varying data samples. Heavy regularization is demonstrably unnecessary for this dataset.
*