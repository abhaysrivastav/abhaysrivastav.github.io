---
layout: post
title: "Decision Trees: A Complete Conceptual Guide"
date: 2026-03-14
categories: [machine-learning, algorithms]
tags: [decision-tree, supervised-learning, classification, regression, ml-series]
author: "Abhay Srivastav"
description: "A precise, concept-first guide to Decision Trees — how they learn, how they split, how they fail, and when to use them. No fluff, no projects."
---

A decision tree is a supervised learning algorithm that makes predictions by learning a hierarchy of binary questions from training data. Each question tests a feature. Each answer leads to another question or a final prediction. The result is a model you can read, trace, and explain — one decision at a time.

This post covers how decision trees work from the ground up: the splitting criteria, the learning algorithm, the failure modes, and where they fit in the broader ML landscape.

---

## The structure of a decision tree

A trained decision tree is a binary tree where every internal node tests a condition on one feature, every branch is the outcome of that test, and every leaf holds a prediction.

```
Is feature A ≤ threshold?
├── Yes → Is feature B ≤ threshold?
│         ├── Yes → Class 0
│         └── No  → Class 1
└── No  → Is feature C ≤ threshold?
          ├── Yes → Class 1
          └── No  → Class 0
```

Four components matter:

| Component | Role |
|-----------|------|
| **Root node** | The first split — the most informative feature across the entire dataset |
| **Internal node** | Any non-leaf node — tests one feature, produces two children |
| **Branch** | The outcome of a test: left for true, right for false |
| **Leaf node** | Terminal node — holds the predicted class (classification) or mean value (regression) |

**Depth** is the number of edges from root to the deepest leaf. Depth controls complexity more than any other structural property.

---

## How the tree is built: the learning algorithm

Decision trees are built using a **greedy, recursive, top-down** algorithm. At each node it does one thing: find the single best split across all features and all thresholds, make that split, then repeat on each child.

It is greedy because it optimises locally at each node — it does not backtrack to reconsider earlier splits. It is recursive because each child node runs the same procedure on its subset of data. It stops when a stopping condition is met: maximum depth reached, too few samples to split, or no split improves purity.

The pseudocode:

```
function BuildTree(data, depth):
    if StoppingCondition(data, depth):
        return Leaf(majority_class(data))

    best_feature, best_threshold = FindBestSplit(data)
    left_data, right_data = Split(data, best_feature, best_threshold)

    return Node(
        test    = (best_feature ≤ best_threshold),
        left    = BuildTree(left_data,  depth + 1),
        right   = BuildTree(right_data, depth + 1)
    )
```

The entire learning problem reduces to one question: **what makes a split "best"?**

---

## Splitting criteria

A good split produces children that are more homogeneous than the parent — where each child contains mostly one class rather than a mixture. Two metrics measure this.

### Gini impurity

Gini impurity measures the probability that a randomly chosen sample from a node would be misclassified if labelled according to the node's class distribution.

$$
\text{Gini}(t) = 1 - \sum_{i=1}^{C} p(i \mid t)^2
$$

Where $p(i \mid t)$ is the proportion of class $i$ at node $t$ and $C$ is the number of classes.

Boundary values:
- **Gini = 0.0** — node is perfectly pure (all one class)
- **Gini = 0.5** — maximum impurity for binary classification (50/50 split)

When evaluating a candidate split into left ($L$) and right ($R$) children:

$$
\text{Gini}_{\text{split}} = \frac{n_L}{n} \cdot \text{Gini}(L) + \frac{n_R}{n} \cdot \text{Gini}(R)
$$

The split that minimises this weighted Gini is chosen. This is the criterion used by **CART** and by scikit-learn's `DecisionTreeClassifier` by default.

**Worked example.** Node with 100 samples: 60 class A, 40 class B.

- Node Gini = 1 − (0.6² + 0.4²) = 1 − 0.52 = **0.48**

