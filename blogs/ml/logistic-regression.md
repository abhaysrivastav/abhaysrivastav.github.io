---
layout: topic
title: "Logistic Regression — From Probability to Prediction"
permalink: /blogs/logistic-regression/
date: 2025-01-25
categories: [machine-learning, classification]
tags: [logistic-regression, classification, sigmoid, gradient-descent, scikit-learn, overfitting]
author: Abhay
math: true
image: /blogs/assests/ml-img/logistic-regression/decision_boundary.png
description: "A complete guide to Logistic Regression — covering the sigmoid function, cost function, gradient descent, and overfitting — with scikit-learn code and worked examples."
---

Logistic Regression is one of those algorithms that sounds deceptively simple but quietly powers a surprising number of real-world systems — spam filters, credit scoring, medical diagnosis, and more. Despite the word "regression" in its name, it is fundamentally a **classification** algorithm. That slight naming confusion trips up a lot of beginners, so let's clear it up immediately and then go deep.

---

## 1. What is Logistic Regression?

Logistic Regression is a **supervised learning algorithm** used for **binary classification** — predicting whether an input belongs to one of two categories (yes/no, spam/not-spam, disease/no-disease).

The key insight: instead of predicting a raw number like linear regression does, logistic regression predicts a **probability** — a value between 0 and 1 — and then applies a threshold to make a final class decision.

| Property | Details |
|---|---|
| **Task** | Binary Classification (extendable to multi-class) |
| **Output** | Probability → Class label |
| **Decision Rule** | If P(y=1 \| x) ≥ 0.5 → Class 1, else Class 0 |
| **Core Function** | Sigmoid (Logistic) Function |
| **Loss Function** | Binary Cross-Entropy (Log-Loss) |
| **Optimizer** | Gradient Descent |

---

## 2. Classification with Logistic Regression

The starting point is the same linear equation you know from linear regression:

$$z = w_1x_1 + w_2x_2 + \ldots + w_nx_n + b = \mathbf{w}^T\mathbf{x} + b$$

The problem? This gives you a number that ranges from −∞ to +∞. You cannot interpret that as a probability. To "squash" it into the (0, 1) range, logistic regression applies the **sigmoid function**:

$$\hat{y} = \sigma(z) = \frac{1}{1 + e^{-z}}$$

This single transformation is what separates logistic regression from linear regression. The output $\hat{y}$ now represents the probability that the input belongs to class 1.

### The Sigmoid Function

![Sigmoid Function](/blogs/assests/ml-img/logistic-regression/sigmoid_function.png)
*The sigmoid function maps any real number to a probability between 0 and 1. The shaded green region predicts Class 1; red predicts Class 0.*

**Key properties of the sigmoid:**
- Output is always in (0, 1)
- Symmetric around z = 0 → σ(0) = 0.5
- As z → +∞, σ(z) → 1; as z → −∞, σ(z) → 0
- Smooth and differentiable — which is critical for gradient descent

### From Probability to Class Label

Once you have $\hat{y}$, the classification rule is straightforward:

$$\text{Predicted class} = \begin{cases} 1 & \text{if } \hat{y} \geq 0.5 \\ 0 & \text{if } \hat{y} < 0.5 \end{cases}$$

The threshold 0.5 is the default, but it is not sacred — you can tune it depending on whether false positives or false negatives matter more in your problem.

### Decision Boundary

When you visualize a trained logistic regression model in 2D, the boundary between the two classes is a **straight line** (in the original feature space). This is a linear decision boundary — which is both a strength (interpretable, fast) and a limitation (cannot capture non-linear separations directly).

![Decision Boundary](/blogs/assests/ml-img/logistic-regression/decision_boundary.png)
*The yellow line is the learned decision boundary. Green points are Class 1, red are Class 0. The background gradient shows predicted probability.*

### Model Architecture

The diagram below shows the complete forward pass — from raw features to a final class prediction:

