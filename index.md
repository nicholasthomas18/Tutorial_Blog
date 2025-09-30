# How Cross Validation Works

> **Goal:** Get a reliable estimate of how your model will perform on _unseen_ data, not just on one lucky (or unlucky) train/test split.

![A 5-fold cross-validation schematic showing rotating validation folds](https://upload.wikimedia.org/wikipedia/commons/1/1c/K-fold_cross_validation_EN.jpg "Five folds; each fold takes a turn as validation")

---

## You’ll learn to:

- Explain cross validation in plain language
- Choose the right CV variant for your data
- Implement CV and hyperparameter tuning in **scikit-learn**

---

## What problem does cross validation solve?

When training a model to predict data, we often have tuning options, little levers that we can pull that make the model more or less accurate. Cross Validation helps us get the best tuning parameters

Mathematically, **K-fold CV** estimates out-of-sample risk as:

$$
\hat{R}_{\text{CV}}(f) \=\ \frac{1}{K}\sum_{k=1}^{K}\ \frac{1}{|V*k|}\sum*{(x_i,y_i)\in V_k} \ L\big(y_i,\ f^{(-k)}(x_i)\big)
$$

- $\(V_k\)$ is the validation fold at iteration $\(k\)$
- $\(f^{(-k)}\)$ is the model trained on the other $\(K-1\)$ folds
- $\(L(\cdot)\)$ is a loss (e.g., squared error, log-loss, 0/1 error)

---

## How cross validation works (the logic)

1. **Split** your data into _K_ equal-ish folds.
2. For each fold \(k\): **train** on the other \(K-1\) folds, **validate** on fold \(k\).
3. **Aggregate** the \(K\) scores (mean ± std). That’s your generalization estimate.

### Common variants (choose the right tool)

| Variant             | When to use                                         | Pros                                             | Cons / Gotchas                                   |
| ------------------- | --------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------ |
| **KFold**           | i.i.d. data (regression or classification).         | Simple, widely used, good bias/variance balance. | Shuffle is recommended; leakage must be checked. |
| **StratifiedKFold** | Classification with class imbalance.                | Preserves class ratios per fold.                 | Still assumes i.i.d.                             |
| **GroupKFold**      | Multiple rows per entity (e.g., patients, users).   | Prevents group leakage.                          | Need group IDs; fewer effective splits.          |
| **TimeSeriesSplit** | Time-ordered data (no future leakage).              | Respects chronology.                             | Fewer training points in early splits.           |
| **LOOCV**           | Very small \(n\); linear models, educational demos. | Max training data in each fit.                   | High variance; expensive (K = n fits).           |

> **Rule of thumb:** If your data has **structure** (time, groups, repeated measures), use a variant that **respects** that structure.

## How to implement it (scikit-learn, diabetes dataset)

Below are runnable snippets that (a) compute K-fold scores, and (b) tune hyperparameters via nested CV (`GridSearchCV` uses CV internally).

> **Tip:** Use **pipelines** to bundle preprocessing + model, so CV evaluates the _full_ workflow.

```python
# --- Setup ---
# pip install scikit-learn numpy
import numpy as np
from sklearn.datasets import load_diabetes
from sklearn.model_selection import KFold, cross_val_score, GridSearchCV
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LinearRegression, Ridge, Lasso

# Data
X, y = load_diabetes(return_X_y=True)

# A 5-fold splitter; shuffling is good for i.i.d. data
kf = KFold(n_splits=5, shuffle=True, random_state=42)
```

## Call to Action

Go the extra mile and do a cross validation. It’s worth it to figure out which parameters are best for your data.

## NOTE TO THE READER

This blogpost was rushed to be published to meet a deadline and is not complete. The publisher is planning on coming back to improve it.