Candidate split produces:
- Left: 80 samples (55A, 25B) → Gini = 1 − (0.69² + 0.31²) = **0.428**
- Right: 20 samples (5A, 15B) → Gini = 1 − (0.25² + 0.75²) = **0.375**
- Weighted Gini = (0.8 × 0.428) + (0.2 × 0.375) = **0.417**

The split reduced impurity from 0.48 to 0.417. The algorithm evaluates every feature and every threshold this way and selects the minimum.

---

### Entropy and information gain

Entropy is the information-theoretic measure of disorder in a node. A pure node has entropy 0. A maximally mixed node has entropy log₂C.

$$
H(t) = -\sum_{i=1}^{C} p(i) \log_2 p(i)
$$

**Information gain** measures how much a split reduces entropy:

$$
\text{IG}(S, A) = H(S) - \sum_{v} \frac{|S_v|}{|S|} \cdot H(S_v)
$$

Where $S$ is the parent node dataset, $A$ is the feature being evaluated, and $S_v$ is the subset where feature $A$ takes value $v$.

The split that maximises information gain is chosen. This is the criterion used by **ID3**.

---

### Gain ratio

Information gain has a known flaw: it favours features with many distinct values. A feature with a unique value per sample (like a customer ID) will always score highest, even though it has zero predictive value.

C4.5 corrects this with **gain ratio**, which normalises information gain by the split's own entropy (called split information):

$$
\text{SplitInfo}(S, A) = -\sum_{v} \frac{|S_v|}{|S|} \log_2 \frac{|S_v|}{|S|}
$$

$$
\text{GainRatio}(S, A) = \frac{\text{IG}(S, A)}{\text{SplitInfo}(S, A)}
$$

A feature that creates many small children will have high SplitInfo, which penalises its GainRatio. This correction makes gain ratio a fairer criterion when features have varying cardinality.

---

### Criterion comparison

| Criterion | Algorithm | Range | Favours | Weakness |
|-----------|-----------|-------|---------|----------|
| Gini impurity | CART | 0 → 0.5 | Isolating the majority class | Slightly biased to frequent class |
| Entropy / IG | ID3 | 0 → log₂C | Balanced splits | Biased to high-cardinality features |
| Gain ratio | C4.5 | 0 → 1 | Balanced splits with cardinality correction | More complex to compute |
| MSE reduction | CART (regression) | 0 → ∞ | Low-variance children | Sensitive to outliers |

In practice, Gini and entropy select the same split in the vast majority of cases. The choice of criterion has far less impact on final model performance than the choice of depth and pruning parameters.

---

## Regression trees

Everything above applies to classification. For regression, the setup changes only in what goes in each leaf and how splits are scored.

**Leaf value:** The mean of target values in that node (sometimes median for robustness).

**Split criterion:** Mean Squared Error (MSE) reduction — pick the split that maximises the drop in variance:

$$
\text{MSE}(t) = \frac{1}{n_t} \sum_{i \in t} (y_i - \bar{y}_t)^2
$$

$$
\Delta\text{MSE} = \text{MSE}(S) - \left(\frac{n_L}{n} \cdot \text{MSE}(L) + \frac{n_R}{n} \cdot \text{MSE}(R)\right)
$$

The split that maximises ΔMSE is chosen. This is equivalent to minimising the weighted sum of child variances.

**Important limitation:** A regression tree produces a step function — the prediction is constant within each leaf region. It cannot extrapolate beyond the range of training values, and it approximates smooth curves poorly without very deep trees (which overfit).

---

## Overfitting and pruning

An unconstrained decision tree will grow until every leaf contains a single training sample. Training accuracy reaches 100%. Generalisation collapses. This is overfitting — the tree has memorised noise rather than learned the underlying pattern.

The bias-variance tradeoff is stark here:
- **Too shallow** → high bias, underfits, misses real patterns
- **Too deep** → high variance, overfits, memorises noise
- **Well-pruned** → balanced, generalises to unseen data

### Pre-pruning