![Logistic Regression Architecture](/blogs/assests/ml-img/logistic-regression/architecture.png)
*End-to-end flow: features → weighted sum → sigmoid → probability → class label. The dashed yellow arrow shows how gradient descent feeds the loss back to update weights.*

---

## 3. Cost Function for Logistic Regression

You might wonder: why not just use mean squared error (MSE) as the loss function? The honest answer — **it does not work well**. When you plug the sigmoid into MSE, the resulting curve becomes non-convex with many local minima, making optimization unreliable.

Logistic regression instead uses **Binary Cross-Entropy** (also called **Log-Loss**):

$$J(\mathbf{w}, b) = -\frac{1}{m} \sum_{i=1}^{m} \left[ y^{(i)} \log(\hat{y}^{(i)}) + (1 - y^{(i)}) \log(1 - \hat{y}^{(i)}) \right]$$

### Why This Formula Works

Notice that it has two terms that activate depending on the true label:

- **When y = 1:** the cost is $-\log(\hat{y})$. If the model predicts $\hat{y}$ close to 1, cost is near 0. If it predicts close to 0, cost shoots to infinity — a huge penalty.
- **When y = 0:** the cost is $-\log(1 - \hat{y})$. If the model predicts close to 0, cost is near 0. If it predicts close to 1 (wrong), cost is very high.

![Log-Loss Cost Function](/blogs/assests/ml-img/logistic-regression/log_loss.png)
*Left: cost when the true label is 1. Right: cost when the true label is 0. The cost grows steeply when the prediction is confidently wrong.*

### Worked Example

Suppose the true label is y = 1 and the model predicts $\hat{y} = 0.9$:

$$\text{Cost} = -[1 \cdot \log(0.9) + 0 \cdot \log(0.1)] = -\log(0.9) \approx 0.105$$

Now suppose the model predicts $\hat{y} = 0.1$ (very wrong):

$$\text{Cost} = -\log(0.1) \approx 2.303$$

The penalty for being confidently wrong is enormous — exactly the behavior you want from a good loss function.

---

## 4. Gradient Descent for Logistic Regression

The goal of training is to find the weights $\mathbf{w}$ and bias $b$ that **minimize** the cost function $J$. Gradient descent does this iteratively by taking small steps in the direction that reduces the cost the fastest.

### The Update Rule

At each iteration, for every weight $w_j$:

$$w_j := w_j - \alpha \frac{\partial J}{\partial w_j}$$
$$b := b - \alpha \frac{\partial J}{\partial b}$$

Where $\alpha$ is the **learning rate** — a hyperparameter that controls how large each step is.

### Computing the Gradients

Through calculus (the chain rule applied to the sigmoid and log-loss), the gradients simplify to:

$$\frac{\partial J}{\partial w_j} = \frac{1}{m} \sum_{i=1}^{m} (\hat{y}^{(i)} - y^{(i)}) x_j^{(i)}$$

$$\frac{\partial J}{\partial b} = \frac{1}{m} \sum_{i=1}^{m} (\hat{y}^{(i)} - y^{(i)})$$

The elegant thing here is that the gradient formula looks identical to linear regression's gradient — the sigmoid's derivative cancels out to produce a clean expression.

### Worked Example

Suppose m = 3 samples, and the model errors $(\hat{y} - y)$ are [0.3, -0.1, 0.2] for a single feature x = [1, 2, 3]:

$$\frac{\partial J}{\partial w_1} = \frac{1}{3}[(0.3)(1) + (-0.1)(2) + (0.2)(3)] = \frac{1}{3}[0.3 - 0.2 + 0.6] = \frac{0.7}{3} \approx 0.233$$

With a learning rate of $\alpha = 0.1$, the weight update is:

$$w_1 := w_1 - 0.1 \times 0.233 = w_1 - 0.0233$$

This single update step reduces the cost slightly. Repeated over hundreds of iterations, it converges to the optimal weights.

