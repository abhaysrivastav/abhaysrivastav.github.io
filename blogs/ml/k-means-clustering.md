---
layout: topic
title: "K-Means Clustering: Algorithm, Intuition, and Practice"
permalink: /blogs/k-means-clustering/
date: 2025-01-01
categories: [Machine Learning, Unsupervised Learning]
tags: [k-means, clustering, unsupervised-learning, scikit-learn]
math: false
author: Abhay
---

# K-Means Clustering: Algorithm, Intuition, and Practice

---

## 1. Concept Definition

**K-Means Clustering** is an unsupervised machine learning algorithm that splits a dataset of *n* observations into *K* non-overlapping groups, where each data point belongs to exactly one cluster. The objective is straightforward — points inside a cluster should be as close to each other as possible, and as far from other clusters as they can be.

K-Means belongs to the family of **partitional clustering** methods. It carves the data into flat, non-hierarchical groups without needing any labeled examples. That makes it a practical first tool whenever you want to discover hidden structure in raw data — whether you are segmenting customers, compressing images, summarizing documents, or flagging anomalies.

---

## 2. Intuition

Before reaching for the math, let the idea land naturally.

Picture a map of cities. You drop *K* pins on it at random. Every city gets assigned to whichever pin is closest. Now you move each pin to the geographic center of all the cities assigned to it. Then reassign each city to the nearest pin again. Then move the pins again. You keep repeating — reassign, recenter, reassign, recenter — until the pins stop shifting. That whole process is K-Means.

Two simple forces drive it:

- Every data point wants to belong to the centroid closest to it.
- Every centroid wants to sit at the average position of its members.

These two forces take turns until the system settles. It is genuinely surprising how much you can get out of such a clean, back-and-forth mechanic.

> **Core idea:** K-Means finds a small set of representative points — called centroids — such that the total distance from every data point to its nearest representative is as small as possible.

The image below shows exactly this transformation. On the left is raw, unlabeled data with no visible structure. On the right, K-Means has carved it into three distinct groups and placed a centroid at the heart of each.

![K-Means Intuition: before and after clustering](/blogs/assests/ml-img/01_intuition.png)
*Figure 1 — Raw data has no visible structure. K-Means reveals three natural clusters and positions a centroid (★) at the center of each.*

---

## 3. The Algorithm

K-Means follows an iterative procedure known as **Lloyd's Algorithm**. The steps are few, and the logic behind each one is transparent.

**Input:** A dataset X of *n* points, and the desired number of clusters *K*.

**Output:** A cluster label for every point, plus the final position of each centroid.

---

**Step 1 — Initialize.** Pick *K* data points at random to serve as starting centroids (mu_1, mu_2, ..., mu_K).

**Step 2 — Assignment Step.** For every data point x_i, compute its distance to each centroid and assign it to the nearest one:

```
c(i) = argmin over k  of  ||x(i) - mu_k||^2
```

In words: assign each point to the cluster whose centroid is closest to it, measured by squared Euclidean distance.

**Step 3 — Update Step.** Recalculate each centroid as the mean of every point currently assigned to it:

```
mu_k = (1 / |C_k|) * sum of x(i)  for all i where c(i) = k
```

**Step 4 — Repeat.** Go back to Step 2. Keep alternating until the centroids stop moving — or a maximum iteration count is reached.

That is the complete algorithm. Two steps, iterated to convergence.

The three panels below trace this process on a small dataset. In the first panel the centroids are random and the assignments are rough. By the third panel the centroids have settled into stable positions and the cluster boundaries are clean.

![Algorithm iteration trace — 3 steps to convergence](/blogs/assests/ml-img/02_algorithm_steps.png)
*Figure 2 — Lloyd's Algorithm in action. Left: random initialization with first assignment. Middle: centroids updated and points reassigned. Right: converged solution.*

---

## 4. Optimization Objective

K-Means is not just a heuristic — it is minimizing a concrete cost called the **Within-Cluster Sum of Squares (WCSS)**, also known as **inertia** or the **distortion cost**:

```
J = sum over all clusters k of
      [ sum over all points x(i) in cluster k of  ||x(i) - mu_k||^2 ]
```

