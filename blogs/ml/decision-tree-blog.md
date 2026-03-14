---
layout: topic
title: "Decision Trees in Machine Learning"
permalink: /blogs/decision-tree-blog/
date: 2026-03-14
categories: [machine-learning, algorithms]
tags: [decision-tree, supervised-learning, classification, ml-series]
description: "A humanized, story-driven introduction to Decision Trees — from Alice's crossroads to Sherlock's deductions — for students at all levels."
---

> *"Would you tell me, please, which way I ought to go from here?"*
> *"That depends a good deal on where you want to get to," said the Cat.*
>
> — Lewis Carroll, *Alice's Adventures in Wonderland*

---

Imagine you're standing at a crossroads in a dark wood. One path leads uphill into the unknown; another winds down into a familiar valley. You don't flip a coin. You ask questions. *How tired am I? What time is it? Did I hear wolves?* Each answer narrows your options until — almost without effort — the right path reveals itself.

This, in essence, is a **Decision Tree** — perhaps the most human of all machine learning algorithms. Long before computers were dreaming of electric sheep, decision trees were living in our fairy tales, our courtrooms, our medical textbooks. We've always thought in branches.

Alice's exchange with the Cheshire Cat is the perfect metaphor for supervised classification. The destination (your label) shapes every fork in the road. A Decision Tree is simply a machine that has learned which questions to ask — and in what order — to get you to the right destination.

![Alice at a fork in the road — Tenniel illustration](/assets/ml-images/alice-crossroads.png)
*John Tenniel's original illustration from Alice's Adventures in Wonderland (1865) — the original decision-point in literature.*

---

## What Exactly Is a Decision Tree?

> 🟢 **Level: Beginner**

A Decision Tree is a **flowchart-shaped supervised learning model**. It learns from labelled training data to build a tree structure — a hierarchy of questions called *nodes* and answers called *branches* that ultimately lead to a prediction called a *leaf*.

The tree repeatedly asks: *"Which feature, if split on, gives us the most information about the correct answer?"* — growing branches until it reaches a confident decision or runs out of things to ask.

### Anatomy of a Decision Tree

| Component | Description | Example |
|-----------|-------------|---------|
| **Root Node** | The very first split — the most informative question | "Is it raining?" |
| **Internal Node** | An intermediate question based on a feature | "Do you have an umbrella?" |
| **Branch** | The outcome of each test (Yes / No, > , ≤) | "Yes → go left" |
| **Leaf Node** | Terminal node — the final prediction | "Go out ✓" or "Stay home ✗" |
| **Depth** | Number of levels from root to deepest leaf | Depth 3 = 3 questions max |

### A Simple Example

```
Is it raining?
├── Yes → Do you have an umbrella?
│         ├── Yes → Go out ✓
│         └── No  → Stay home ✗
└── No  → Is it a weekend?
          ├── Yes → Go to the park ✓
          └── No  → Work from home ✓
```

Notice how the tree doesn't ask *everything at once*. It prioritises. It starts with the most discriminating question — the one that separates the data most cleanly — and works its way down. This greedy, recursive elegance is what makes decision trees both powerful and surprisingly interpretable.

---

## Sherlock Holmes Was a Decision Tree

> *"When you have eliminated the impossible, whatever remains, however improbable, must be the truth."*
>
> — Arthur Conan Doyle, *The Sign of the Four*

> 🟢 **Level: Beginner**

Sir Arthur Conan Doyle's genius detective didn't have a random forest at his disposal. But he had something just as powerful: the ability to ask the right question, observe the split, and recurse.

Holmes's method is pure decision-tree logic. Upon meeting Watson for the first time, he didn't dump all observations simultaneously. He sequenced them:

```
Tan line?
└── Yes → Afghan service?
          └── Yes → Arm injury?
                    └── Yes → Military doctor ← Final Prediction
```

Each observation was a node split, narrowing the hypothesis space until only one leaf remained.