### Convergence

![Gradient Descent Convergence](/blogs/assests/ml-img/logistic-regression/gradient_descent.png)
*Cost decreases over iterations and flattens near the global minimum. This is the hallmark of a well-tuned gradient descent run.*

**Practical hyperparameter tips:**
- If the loss oscillates or diverges → learning rate is too high, reduce it
- If convergence is painfully slow → learning rate is too low, increase it
- Try learning rates on a log scale: 0.001, 0.01, 0.1, 1.0

---

## 5. Pseudocode

```
ALGORITHM: Logistic Regression via Gradient Descent

INPUT:
  X         — feature matrix (m × n)
  y         — true labels (m × 1), values in {0, 1}
  alpha     — learning rate
  epochs    — number of iterations

INITIALIZE:
  w ← zeros(n)     # weight vector
  b ← 0            # bias

FOR each epoch:
  z ← X · w + b                        # linear combination (m × 1)
  y_hat ← sigmoid(z)                   # predicted probabilities
  error ← y_hat - y                    # prediction error

  dw ← (1/m) * Xᵀ · error             # gradient w.r.t. weights
  db ← (1/m) * sum(error)              # gradient w.r.t. bias

  w ← w - alpha * dw                   # update weights
  b ← b - alpha * db                   # update bias

  J ← -(1/m) * sum(y * log(y_hat)
        + (1 - y) * log(1 - y_hat))    # compute cost (monitor)

RETURN w, b
```

---

## 6. Logistic Regression with scikit-learn

In practice, you will never implement logistic regression from scratch — scikit-learn's `LogisticRegression` is fast, well-tested, and production-ready.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import (
    accuracy_score, classification_report,
    confusion_matrix, ConfusionMatrixDisplay, roc_auc_score, roc_curve
)
from sklearn.datasets import load_breast_cancer

# ── 1. Load data ──────────────────────────────────────────────────────────────
data = load_breast_cancer()
X, y = data.data, data.target
print(f"Dataset shape: {X.shape} | Classes: {data.target_names}")

# ── 2. Train-test split ───────────────────────────────────────────────────────
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# ── 3. Feature scaling (important for logistic regression!) ───────────────────
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s  = scaler.transform(X_test)

# ── 4. Train the model ────────────────────────────────────────────────────────
model = LogisticRegression(
    C=1.0,              # inverse regularization strength (smaller = stronger reg)
    solver='lbfgs',     # good default for small-medium datasets
    max_iter=1000,
    random_state=42
)
model.fit(X_train_s, y_train)

# ── 5. Predictions ────────────────────────────────────────────────────────────
y_pred      = model.predict(X_test_s)
y_prob      = model.predict_proba(X_test_s)[:, 1]   # probability for class 1

# ── 6. Evaluation ─────────────────────────────────────────────────────────────
print(f"\nAccuracy : {accuracy_score(y_test, y_pred):.4f}")
print(f"ROC-AUC  : {roc_auc_score(y_test, y_prob):.4f}")
print("\nClassification Report:")
print(classification_report(y_test, y_pred, target_names=data.target_names))

# ── 7. Confusion Matrix ───────────────────────────────────────────────────────
cm = confusion_matrix(y_test, y_pred)
disp = ConfusionMatrixDisplay(cm, display_labels=data.target_names)
disp.plot(cmap='Blues')
plt.title("Confusion Matrix — Breast Cancer Dataset")
plt.tight_layout()
plt.show()

# ── 8. ROC Curve ──────────────────────────────────────────────────────────────
fpr, tpr, _ = roc_curve(y_test, y_prob)
plt.figure()
plt.plot(fpr, tpr, label=f"AUC = {roc_auc_score(y_test, y_prob):.3f}", lw=2)
plt.plot([0,1],[0,1],'--', color='gray', label='Random classifier')
plt.xlabel("False Positive Rate")
plt.ylabel("True Positive Rate")
plt.title("ROC Curve — Logistic Regression")
plt.legend()
plt.tight_layout()
plt.show()