For every point, compute its squared distance to its assigned centroid. Add all those values together. That total is J, and K-Means is trying to make J as small as possible.

The diagram below visualizes what J measures. Every line connects a data point to its centroid. The total squared length of all those lines is exactly what the algorithm is driving down.

![WCSS visualization — squared distances to centroids](/blogs/assests/ml-img/03_wcss.png)
*Figure 3 — Each line represents the squared distance from a point to its assigned centroid. K-Means minimizes the total of all these squared distances (WCSS / inertia).*

### Why This Is Hard — And Why K-Means Works Anyway

Finding the globally optimal partition of *n* points into *K* clusters is **NP-hard**. The number of possible partitions grows explosively, so exhaustive search is off the table for any real dataset.

K-Means sidesteps this by splitting the problem into two simpler subproblems that it solves in turns:

- **Fix centroids, optimize assignments.** Given fixed centroids, the cheapest assignment for each point is simply the nearest one. Closed-form and exact.
- **Fix assignments, optimize centroids.** Given fixed assignments, the centroid minimizing squared distances to its members is their mean. Straightforward calculus.

Each step either decreases J or leaves it unchanged — it can never increase it. Since J is bounded below by zero, the algorithm must stop in a finite number of steps.

The catch is that it converges to a **local minimum**, not necessarily the global one. Which local minimum you land in depends on where you started. That is exactly why initialization matters so much, and why running the algorithm multiple times is standard practice.

### Monotonic Decrease Property

At every iteration *t*:

```
J(t+1)  <=  J(t)
```

The cost never goes up. Convergence is guaranteed.

---

## 5. Formulae and Worked Example

### Setup

Six two-dimensional points, K = 2:

| Point | x1  | x2  |
|-------|-----|-----|
| A     | 1.0 | 1.0 |
| B     | 1.5 | 2.0 |
| C     | 3.0 | 4.0 |
| D     | 5.0 | 7.0 |
| E     | 3.5 | 5.0 |
| F     | 4.5 | 5.0 |

**Initialization:** centroid_1 = A = (1, 1),  centroid_2 = D = (5, 7)

---

**Iteration 1 — Assignment Step**

Squared Euclidean distance from each point to both centroids:

| Point | dist^2 to C1 | dist^2 to C2 | Assigned to |
|-------|-------------|-------------|-------------|
| A     | 0           | 52.00       | Cluster 1   |
| B     | 1.25        | 37.25       | Cluster 1   |
| C     | 13.00       | 13.00       | Cluster 1 (tie) |
| D     | 52.00       | 0           | Cluster 2   |
| E     | 22.25       | 6.25        | Cluster 2   |
| F     | 42.25       | 5.00        | Cluster 2   |

Result: Cluster 1 = {A, B, C},  Cluster 2 = {D, E, F}

---

**Iteration 1 — Update Step**

```
centroid_1 = mean( A, B, C )
           = ( (1 + 1.5 + 3) / 3 ,  (1 + 2 + 4) / 3 )
           = ( 5.5/3 ,  7/3 )
           ≈ ( 1.83 ,  2.33 )

centroid_2 = mean( D, E, F )
           = ( (5 + 3.5 + 4.5) / 3 ,  (7 + 5 + 5) / 3 )
           = ( 13/3 ,  17/3 )
           ≈ ( 4.33 ,  5.67 )
```

---

**Iteration 2 — Assignment Step**

Recomputing distances with the updated centroids gives the same groupings — Cluster 1 = {A, B, C}, Cluster 2 = {D, E, F}. Centroids do not shift. **Converged.**

---

**Final WCSS:**

```
J = ||A - centroid_1||^2  +  ||B - centroid_1||^2  +  ||C - centroid_1||^2
  + ||D - centroid_2||^2  +  ||E - centroid_2||^2  +  ||F - centroid_2||^2
```

The figure below plots both iterations side by side. The left panel shows the initial assignment with centroids at A and D. The right panel shows the final state after centroids have shifted to their cluster means and the algorithm has converged.

![Worked example — scatter plots for both iterations](/blogs/assests/ml-img/04_worked_example.png)
*Figure 4 — Left: initial centroid positions (×) at A and D, with first cluster assignments. Right: centroids have moved to the cluster means and the solution has converged. Dashed lines connect each point to its assigned centroid.*