![Sidney Paget's iconic illustration of Sherlock Holmes](/assets/ml-images/sherlock-holmes-paget.jpg)
*Sidney Paget's iconic 1904 illustration of Sherlock Holmes — the world's most famous decision-tree practitioner.*

In machine learning terms, Holmes was doing **classification**. The *features* were the visible clues. The *target variable* was the truth he sought. His mind was the trained model. And his confidence? That came from good training data — thousands of cases observed over a career.

---

## How Does the Tree Actually Learn? The Mathematics

> 🟡 **Level: Intermediate**

Behind the elegant branching lies one key idea: *reducing uncertainty*. At each node, the tree evaluates every possible split on every feature, and picks the one that makes child nodes most "pure" — i.e., where the class labels are least mixed up.

### Gini Impurity — The Messy Room Test

Imagine a room full of coloured marbles. If every marble is red, the room is "pure" — **Gini = 0**. If marbles are split evenly between red and blue, the room is maximally impure — **Gini = 0.5**. The tree tries to split data into rooms that are as pure as possible.

$$
\text{Gini}(t) = 1 - \sum_{i} [p(i \mid t)]^2
$$

Where:
- **p(i|t)** = proportion of class *i* at node *t*
- **Gini = 0** → perfect purity (all same class)
- **Gini = 0.5** → maximum chaos (binary, 50/50 split)

**Worked Example:**
- Node with 70 class-A, 30 class-B samples
- Gini = 1 − (0.7² + 0.3²) = 1 − (0.49 + 0.09) = **0.42**

### Information Gain — The Shannon Metric

> 🔴 **Level: Advanced**

Rooted in Claude Shannon's information theory, **entropy** measures randomness in your data. A perfect split that separates all cats from all dogs yields maximum information gain. The **ID3 algorithm** uses this to choose splits.

$$
\text{Entropy}(S) = -\sum_{i} p(i) \cdot \log_2 p(i)
$$

$$
\text{InfoGain}(S, A) = \text{Entropy}(S) - \sum_{v} \frac{|S_v|}{|S|} \cdot \text{Entropy}(S_v)
$$

Where:
- **S** = dataset at the parent node
- **A** = feature being evaluated for the split
- **S_v** = subset where feature A = value *v*

### Which Criterion to Use?

| Criterion | Algorithm | Best For | Watch Out |
|-----------|-----------|----------|-----------|
| **Gini Impurity** | CART (scikit-learn default) | Speed, binary classification | Slightly biased to frequent class |
| **Information Gain** | ID3 | Theoretical grounding | Biased to high-cardinality features |
| **Gain Ratio** | C4.5 | Correcting cardinality bias | More complex to compute |
| **MSE / MAE** | CART (regression) | Continuous target variable | — |

---

## Dante's Dark Forest and the Problem of Overfitting

> *"Midway upon the journey of our life I found myself within a forest dark, for the straightforward pathway had been lost."*
>
> — Dante Alighieri, *Inferno*, Canto I (trans. Longfellow)

> 🟡 **Level: Intermediate**

A tree that grows too deep memorises Dante's every footstep — every specific quirk of the training data — rather than learning the general path. This is called **overfitting**: the model performs brilliantly on training data and fails on anything new.

![William Blake's watercolour illustration of Dante's Inferno](/assets/ml-images/blake-dante-inferno.jpg)
*William Blake's watercolour of Dante and Virgil beginning their journey, c. 1824 — navigating the dark forest of training data noise.*

Just as Dante needed Virgil to guide him and cut through the chaos, a decision tree needs **pruning** to cut back branches that memorise noise rather than learn truth.

### Pruning Strategies

**Pre-pruning (Early Stopping)** — Stop the tree from growing beyond what generalises:

| Hyperparameter | What it controls |
|----------------|-----------------|
| `max_depth` | Maximum depth of the tree |
| `min_samples_split` | Min samples required to attempt a split |
| `min_samples_leaf` | Min samples required at a leaf node |
| `min_impurity_decrease` | Min impurity drop to justify a split |

**Post-pruning (Cost-Complexity Pruning)** — Grow the full tree, then cut back:

```python
from sklearn.tree import DecisionTreeClassifier

# Find the best ccp_alpha via cross-validation
path = clf.cost_complexity_pruning_path(X_train, y_train)
ccp_alphas = path.ccp_alphas

# Train trees across alpha values and pick the best
clfs = [DecisionTreeClassifier(ccp_alpha=a).fit(X_train, y_train)
        for a in ccp_alphas]
```

---

## Strengths and Weaknesses

> 🟢 **Level: Beginner**

### ✅ Strengths

- **Interpretable** — You can literally read every path the tree takes. No black box.
- **No feature scaling needed** — Unlike SVM or KNN, raw features work fine.
- **Handles mixed data types** — Numerical and categorical features together.
- **Captures feature interactions** — Naturally handles if *feature A AND feature B* together predict the outcome.
- **Built-in feature importance** — The tree tells you which features it relied on most.
- **Fast** — Both training and prediction are computationally cheap.

### ⚠️ Weaknesses

- **Prone to overfitting** — Especially on noisy data without pruning.
- **High variance** — Small changes in training data can produce a completely different tree.
- **Axis-aligned splits only** — Cannot create diagonal decision boundaries naturally.
- **Biased toward high-cardinality features** — Features with many distinct values get unfair preference in Information Gain.
- **Not great for regression** — Produces step-function outputs, not smooth curves.

---

## Algorithm Comparison

> 🔴 **Level: Advanced**

| Algorithm | Year | Split Criterion | Features | Pruning | Notes |
|-----------|------|----------------|----------|---------|-------|
| **ID3** | 1986 | Information Gain | Categorical only | None | Origin; simple |
| **C4.5** | 1993 | Gain Ratio | Mixed | Post-pruning | Fixes ID3 cardinality bias |
| **CART** | 1984 | Gini / MSE | Mixed | Pre + Post | scikit-learn implementation |
| **C5.0** | 1997 | Gain Ratio | Mixed | Post-pruning | Commercial; adds boosting |

---

## One Tree Is Good. A Forest Is Better.

> *"No man is an island, entire of itself; every man is a piece of the continent, a part of the main."*
>
> — John Donne, *Devotions Upon Emergent Occasions* (1624)

> 🔴 **Level: Advanced**

A single decision tree, for all its beauty, is fragile. Change a few training examples and you might get a completely different tree. The machine learning community solved this with an elegant idea: **why not grow a thousand trees, let each vote, and go with the majority?**

This is the **Random Forest** — an ensemble of decision trees, each trained on a random subset of data and features. The individual trees disagree on details but agree on the big picture. Their collective wisdom is remarkably robust.

### How Random Forest Works

1. **Bootstrap Sampling** — Each tree trains on a random sample drawn *with replacement* from the training set (~63% of data per tree; rest is "out-of-bag").
2. **Feature Randomness** — At each split, only **√p** random features are considered (where *p* = total features). This decorrelates the trees.
3. **Aggregation** — Classification: majority vote. Regression: mean prediction.
4. **OOB Error** — The out-of-bag samples provide a built-in validation estimate at no extra cost.

### Beyond Random Forest: The Ensemble Family

| Method | Strategy | Reduces | Example |
|--------|----------|---------|---------|
| **Random Forest** | Parallel trees + bagging | Variance | `RandomForestClassifier` |
| **AdaBoost** | Sequential stumps, weighted | Bias | `AdaBoostClassifier` |
| **Gradient Boosting** | Sequential, corrects residuals | Bias | `GradientBoostingClassifier` |
| **XGBoost / LightGBM** | Optimised gradient boosting | Bias + speed | `xgb.XGBClassifier` |

---

## Scikit-learn Quick Reference

```python
from sklearn.tree import DecisionTreeClassifier, plot_tree
from sklearn.ensemble import RandomForestClassifier
import matplotlib.pyplot as plt

# ── Single Decision Tree ──────────────────────────────────
clf = DecisionTreeClassifier(
    criterion='gini',          # or 'entropy'
    max_depth=5,               # pre-pruning: max depth
    min_samples_split=10,      # pre-pruning: min samples to split
    min_samples_leaf=5,        # pre-pruning: min samples at leaf
    ccp_alpha=0.01,            # post-pruning: cost-complexity
    random_state=42
)
clf.fit(X_train, y_train)
print(f"Test Accuracy: {clf.score(X_test, y_test):.3f}")

# ── Visualise the Tree ────────────────────────────────────
plt.figure(figsize=(20, 10))
plot_tree(clf, feature_names=feature_names,
          class_names=class_names, filled=True)
plt.savefig('decision_tree.png', dpi=150, bbox_inches='tight')

# ── Random Forest ─────────────────────────────────────────
rf = RandomForestClassifier(
    n_estimators=200,          # number of trees
    max_depth=10,
    max_features='sqrt',       # features per split = √p
    oob_score=True,            # free validation on OOB samples
    random_state=42
)
rf.fit(X_train, y_train)
print(f"OOB Score: {rf.oob_score_:.3f}")

# ── Feature Importance ────────────────────────────────────
importances = rf.feature_importances_
```

---

## Key Takeaways

> **For beginners:** A decision tree is a flowchart that the machine learns by asking "which question best separates my data?" at each step.

> **For intermediate:** The split criterion (Gini or Information Gain) measures node purity. Pruning controls overfitting. Depth is your main lever.

> **For advanced:** CART produces binary splits using Gini/MSE. C4.5's Gain Ratio corrects cardinality bias. Random Forests reduce variance via bagging + feature randomness. Gradient Boosted Trees reduce bias via sequential residual correction.

---

## Glossary

| Term | Definition |
|------|-----------|
| **Root Node** | First and most informative split in the tree |
| **Internal Node** | Any non-leaf node that tests a feature condition |
| **Leaf Node** | Terminal node holding the final prediction |
| **Gini Impurity** | Measure of node impurity; range 0 (pure) → 0.5 (max, binary) |
| **Entropy** | Information-theoretic disorder measure; 0 = pure node |
| **Information Gain** | Entropy reduction achieved by a split; maximised by ID3 |
| **Gain Ratio** | Information Gain normalised by split information; used by C4.5 |
| **Pruning** | Removing branches to reduce overfitting |
| **Overfitting** | Model memorises training noise; high train, low test accuracy |
| **Bagging** | Bootstrap Aggregating — training each tree on a resampled dataset |
| **Random Forest** | Ensemble of bagged decision trees with feature randomness |
| **CART** | Classification and Regression Trees — scikit-learn's algorithm |
| **OOB Error** | Out-of-bag validation error — free estimate using unused bootstrap samples |
| **Feature Importance** | Each feature's contribution to reducing impurity across all splits |

---

## Further Reading

- Breiman, L. et al. (1984). *Classification and Regression Trees* (CART).
- Quinlan, J.R. (1986). *Induction of Decision Trees* (ID3). Machine Learning, 1(1).
- Quinlan, J.R. (1993). *C4.5: Programs for Machine Learning*.
- Breiman, L. (2001). *Random Forests*. Machine Learning, 45(1), 5–32.
- scikit-learn documentation: [Decision Trees](https://scikit-learn.org/stable/modules/tree.html)