# ── 9. Feature importance (coefficients) ─────────────────────────────────────
coef_df = pd.DataFrame({
    'feature': data.feature_names,
    'coefficient': model.coef_[0]
}).sort_values('coefficient', key=abs, ascending=False)
print("\nTop 10 most influential features:")
print(coef_df.head(10).to_string(index=False))
```

**Sample output:**
```
Accuracy : 0.9825
ROC-AUC  : 0.9971

Classification Report:
              precision    recall  f1-score   support
   malignant       0.98      0.98      0.98        42
      benign       0.99      0.99      0.99        72
```

---

## 7. Key Comparisons

### Logistic Regression vs Linear Regression

| Aspect | Linear Regression | Logistic Regression |
|---|---|---|
| **Task** | Regression (continuous output) | Classification (binary/multi-class) |
| **Output** | Any real number (−∞ to +∞) | Probability in (0, 1) |
| **Activation** | None (identity) | Sigmoid function |
| **Loss Function** | Mean Squared Error | Binary Cross-Entropy |
| **Decision** | Not applicable | Threshold on probability |

### Logistic Regression vs Other Classifiers

| Aspect | Logistic Regression | Decision Tree | SVM | Neural Network |
|---|---|---|---|---|
| **Interpretability** | ✅ Very high | ✅ High | ❌ Low | ❌ Very low |
| **Training speed** | ✅ Fast | ✅ Fast | ⚠️ Medium | ❌ Slow |
| **Non-linear boundaries** | ❌ No (linear only) | ✅ Yes | ✅ Yes (kernels) | ✅ Yes |
| **Feature scaling needed** | ✅ Yes | ❌ No | ✅ Yes | ✅ Yes |
| **Handles large features** | ✅ Yes | ⚠️ Moderate | ⚠️ Moderate | ✅ Yes |
| **Probabilistic output** | ✅ Yes (calibrated) | ❌ No | ❌ No | ✅ Yes |

---

## 8. The Problem of Overfitting

Overfitting happens when the model learns the training data so well — including its noise — that it performs poorly on unseen data. In logistic regression, overfitting is typically controlled via **regularization**.

### The Bias-Variance Tradeoff

![Overfitting vs Underfitting](/blogs/assests/ml-img/logistic-regression/overfitting.png)
*Left: Underfitting (too simple). Centre: Good fit. Right: Overfitting (too complex, memorizes noise). Logistic regression controls this via the C hyperparameter.*

In scikit-learn's `LogisticRegression`, the `C` parameter controls regularization:
- **Low C** (e.g., 0.001) → **Stronger regularization** → simpler model → risk of underfitting
- **High C** (e.g., 1000) → **Weaker regularization** → complex model → risk of overfitting
- **C = 1** → balanced default

### Regularization Types

| Type | Formula Penalty | Effect |
|---|---|---|
| **L2 (Ridge)** | $\lambda \sum w_j^2$ | Shrinks all weights uniformly — default in sklearn |
| **L1 (Lasso)** | $\lambda \sum \|w_j\|$ | Drives some weights to zero — built-in feature selection |
| **ElasticNet** | Combination of L1 + L2 | Best of both worlds |

```python
# Choosing regularization in scikit-learn
from sklearn.linear_model import LogisticRegressionCV