---

## 6. Initializing K-Means

Where you start matters a great deal. Two different initializations on the same dataset can converge to very different solutions — one tight and sensible, the other lopsided and misleading. This is one of the most underappreciated practical details of K-Means.

### Random Initialization (Naive)

Pick *K* data points uniformly at random and use them as starting centroids. Simple to implement, but unreliable. It can stack multiple centroids in the same dense region while leaving other parts of the data completely uncovered, leading to poor local minima that are hard to escape.

### K-Means++ (Recommended)

Proposed by Arthur and Vassilvitskii in 2007, **K-Means++** uses a probabilistic strategy that spreads the initial centroids across the data before the first iteration even begins.

**How it works:**

1. Pick the first centroid uniformly at random.
2. For each remaining centroid, compute D(x) — the squared distance from each data point to its nearest already-chosen centroid.
3. Sample the next centroid from the data with probability proportional to D(x)^2:

```
P( x_i ) = D(x_i)^2  /  sum over all j of  D(x_j)^2
```

4. Repeat until all *K* centroids are in place.

The effect is intuitive — points far from existing centroids are far more likely to be picked next, so the initial set covers the data broadly rather than clustering in one corner. K-Means++ carries a theoretical guarantee: the expected final cost is within O(log K) of the optimal. In practice, it converges faster and produces better clusters than random initialization on the vast majority of datasets.

scikit-learn uses K-Means++ by default (`init='k-means++'`), so you are already getting this improvement unless you have explicitly changed it.

### Multiple Random Restarts

Even K-Means++ does not guarantee the global optimum. The standard remedy is to run K-Means several times from different initializations and keep the run that achieved the lowest final J. The `n_init=10` default in scikit-learn handles this automatically — ten independent runs, best result kept.

The side-by-side below makes the stakes of initialization concrete. Same dataset, same K — but random initialization lands in a noticeably worse solution than K-Means++.

![Initialization comparison — random vs K-Means++](/blogs/assests/ml-img/05_init_comparison.png)
*Figure 5 — Random initialization (left) can produce uneven, overlapping clusters with high inertia. K-Means++ (right) spreads centroids intelligently and consistently finds a tighter, lower-cost solution.*

---

## 7. Choosing the Number of Clusters (K)

Picking K is the single most consequential decision you make when running K-Means. Too few clusters and distinct groups get merged into one. Too many and you are artificially splitting groups that belong together. There is no universally correct answer, but several well-established methods help narrow the search.

### The Elbow Method

Run K-Means for K = 1, 2, 3, ... up to some reasonable maximum. After each run, record the inertia. Plot inertia on the y-axis against K on the x-axis.

As K grows, inertia always falls — more clusters always reduces total within-cluster spread. But the rate of decrease slows down. The "elbow" in the curve, where adding one more cluster stops delivering much improvement, suggests a natural value of K.

In practice, the elbow is often fuzzy rather than sharp. Treat it as a starting hypothesis, not a definitive answer.

### The Silhouette Score

The silhouette score measures how comfortably each point sits inside its cluster compared to the next-nearest cluster. For a single point x_i:

```
a(i)  =  mean distance from x_i to all other points in its own cluster  [cohesion]
b(i)  =  mean distance from x_i to all points in the nearest other cluster  [separation]

s(i)  =  ( b(i) - a(i) )  /  max( a(i), b(i) )
```

The score ranges from -1 to +1:
- Near **+1** — the point sits well inside its cluster, far from others. Good.
- Near **0** — the point is on the boundary between two clusters.
- Near **-1** — the point probably belongs to a different cluster.

You compute this for every point and take the mean. The K that maximizes the mean silhouette score is generally the sounder choice. More objective than the elbow method, though it gets expensive for very large datasets.

### Gap Statistic

The gap statistic compares the within-cluster dispersion of your actual data against the expected dispersion of randomly generated data that has no real cluster structure. If K matches the true number of clusters, the gap between real and random dispersion is widest at that K.

It is the most statistically rigorous of the three approaches but requires generating and clustering multiple reference datasets, which adds computation time.

