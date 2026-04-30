---
layout: topic
title: "Class Imbalance — Why Accuracy Can Lie"
permalink: /blogs/class-imbalance/
date: 2025-04-30
categories: [machine-learning, classification]
tags: [class-imbalance, fraud-detection, imbalanced-data, precision-recall, smote, threshold-tuning]
author: Abhay
math: true
image: /blogs/assests/ml-img/imbalance/01_class_distribution.png
description: "A practical guide to class imbalance, showing why accuracy fails, which metrics to use instead, and how to fix the problem with class weighting, SMOTE, threshold tuning, and ensembles."
---

# When Your Model Is 98% Accurate and Completely Useless

*A story about the mistake I almost shipped to production — and how to never make it yourself.*

---

I remember the exact moment.

It was a Friday afternoon, the kind where you're already mentally halfway out the door. I'd spent three weeks building a fraud detection model. The numbers looked great. Accuracy: **98.7%**. I was proud of it. The kind of quiet pride you feel when the work just... comes together.

I almost deployed it without a second thought.

A colleague stopped by my desk, looked at the confusion matrix I'd half-minimized, and said — almost casually — *"How many frauds is it actually catching?"*

Zero. It was catching zero.

The model had figured out the laziest possible strategy: predict every single transaction as legitimate, score 98.7% accuracy, and call it a day. And I had been about to reward it for that.

That was my introduction to **class imbalance** — and I've never looked at an accuracy score the same way since.

---

## First, let's understand why this happens

Say you have 10,000 transactions. 9,800 are legitimate. 200 are fraudulent. If I build the dumbest model imaginable — one that just says "legitimate" no matter what — it's right 9,800 out of 10,000 times.

That's 98% accuracy. By just... doing nothing.

![Class Imbalance Distribution](/blogs/assests/ml-img/imbalance/01_class_distribution.png)

And here's the thing — this isn't a fraud-only problem. Look at the chart on the right. Medical diagnosis, equipment failure detection, churn prediction — nearly every domain where the stakes are highest is also the domain where your minority class is most scarce. The rare thing is usually the important thing. And accuracy will never tell you you're missing it.

This is what makes class imbalance so dangerous. It doesn't feel like a problem. The numbers look fine. The model trains fast. Everything seems okay until it isn't.

---

## Step 1: Stop measuring the wrong thing

Before you touch a single line of preprocessing code, there's one change that costs you nothing and immediately makes the problem visible: **switch your evaluation metric**.

Accuracy is lying to you. Here's what to use instead.

![Accuracy Paradox](/blogs/assests/ml-img/imbalance/02_accuracy_paradox.png)

Look at the confusion matrix on the left. That's our "98% accurate" model. TN (true negatives) is a beautiful 9,800. TP (true positives — actual fraud caught)? Zero. Every single fraudulent transaction slipped through. The bar chart on the right shows what actually happens once you apply real fixes — F1 goes from 4% to 76%. Recall on fraud goes from 0% to 83%.

Same data. Same underlying problem. The difference is whether you could *see* the failure.

**Here's how I think about the three metrics that actually matter:**

**Precision** is about trust. If your model raises an alarm, how often is it right? Nobody wants their legitimate transactions getting flagged every other day.

$$\text{Precision} = \frac{TP}{TP + FP}$$

**Recall** is about safety. Of all the actual fraud that happened, how much did you catch? This is the one that keeps the fraud team up at night.

$$\text{Recall} = \frac{TP}{TP + FN}$$

**F1-Score** is the peacemaker. It forces you to balance both — and it falls sharply if either one collapses.

$$\text{F1} = 2 \times \frac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}$$

### But there's something even better than F1

Once I started digging deeper, I found that ROC-AUC — which most tutorials push as the go-to metric — has the same blind spot as accuracy when data is imbalanced.

![PR vs ROC Curves](/blogs/assests/ml-img/imbalance/03_pr_roc_curves.png)

See how on the right (ROC), the weak model still scores a comfortable 0.88? It *looks* fine. It would pass most code reviews without a question. But on the left, that same model's Precision-Recall curve collapses to 0.22 — barely above a random baseline.

**PR-AUC doesn't let bad models hide.** It's the metric I reach for first on any imbalanced dataset now.

> Quick rule: Use **F1** when you need one number to watch. Use **PR-AUC** when you want the full picture of how your model behaves across all possible thresholds.

---

## Step 2: Class weighting — the fix nobody tries first but should

Here's what I wish someone had told me early on: before you get into resampling techniques, SMOTE, ensembles — try class weighting. It takes one line of code and it works surprisingly often.

The intuition is simple. During training, your model is minimizing loss. With 9,800 legitimate transactions and 200 fraudulent ones, missing a fraud costs the same as misclassifying a legitimate transaction. So the model just... ignores the fraud. Why bother when there's so little of it?

Class weighting changes that deal. You're telling the model: *"Every time you miss a fraud, that mistake costs you 49 times more."* Suddenly it pays attention.