# Automatically cross-validates to find the best C
model_cv = LogisticRegressionCV(
    Cs=10,           # number of C values to try
    cv=5,            # 5-fold cross-validation
    penalty='l2',    # regularization type
    solver='lbfgs',
    max_iter=1000,
    random_state=42
)
model_cv.fit(X_train_s, y_train)
print(f"Best C found: {model_cv.C_[0]:.4f}")
print(f"Test Accuracy: {accuracy_score(y_test, model_cv.predict(X_test_s)):.4f}")
```

---

## 9. Strengths and Limitations

### Strengths

- **Probabilistic output** — you get confidence scores, not just labels
- **Highly interpretable** — coefficients directly tell you feature influence and direction
- **Computationally efficient** — scales well to large datasets
- **Baseline benchmark** — always try logistic regression first before complex models
- **Well-calibrated probabilities** — useful when you need reliable confidence estimates
- **Handles many features** — works well with high-dimensional data (text, etc.)

### Limitations

- **Linear decision boundary only** — cannot capture curved or complex separation patterns
- **Assumes feature independence** — multicollinearity can distort coefficient interpretation
- **Feature scaling required** — unscaled features lead to poor convergence
- **Not ideal for non-linear problems** — needs polynomial features or a kernel trick to handle them
- **Class imbalance sensitive** — needs careful handling via `class_weight='balanced'` or sampling

---

## 10. scikit-learn API Reference

| Parameter | Default | What it Controls |
|---|---|---|
| `penalty` | `'l2'` | Regularization type: `'l1'`, `'l2'`, `'elasticnet'`, `None` |
| `C` | `1.0` | Inverse of regularization strength |
| `solver` | `'lbfgs'` | Optimization algorithm |
| `max_iter` | `100` | Maximum iterations for convergence |
| `class_weight` | `None` | Set to `'balanced'` for imbalanced datasets |
| `multi_class` | `'auto'` | `'ovr'` or `'multinomial'` for multi-class |
| `random_state` | `None` | For reproducibility |

**Solver recommendations:**
- `lbfgs` → Default, good for most cases
- `saga` → Required for `l1` penalty, handles large datasets well
- `liblinear` → Small datasets, L1/L2
- `newton-cg` → L2 only, medium-large datasets

**Quick usage snippet:**

```python
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

# Best practice: always wrap in a Pipeline
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', LogisticRegression(C=1.0, solver='lbfgs', max_iter=1000, random_state=42))
])
pipe.fit(X_train, y_train)
print(f"Accuracy: {pipe.score(X_test, y_test):.4f}")
```

---

## 11. Glossary

| Term | Definition |
|---|---|
| **Sigmoid function** | Maps any real number to (0, 1); the core activation of logistic regression |
| **Log-Loss** | Binary cross-entropy loss function; penalizes confident wrong predictions heavily |
| **Decision boundary** | The hyperplane that separates predicted class 0 from class 1 |
| **Gradient descent** | Iterative optimization algorithm that minimizes the cost function |
| **Learning rate (α)** | Controls how large each weight update step is |
| **Regularization** | Penalty on large weights to prevent overfitting |
| **L1 / Lasso** | Regularization that sets some weights to zero (sparse solution) |
| **L2 / Ridge** | Regularization that shrinks all weights but rarely zeros them |
| **C (scikit-learn)** | Inverse of regularization strength; higher C = less regularization |
| **ROC-AUC** | Area under the ROC curve; measures classifier performance across all thresholds |
| **Bias-variance tradeoff** | Balance between underfitting (high bias) and overfitting (high variance) |
| **Binary cross-entropy** | Another name for log-loss; standard loss for binary classification |

---

## 12. Further Reading

- **Andrew Ng's Machine Learning Specialization** (Coursera) — covers logistic regression exhaustively, including the cost function derivation and gradient descent
- **scikit-learn Documentation** — [LogisticRegression API](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html)
- **The Elements of Statistical Learning** (Hastie, Tibshirani, Friedman) — rigorous mathematical treatment in Chapter 4
- **Pattern Recognition and Machine Learning** (Bishop) — probabilistic perspective on logistic regression
- **Hands-On Machine Learning with Scikit-Learn, Keras & TensorFlow** (Aurélien Géron) — practical, code-first coverage

---

*If this post helped you understand logistic regression more deeply, consider sharing it with a peer or classmate who is navigating the same concepts. Good ML fundamentals compound — every algorithm you master cleanly makes the next one easier.*