### Practical Guidance

No single method should be trusted in isolation. A sensible workflow: start with the elbow plot to get a rough range, verify with silhouette scores across that range, and apply domain knowledge to make the final call. A practitioner who knows their customer base has three distinct segments should not ignore that context just because the curve suggests four.

The plots below show both methods applied to the same dataset. The elbow at K = 4 and the silhouette peak at K = 4 agree — a reassuring sign when they align like this.

![Elbow method and silhouette analysis side by side](/blogs/assests/ml-img/06_elbow_silhouette.png)
*Figure 6 — Left: the elbow in the inertia curve points to K = 4. Right: the silhouette score peaks at K = 4, confirming the choice. When both methods agree, the decision is easy.*

| Method | Best For | Limitation |
|--------|----------|------------|
| Elbow Method | Quick visual sweep | Often ambiguous — elbow can be a gentle curve |
| Silhouette Score | Objective, per-point measure | Computationally expensive for large n |
| Gap Statistic | Statistically principled | Requires generating reference distributions |
| Domain Knowledge | Production decisions | Requires subject-matter expertise |

---

## 8. Strengths and Limitations

### Strengths

**Fast and scalable.** Time complexity is O(n · K · t · d), where t is iterations and d is features. For most datasets, convergence happens in well under a hundred iterations. Mini-Batch K-Means pushes scalability even further for very large data.

**Easy to understand and explain.** Centroids are concrete objects — they represent the average member of each cluster. Non-technical stakeholders can follow the logic without a statistics background.

**Guaranteed convergence.** The cost function decreases monotonically and is bounded below by zero, so the algorithm always terminates.

**Works well when the data fits the assumptions.** When clusters are roughly spherical, similarly sized, and well-separated, K-Means produces excellent results with minimal tuning.

### Limitations

**K must be specified in advance.** If your initial guess is wrong, the results can be meaningless. Use the elbow method, silhouette scores, and domain knowledge together.

**Converges to local minima only.** K-Means++ and multiple restarts reduce this risk considerably but do not eliminate it. On particularly tricky data, even ten restarts may not find the best solution.

**Assumes spherical, equally-sized clusters.** Euclidean distance is the heart of K-Means, and it implicitly assumes clusters are round and roughly the same size. Elongated, crescent-shaped, or density-varying clusters will be handled poorly. DBSCAN or spectral clustering handles those cases better.

**Outliers pull centroids off course.** A single extreme point can drag a centroid far from the rest of its cluster. Pre-processing with outlier removal, or switching to K-Medoids, addresses this.

**Feature scale matters.** A feature in the thousands dominates over one measured in single digits. Always standardize to zero mean and unit variance before clustering.

**Hard assignments only.** Every point gets one cluster, fully. For borderline points sitting between clusters, Gaussian Mixture Models — which assign partial probabilities — are a more honest representation.

**Struggles in high dimensions.** As dimensionality grows, Euclidean distances lose discrimination (the curse of dimensionality). Running PCA or UMAP to reduce dimensions before clustering usually makes a significant difference.

The figure below brings the biggest limitation to life. On compact, spherical data K-Means excels. On moon-shaped, non-convex data it fails badly — splitting each moon arbitrarily rather than following the true structure.

![K-Means strengths vs limitations — spherical vs moon-shaped data](/blogs/assests/ml-img/07_strengths_limits.png)
*Figure 7 — Left: K-Means handles compact, spherical clusters cleanly. Right: on non-convex (moon-shaped) data it cuts across the natural boundaries, producing meaningless clusters.*

---

## 9. scikit-learn Reference

### `sklearn.cluster.KMeans`

```python
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.datasets import make_blobs

X, _ = make_blobs(n_samples=500, centers=4, random_state=42)

# Always scale before K-Means — raw feature magnitudes affect distance
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('kmeans', KMeans(
        n_clusters=4,
        init='k-means++',   # smarter initialization (default)
        n_init=10,          # run 10 times, keep the best (default)
        max_iter=300,       # iteration cap per run (default)
        tol=1e-4,           # convergence tolerance
        random_state=42
    ))
])

pipeline.fit(X)
km = pipeline.named_steps['kmeans']

print(f"Cluster labels (first 10): {km.labels_[:10]}")
print(f"Centroids shape:           {km.cluster_centers_.shape}")
print(f"Inertia (WCSS):            {km.inertia_:.2f}")
print(f"Iterations to converge:    {km.n_iter_}")
```