Stop the tree from growing beyond what is useful. Controlled by hyperparameters set before training:

| Hyperparameter | Effect |
|----------------|--------|
| `max_depth` | Hard cap on tree depth. Most impactful single parameter. Start at 3–5. |
| `min_samples_split` | Minimum samples required at a node before it can split |
| `min_samples_leaf` | Minimum samples required at any leaf |
| `min_impurity_decrease` | Only split if impurity drops by at least this amount |
| `max_features` | Number of features considered per split — adds randomness |

### Post-pruning: cost-complexity pruning

Grow the full tree, then remove subtrees that offer insufficient accuracy per unit of complexity. Controlled by the parameter $\alpha$ (ccp_alpha in scikit-learn).

The cost-complexity measure of a subtree $T$:

$$
R_\alpha(T) = R(T) + \alpha \cdot |T|
$$

Where $R(T)$ is the tree's misclassification rate, $|T|$ is the number of leaves, and $\alpha$ is the complexity penalty. Increasing $\alpha$ prunes more aggressively.

```python
from sklearn.tree import DecisionTreeClassifier

# Step 1: find the pruning path
clf = DecisionTreeClassifier(random_state=42)
path = clf.cost_complexity_pruning_path(X_train, y_train)
ccp_alphas = path.ccp_alphas[:-1]  # exclude the degenerate root-only tree

# Step 2: train one tree per alpha value
clfs = [DecisionTreeClassifier(ccp_alpha=a, random_state=42).fit(X_train, y_train)
        for a in ccp_alphas]

# Step 3: pick the alpha with the best cross-validated test score
test_scores = [c.score(X_test, y_test) for c in clfs]
best_alpha = ccp_alphas[test_scores.index(max(test_scores))]
```

---

## Feature importance

After training, each feature receives an importance score equal to its total weighted impurity reduction across all splits in the tree:

$$
\text{Importance}(f) = \sum_{\text{node } t \text{ splits on } f} \frac{n_t}{n} \cdot \Delta\text{Impurity}(t)
$$

Importances are normalised to sum to 1. A feature that appears at shallower nodes (closer to the root) and splits large amounts of data will have high importance. This provides a direct measure of which features the model relied on most.

```python
import pandas as pd

importances = pd.Series(
    clf.feature_importances_,
    index=feature_names
).sort_values(ascending=False)
```

**Caveat:** Feature importance from a single tree is unstable — it changes significantly with different training samples. Random Forest importances, averaged across many trees, are considerably more reliable.

---

## Algorithm comparison

| Algorithm | Year | Criterion | Features | Pruning | Multi-way splits |
|-----------|------|-----------|----------|---------|-----------------|
| **ID3** | 1986 | Information gain | Categorical only | None | Yes |
| **C4.5** | 1993 | Gain ratio | Mixed | Post-pruning (error-based) | Yes |
| **CART** | 1984 | Gini / MSE | Mixed | Pre + cost-complexity | Binary only |
| **C5.0** | 1997 | Gain ratio | Mixed | Post-pruning | Yes |

CART is the algorithm implemented in scikit-learn. It produces strictly binary trees — every internal node has exactly two children. ID3 and C4.5 support multi-way splits, which can make trees shallower but harder to read and more prone to overfitting on high-cardinality features.

---

## Strengths and limitations

### Strengths

- **Interpretable.** Every prediction traces to an explicit sequence of feature tests. Any path from root to leaf is a human-readable rule.
- **No feature scaling.** Splits are threshold comparisons — the absolute scale of a feature does not affect the result.
- **Mixed feature types.** Handles numerical and categorical features natively (with appropriate encoding).
- **Captures feature interactions.** A split on feature B that only appears in the left subtree of feature A implicitly captures the interaction A ∧ B.
- **Fast training and inference.** Training is O(n · p · log n) for n samples and p features. Inference is O(depth).

### Limitations