```python
from sklearn.linear_model import LogisticRegression

# That's genuinely all it takes to start
model = LogisticRegression(class_weight='balanced')
```

The math behind `balanced` is straightforward:

$$w_j = \frac{n\_samples}{n\_classes \times n\_samples_j}$$

Rarer class = higher weight. sklearn computes all of this for you.

Here's where it fits in the overall strategy:

| Your imbalance ratio | What I'd try |
|---------------------|--------------|
| 1:2 to 1:10 | Class weighting alone — probably enough |
| 1:10 to 1:100 | Class weighting + threshold tuning |
| 1:100 and beyond | Needs resampling too |

Don't skip this to go straight to SMOTE because it feels more sophisticated. I've seen class weighting solve problems that were being massively over-engineered.

---

## Step 3: When you need more — enter SMOTE

Sometimes the imbalance is severe enough that reweighting the loss isn't enough. The model just doesn't have enough examples of the minority class to learn what it actually looks like. That's when you reach for resampling.

**SMOTE** (Synthetic Minority Oversampling Technique) is the most well-known approach, and it's genuinely clever. Instead of just copying existing minority samples — which adds no new information — it *generates* new ones by interpolating between the ones that exist.

![SMOTE Visualization](/blogs/assests/ml-img/imbalance/04_smote_visualization.png)

Look at the left plot. 15 fraud samples. 300 legitimate ones. There's almost nothing for the model to learn patterns from. On the right, SMOTE has created synthetic fraud samples (the green stars) by drawing points along the line between neighboring fraud cases. The model now has enough signal to actually distinguish the two classes.

Here's the core algorithm in plain terms:

```
Pick any minority sample X
Find its nearest minority neighbors
Pick one of them — call it X_neighbor
New point = X + random fraction × (X_neighbor − X)
```

You're placing a point somewhere on the line between two real fraud cases. Simple — but effective.

### The mistake that will quietly ruin your evaluation

I need to pause here because this is the error I see most often, even from experienced people.

**Do not apply SMOTE before splitting your data.**

If you oversample first and then split, some synthetic samples in your training set were generated using real samples that ended up in your test set. Your model has effectively "seen" your test data. Metrics will look better than they are — and you won't find out until production.

```python
from imblearn.over_sampling import SMOTE
from imblearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import StratifiedKFold, cross_val_score

# SMOTE lives inside the pipeline — it only ever sees training folds
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('smote', SMOTE(sampling_strategy=0.5, k_neighbors=5, random_state=42)),
    ('classifier', RandomForestClassifier(class_weight='balanced'))
])

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(pipeline, X, y, cv=cv, scoring='f1')
```

When SMOTE is inside the pipeline, it only touches training data. The test set stays completely untouched.

### Going further: hybrid methods

Once you've got SMOTE working, the next level is combining it with undersampling. Generate more minority samples *and* clean up the messy boundary between classes.

**SMOTE + Tomek Links** removes borderline majority samples sitting suspiciously close to the minority cluster — cleaning the decision boundary after you've oversampled.

**SMOTE + ENN** is more aggressive. After oversampling, it removes any sample — from *either* class — that gets misclassified by its nearest neighbors. Better for noisier datasets where the classes genuinely overlap.

```python
from imblearn.combine import SMOTETomek, SMOTEENN

sampler = SMOTETomek(random_state=42)  # Gentler cleanup
sampler = SMOTEENN(random_state=42)    # More aggressive, for noisy data
```

---

## Step 4: The lever almost everyone ignores

After all the resampling work, there's still one move left that costs nothing and often moves the needle more than expected: **changing the decision threshold**.

Every classification model outputs a probability. By default, if that probability is above 0.5, it predicts positive. Below 0.5, negative. That 0.5 is completely arbitrary. It's not derived from your data. It's not calibrated to your problem. It's just a default someone picked.

![Threshold Tuning](/blogs/assests/ml-img/imbalance/05_threshold_tuning.png)

The left chart shows what happens when you sweep through different thresholds. Precision climbs as you raise the bar. Recall falls. F1 peaks somewhere in the middle — and that peak almost never lands exactly at 0.5.

The right chart shows how the optimal threshold shifts by context. A cancer screening tool should sit around 0.15 — catch everything, even at the cost of false alarms. A spam filter can afford to be stricter.

```python
from sklearn.metrics import precision_recall_curve
import numpy as np

y_probs = model.predict_proba(X_val)[:, 1]
precisions, recalls, thresholds = precision_recall_curve(y_val, y_probs)

f1_scores = 2 * (precisions * recalls) / (precisions + recalls + 1e-8)
optimal_threshold = thresholds[np.argmax(f1_scores)]

# Use this instead of the default 0.5
y_pred = (y_probs >= optimal_threshold).astype(int)
```