### `sklearn.cluster.MiniBatchKMeans` — for large datasets

```python
from sklearn.cluster import MiniBatchKMeans

mbkm = MiniBatchKMeans(
    n_clusters=4,
    batch_size=256,    # samples used per iteration
    n_init=3,
    random_state=42
)
mbkm.fit(X)
```

### Key Parameters at a Glance

| Parameter | Default | What It Controls |
|-----------|---------|-----------------|
| `n_clusters` | 8 | Number of clusters K |
| `init` | `'k-means++'` | Initialization strategy — `'k-means++'` or `'random'` |
| `n_init` | 10 | Number of independent runs to attempt |
| `max_iter` | 300 | Iteration cap per run |
| `tol` | 1e-4 | Convergence threshold — relative shift in centroids |
| `algorithm` | `'lloyd'` | `'lloyd'` (standard) or `'elkan'` (faster in lower dimensions) |

---

## 10. Glossary

**Centroid:** The mean position of all data points in a cluster. Serves as that cluster's representative or prototype.

**WCSS (Within-Cluster Sum of Squares):** The quantity K-Means minimizes — total squared distance from every point to its assigned centroid. Lower means tighter, more compact clusters.

**Inertia:** scikit-learn's name for WCSS. Accessible via `km.inertia_` after fitting.

**Hard Assignment:** Each point is assigned to exactly one cluster with no ambiguity, as opposed to soft or probabilistic assignment where points can partially belong to multiple clusters.

**K-Means++:** An initialization method that places centroids far apart from each other by sampling with probability proportional to squared distance from already-placed centroids.

**Silhouette Score:** Measures how well each point fits its assigned cluster versus neighboring clusters. Ranges from -1 (likely misassigned) to +1 (well-clustered).

**Elbow Method:** A visual heuristic for picking K — plot inertia against K and look for where adding another cluster stops delivering meaningful improvement.

**Local Minimum:** A solution that cannot be improved by small perturbations, but may not be globally optimal. K-Means only guarantees convergence to a local minimum.

**Lloyd's Algorithm:** The standard iterative procedure at the heart of K-Means — alternate between assigning points to their nearest centroid and recomputing centroids as cluster means.

**Mini-Batch K-Means:** A faster variant that updates centroids using small random subsets of data each iteration, trading a small amount of accuracy for significant speed on large datasets.

**Curse of Dimensionality:** In high-dimensional spaces, Euclidean distances between points become increasingly similar, degrading the effectiveness of distance-based methods like K-Means.

---

## 11. Further Reading

**Foundational Papers**
- Lloyd, S. P. (1982). *Least squares quantization in PCM.* IEEE Transactions on Information Theory, 28(2), 129–137.
- Arthur, D., & Vassilvitskii, S. (2007). *k-means++: The advantages of careful seeding.* Proceedings of SODA 2007.

**Books**
- Bishop, C. M. (2006). *Pattern Recognition and Machine Learning.* Springer. Chapter 9: Mixture Models and EM.
- Géron, A. (2022). *Hands-On Machine Learning with Scikit-Learn, Keras, and TensorFlow* (3rd ed.). O'Reilly. Chapter 9: Unsupervised Learning.

**Official Documentation**
- [sklearn.cluster.KMeans](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html)
- [Clustering User Guide — scikit-learn](https://scikit-learn.org/stable/modules/clustering.html)
- [Clustering Algorithm Comparison — scikit-learn Examples](https://scikit-learn.org/stable/auto_examples/cluster/plot_cluster_comparison.html)

**Where to Go Next**
- Gaussian Mixture Models — the probabilistic big sibling of K-Means
- DBSCAN — handles arbitrary cluster shapes and noisy data naturally
- Hierarchical Agglomerative Clustering — builds a full tree of merges
- Dimensionality reduction before clustering: PCA, UMAP, t-SNE