- **High variance.** The tree structure is sensitive to small changes in training data. Different random seeds or data subsets can produce very different trees.
- **Axis-aligned splits only.** Every split is a threshold on a single feature. Diagonal or curved decision boundaries require many splits to approximate.
- **Overfits without pruning.** An unconstrained tree will always memorise the training set.
- **Instability under class imbalance.** Without class weighting, trees optimise for the majority class. Use `class_weight='balanced'` in scikit-learn.
- **Poor extrapolation in regression.** Predictions are bounded by the training range. A regression tree cannot predict values outside the range it has seen.

---

## Scikit-learn reference

```python
from sklearn.tree import DecisionTreeClassifier, DecisionTreeRegressor
from sklearn.tree import export_text, plot_tree

# Classification
clf = DecisionTreeClassifier(
    criterion='gini',          # 'gini' or 'entropy'
    max_depth=5,
    min_samples_split=20,
    min_samples_leaf=10,
    min_impurity_decrease=0.0,
    class_weight='balanced',   # use when classes are imbalanced
    ccp_alpha=0.0,             # post-pruning — tune via cross-validation
    random_state=42
)

# Regression
reg = DecisionTreeRegressor(
    criterion='squared_error',  # 'squared_error', 'friedman_mse', 'absolute_error'
    max_depth=5,
    min_samples_leaf=10,
    random_state=42
)

# Inspect the learned rules — works for both
print(export_text(clf, feature_names=feature_names))

# Visualise
plot_tree(clf, feature_names=feature_names,
          class_names=class_names, filled=True)
```

---

## From tree to ensemble

A single decision tree's high variance is its fundamental weakness. Three ensemble methods address this directly, each with a different mechanism:

| Method | Mechanism | Reduces | Notes |
|--------|-----------|---------|-------|
| **Random Forest** | Bagging + random feature subsets | Variance | Parallel trees, majority vote |
| **AdaBoost** | Sequential reweighting of misclassified samples | Bias | Uses depth-1 stumps by default |
| **Gradient Boosting** | Sequential trees fitting residuals of the previous | Bias | XGBoost, LightGBM, CatBoost |

Random Forest is the natural extension of a single tree when variance is the problem. Gradient Boosting is the right choice when you need maximum predictive performance and interpretability is less critical.

---

## Glossary

| Term | Definition |
|------|-----------|
| **Root node** | The topmost node — the first and most informative split |
| **Internal node** | Any non-leaf node that tests a feature condition |
| **Leaf node** | Terminal node holding the final prediction |
| **Depth** | Length of the longest path from root to any leaf |
| **Gini impurity** | Probability of misclassifying a random sample; range 0–0.5 (binary) |
| **Entropy** | Information-theoretic disorder; 0 = pure, log₂C = maximum |
| **Information gain** | Entropy reduction achieved by a split |
| **Gain ratio** | Information gain normalised by split information |
| **Pruning** | Removing branches to reduce overfitting |
| **ccp_alpha** | Cost-complexity pruning parameter in scikit-learn |
| **Feature importance** | Weighted impurity reduction attributed to each feature |
| **Bootstrap sample** | Random sample with replacement; ~63% unique samples |
| **OOB score** | Out-of-bag validation estimate — available in Random Forest |
| **Bagging** | Bootstrap Aggregating — training each model on a resampled dataset |

---

## Further reading

- Breiman, L. et al. (1984). *Classification and Regression Trees.* Wadsworth.
- Quinlan, J.R. (1986). Induction of Decision Trees. *Machine Learning*, 1(1), 81–106.
- Quinlan, J.R. (1993). *C4.5: Programs for Machine Learning.* Morgan Kaufmann.
- Breiman, L. (2001). Random Forests. *Machine Learning*, 45(1), 5–32.
- scikit-learn: [Decision Trees](https://scikit-learn.org/stable/modules/tree.html)
- scikit-learn: [Cost-Complexity Pruning](https://scikit-learn.org/stable/auto_examples/tree/plot_cost_complexity_pruning.html)