I think of threshold tuning as the last layer of polish. You've trained a good model. You've resampled thoughtfully. Now you're just making sure the cut-off is in the right place for your specific situation. It takes ten minutes and often gives you 10–15 points of recall for free.

---

## Step 5: When you need the big guns — ensemble methods

If you've done everything above and you're still not where you need to be, it's time to upgrade the model itself.

**XGBoost** has a parameter built specifically for this problem:

$$\text{scale\_pos\_weight} = \frac{\text{count(negative class)}}{\text{count(positive class)}}$$

Set it, and XGBoost adjusts internally how much it penalizes minority class mistakes — similar to class weighting, but woven directly into the gradient boosting process.

```python
import xgboost as xgb

neg = (y_train == 0).sum()
pos = (y_train == 1).sum()

model = xgb.XGBClassifier(
    scale_pos_weight=neg / pos,  # The key parameter
    eval_metric='aucpr',         # Monitor with PR-AUC, not log loss
    max_depth=6,
    learning_rate=0.1,
    n_estimators=300,
    random_state=42
)
```

For really extreme imbalance — 1:500 or worse — `BalancedRandomForest` and `EasyEnsemble` are worth knowing. They balance each bootstrap sample before fitting each tree, which is a more structural fix than reweighting.

```python
from imblearn.ensemble import BalancedRandomForestClassifier, EasyEnsembleClassifier

brf = BalancedRandomForestClassifier(n_estimators=100, random_state=42)
eec = EasyEnsembleClassifier(n_estimators=10, random_state=42)
```

---

## Putting it all together — the framework I actually use

Here's the decision flow I've internalized after doing this enough times. Not the textbook version — the version I actually follow.

![Architecture Diagram](/blogs/assests/ml-img/imbalance/07_architecture_diagram.png)

Start simple. Measure honestly at every step. Escalate complexity only when the simpler solution genuinely isn't enough. Most problems don't need all five steps.

And when you're deciding which tool to reach for, this heatmap helps:

![Technique Heatmap](/blogs/assests/ml-img/imbalance/08_technique_heatmap.png)

Class weighting and threshold tuning win on simplicity and speed — they're always worth trying before adding complexity. SMOTE + ENN and EasyEnsemble are your heavy hitters for severe imbalance, but they come with real cost. Use them when you need them.

---

## What this actually looks like, step by step

Here's the exact progression I tracked on a fraud detection task with a 1:50 imbalance ratio:

![Experiment Progression](/blogs/assests/ml-img/imbalance/06_experiment_progression.png)

The starting point is painful — recall of 0.14. The model is missing 86% of fraud. By the time we've layered on class weighting, SMOTE, hybrid resampling, threshold tuning, and XGBoost, recall is at 0.85.

But notice something uncomfortable in that chart: **accuracy went down** throughout this process. From 98.2% to around 94.8%. That used to bother me. It doesn't anymore.

Those "lost" accuracy points represent the model stopping its habit of lazily classifying everything as legitimate. That downward number represents the model getting more *honest*, not worse. The trade is real — slightly more false alarms on the majority class — but in exchange, you're actually catching the thing you built the model to catch.

---

## What I've taken away from all of this

The model that showed 98.7% accuracy and caught zero fraud wasn't broken. It wasn't stupid. It was doing exactly what we'd incentivized it to do — minimize errors across the full dataset, weighted by how often each class appeared.

We gave it the wrong incentive. It optimized for the wrong thing. Perfectly.

The fix wasn't more data or a fancier architecture. It was mostly just being clearer about what we actually wanted the model to do — and then giving it the right tools and the right feedback signal to do it.

Class imbalance isn't primarily a technical problem. It's a clarity problem. Once you're clear on what "good" looks like, the technical pieces fall into place quickly.

---

## Quick reference — what to do and when

```
THE PLAYBOOK
──────────────────────────────────────────────────────

ALWAYS start here:
→ Drop accuracy as your primary metric
→ Use F1, PR-AUC, and the full confusion matrix
→ Use StratifiedKFold so every fold sees both classes

Then, in order of complexity:
1. class_weight='balanced'          ← free, always try first
2. SMOTE (inside your pipeline)     ← never on the full dataset
3. SMOTE + Tomek or SMOTE + ENN    ← when boundaries are messy
4. Threshold tuning                 ← do this before every deployment
5. XGBoost scale_pos_weight         ← when you need the full package

──────────────────────────────────────────────────────
The rarest thing in your dataset is usually
the most important thing in your problem.
Don't let the majority class drown it out.
```

---

## References

- Chawla et al. (2002). *SMOTE: Synthetic Minority Over-sampling Technique.* JAIR.
- He & Garcia (2009). *Learning from Imbalanced Data.* IEEE TKDE.
- [imbalanced-learn documentation](https://imbalanced-learn.org)
- [XGBoost docs — handling imbalanced data](https://xgboost.readthedocs.io)
- Davis & Goadrich (2006). *The relationship between Precision-Recall and ROC curves.* ICML.

---

