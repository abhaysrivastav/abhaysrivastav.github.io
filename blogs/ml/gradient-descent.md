---
layout: post
title: "Gradient Descent: The Engine Behind Machine Learning Optimization"
date: 2025-01-01
categories: [machine-learning, optimization]
tags: [gradient-descent, linear-regression, feature-scaling, learning-rate]
math: true
permalink: /blogs/gradient-descent/
image: /blogs/assests/ml-img/01_cost_surface.png
---

Gradient Descent is the workhorse of nearly every trained machine learning model. Whether you are fitting a simple line to data or training a deep neural network, the same core algorithm is taking small, calculated steps to minimize error. This post builds the concept from first principles, walks through the mathematics, and extends the idea to multiple features, feature scaling, and learning rate selection — with everything you need to apply it confidently.

---

## 1. Concept Definition

**Gradient Descent** is an iterative first-order optimization algorithm used to minimize a differentiable cost function J(θ) by updating parameters in the direction of the negative gradient.

The intuition: imagine standing on a hilly terrain in fog. You want to reach the valley (minimum). At every step, you feel the slope under your feet and take one step in the steepest downhill direction. That is gradient descent.

In machine learning, the "terrain" is the cost surface — a function of model parameters — and the "valley" is where prediction error is smallest.

---

## 2. Gradient Descent for Linear Regression

### The Model

Linear regression predicts a continuous output:

```
ŷ = h_θ(x) = θ₀ + θ₁x
```

where θ₀ is the intercept (bias) and θ₁ is the slope (weight).

### The Cost Function

We use the **Mean Squared Error (MSE)**:

```
J(θ₀, θ₁) = (1/2m) Σᵢ ( h_θ(x⁽ⁱ⁾) - y⁽ⁱ⁾ )²
```

The factor of 1/2 is a convenience — it cancels the 2 that appears when we differentiate.

### The Update Rule

For each parameter θⱼ, the update is:

```
θⱼ := θⱼ - α · (∂J / ∂θⱼ)
```

where α is the **learning rate** (step size).

For simple linear regression, the partial derivatives work out to:

```
∂J/∂θ₀ = (1/m) Σᵢ ( h_θ(x⁽ⁱ⁾) - y⁽ⁱ⁾ )

∂J/∂θ₁ = (1/m) Σᵢ ( h_θ(x⁽ⁱ⁾) - y⁽ⁱ⁾ ) · x⁽ⁱ⁾
```

**Critical:** Both parameters must be updated **simultaneously**:

```
temp0 = θ₀ - α · (∂J/∂θ₀)
temp1 = θ₁ - α · (∂J/∂θ₁)
θ₀ = temp0
θ₁ = temp1
```

Updating θ₀ first and then using the updated value to compute θ₁ is incorrect.

---

## 3. Gradient Descent Intuitions

### The Bowl-Shaped Cost Surface

For linear regression, J(θ₀, θ₁) is a **convex** function — it has exactly one global minimum. Its shape resembles a bowl (or a paraboloid). The gradient (slope) at any point on this surface tells us which direction increases cost. We move in the **opposite** direction to decrease it.

![Bowl-shaped cost surface showing gradient descent steps converging to the global minimum](/blogs/assests/ml-img/01_cost_surface.png)

### How Step Size Affects Convergence

| Learning Rate | Behaviour |
|---|---|
| Too small (e.g. 0.0001) | Converges, but takes very many iterations |
| Just right (e.g. 0.01) | Converges smoothly to minimum |
| Too large (e.g. 1.0) | Overshoots, may oscillate or diverge |

![Three panels showing cost vs iterations for: α too small (slow), α just right (smooth), α too large (oscillating)](/blogs/assests/ml-img/02_learning_rate_effect.png)

### At the Minimum

At the minimum, the gradient equals zero. Therefore, the update becomes:

```
θⱼ := θⱼ - α · 0 = θⱼ
```

The algorithm naturally stops changing parameters once it reaches the minimum — no explicit stopping condition is needed for a perfectly converged run. In practice, we stop when the change in J per iteration falls below a small threshold ε.

### Gradient Descent Variants

| Variant | Batch Size | Pros | Cons |
|---|---|---|---|
| **Batch GD** | All m samples | Stable, guaranteed descent | Slow for large datasets |
| **Stochastic GD (SGD)** | 1 sample | Fast updates | Noisy, may not converge exactly |
| **Mini-Batch GD** | k samples (e.g. 32, 64) | Balance of speed and stability | Requires tuning k |

Mini-batch gradient descent is the standard in modern deep learning.

---

## 4. Gradient Descent for Multiple Linear Regression

### The Model

When there are n features:

```
ŷ = θ₀ + θ₁x₁ + θ₂x₂ + ... + θₙxₙ
```

In vectorised form (using the convention x₀ = 1):

```
ŷ = θᵀx
```

### Cost Function

Identical in structure to the univariate case:

```
J(θ) = (1/2m) Σᵢ ( θᵀx⁽ⁱ⁾ - y⁽ⁱ⁾ )²
```

### Update Rule (Vectorised)

For each parameter θⱼ where j = 0, 1, ..., n:

```
θⱼ := θⱼ - α · (1/m) Σᵢ ( h_θ(x⁽ⁱ⁾) - y⁽ⁱ⁾ ) · xⱼ⁽ⁱ⁾
```

In compact matrix notation:

```
θ := θ - (α/m) · Xᵀ(Xθ - y)
```

where **X** is the m × (n+1) design matrix, **θ** is the (n+1) × 1 parameter vector, and **y** is the m × 1 target vector.

The vectorised form is far more computationally efficient — it replaces Python loops with optimised NumPy operations.

---

## 5. Feature Scaling

### Why It Matters

When features have very different ranges, the cost surface becomes elongated — an ellipse rather than a circle. Gradient descent on an elongated surface oscillates back and forth across the narrow direction, makes very slow progress along the broad direction, and requires a much smaller learning rate to avoid divergence.

![Contour plots showing elongated vs circular cost surface, and how gradient descent paths differ with and without feature scaling](/blogs/assests/ml-img/03_feature_scaling_contour.png)

The remedy is to bring all features into a comparable range before training.

### Method 1 — Max Normalization (Simple Rescaling)

```
x_scaled = x / max(x)
```

- Range after scaling: [0, 1] (assuming non-negative features)
- Simple but sensitive to outliers (the max may be an outlier)

**Worked example:**

| House Size (sq ft) | Raw | max = 2000 | Scaled |
|---|---|---|---|
| Sample 1 | 800 | 2000 | 0.40 |
| Sample 2 | 1200 | 2000 | 0.60 |
| Sample 3 | 2000 | 2000 | 1.00 |

### Method 2 — Mean Normalization

```
x_scaled = (x - μ) / (max(x) - min(x))
```

- Range after scaling: approximately [-1, 1]
- Centers the data around zero

**Worked example** (House Size: min=600, max=2000, mean=1200):

| Sample | Raw | Scaled |
|---|---|---|
| 1 | 800 | (800−1200)/(2000−600) = −0.286 |
| 2 | 1200 | (1200−1200)/(2000−600) = 0.000 |
| 3 | 2000 | (2000−1200)/(2000−600) = +0.571 |

### Method 3 — Z-Score Normalization (Standardization)

```
x_scaled = (x - μ) / σ
```

where μ is the mean and σ is the standard deviation of the feature.

- Range after scaling: centered at 0, unit variance; typically [-3, 3]
- Handles outliers better than max normalization
- Required for algorithms that assume Gaussian-distributed features (e.g., SVM, PCA)

**Worked example** (House Size: mean=1200, std=509.9):

| Sample | Raw | Scaled |
|---|---|---|
| 1 | 800 | (800−1200)/509.9 = −0.785 |
| 2 | 1200 | (1200−1200)/509.9 = 0.000 |
| 3 | 2000 | (2000−1200)/509.9 = +1.569 |

### Comparison Table

| Method | Formula | Output Range | Best Used When |
|---|---|---|---|
| Max Normalization | x / max(x) | [0, 1] | No outliers, non-negative features |
| Mean Normalization | (x − μ) / (max − min) | approx [-1, 1] | General purpose, no severe outliers |
| Z-Score Normalization | (x − μ) / σ | approx [-3, 3] | Outliers present, downstream algorithms assume Gaussian |

> **Rule of thumb:** aim for features in the range [-1, 1] or [-3, 3]. Any feature with a range like [0, 1000] or [-0.001, 0.001] should be rescaled.

---

## 6. How to Verify Gradient Descent is Working Correctly

### The Learning Curve

Plot J(θ) (cost) on the y-axis against the number of iterations on the x-axis. A correctly converging run should produce a **monotonically decreasing** curve that flattens out:

![Learning curve showing cost monotonically decreasing and flattening at convergence](/blogs/assests/ml-img/04_learning_curve_correct.png)

### Warning Signs

| Pattern | Diagnosis |
|---|---|
| Cost increases then decreases | Learning rate too large — overshooting |
| Cost oscillates wildly | Learning rate far too large |
| Cost decreases but extremely slowly | Learning rate too small or features not scaled |
| Cost increases monotonically | Bug in implementation (e.g., +α instead of −α) |

![Three warning learning curve patterns: diverging, oscillating, and too slow](/blogs/assests/ml-img/05_warning_curves.png)

### Convergence Test

Declare convergence when the decrease in cost per iteration falls below a threshold:

```
J(θ⁽ᵗ⁾) - J(θ⁽ᵗ⁺¹⁾) < ε
```

A typical value is ε = 10⁻³. The exact value depends on the problem scale. Monitoring the learning curve visually is often more informative than a fixed threshold.

### Gradient Check (for custom implementations)

Verify analytical gradients using the **numerical gradient**:

```
∂J/∂θⱼ ≈ [ J(θⱼ + ε) - J(θⱼ - ε) ] / (2ε)
```

Use ε = 10⁻⁴. If analytical and numerical gradients differ by more than ~10⁻⁵, there is a bug in the gradient computation. Disable this check during training — it is only for debugging.

---

## 7. How to Choose the Learning Rate

### Systematic Search

Try values on a logarithmic scale:

```
α ∈ { 0.001, 0.003, 0.01, 0.03, 0.1, 0.3, 1.0 }
```

For each α, run a short training run (e.g., 100 iterations) and plot the learning curve. Choose the largest α that produces a smoothly decreasing curve.

![Learning rate search plot showing multiple α values on the same cost vs iterations chart](/blogs/assests/ml-img/06_lr_search.png)

### Adaptive Learning Rate Methods

In practice, manually tuning a fixed α is often replaced by **adaptive optimisers** that adjust the learning rate automatically during training:

| Optimiser | Key Idea | Common Use Case |
|---|---|---|
| **AdaGrad** | Per-parameter learning rates, decays for frequent features | Sparse data, NLP |
| **RMSProp** | Exponentially weighted per-parameter rates | Non-stationary problems, RNNs |
| **Adam** | Combines momentum + RMSProp; the modern default | Most deep learning tasks |
| **SGD + Momentum** | Adds velocity to updates | Computer vision, fine-tuned models |

For linear regression and standard ML problems, a well-chosen fixed α with feature scaling is sufficient.

### Learning Rate Schedules

Rather than fixing α throughout training, a schedule reduces it over time:

```
αₜ = α₀ / (1 + decay_rate × t)
```

This allows larger steps early (fast progress) and smaller steps later (fine-grained convergence).

---

## 8. Strengths and Limitations

### Strengths

- **Scalable**: Mini-batch variant handles datasets too large to fit in memory.
- **General**: Works on any differentiable cost function — far beyond linear regression.
- **Foundational**: Understanding GD is prerequisite knowledge for neural networks, logistic regression, and virtually all parameterised models.
- **Controllable**: Learning rate, batch size, and stopping criteria offer fine-grained control.

### Limitations

- **Hyperparameter sensitivity**: Learning rate requires tuning. Too large diverges; too small is impractical.
- **Feature scaling required**: Without it, convergence is slow and may require impractically small α.
- **Local minima**: For non-convex problems (deep networks), GD may converge to a local minimum rather than the global one. For linear regression, the cost surface is convex, so this is not a concern.
- **Saddle points**: In high dimensions, saddle points — not local minima — are the more common problem. Adaptive optimisers handle these better.
- **Not always the fastest**: For small datasets with n features, the **Normal Equation** (XᵀX)⁻¹Xᵀy gives the exact solution in one step without iteration. However, it scales as O(n³) and is impractical for n > 10,000.

| | Gradient Descent | Normal Equation |
|---|---|---|
| Needs α | Yes | No |
| Needs feature scaling | Yes | No |
| Iterations required | Many | None (one-shot) |
| Works for large n | Yes | No (O(n³)) |
| Generalises to other algorithms | Yes | No |

---

## 9. scikit-learn Reference

scikit-learn's `LinearRegression` uses the Normal Equation by default (via `np.linalg.lstsq`). For gradient-descent-based linear regression, use `SGDRegressor`:

```python
from sklearn.linear_model import SGDRegressor
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.datasets import make_regression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error
import numpy as np

# Generate data
X, y = make_regression(n_samples=1000, n_features=5, noise=20, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Pipeline: feature scaling → SGD linear regression
model = Pipeline([
    ("scaler", StandardScaler()),                 # Z-score normalisation
    ("sgd", SGDRegressor(
        loss="squared_error",                     # MSE cost function
        eta0=0.01,                                # initial learning rate
        learning_rate="invscaling",               # schedule: η = η₀ / t^0.25
        max_iter=1000,                            # maximum epochs
        tol=1e-4,                                 # convergence threshold
        random_state=42,
        verbose=0
    ))
])

model.fit(X_train, y_train)
y_pred = model.predict(X_test)

print(f"Test MSE:  {mean_squared_error(y_test, y_pred):.4f}")
print(f"Test RMSE: {np.sqrt(mean_squared_error(y_test, y_pred)):.4f}")
```

### Key `SGDRegressor` Parameters

| Parameter | Description | Typical Value |
|---|---|---|
| `eta0` | Initial learning rate | 0.01 |
| `learning_rate` | Schedule (`"constant"`, `"invscaling"`, `"adaptive"`) | `"invscaling"` |
| `max_iter` | Maximum number of passes over training data | 1000 |
| `tol` | Stopping criterion (change in loss) | 1e-4 |
| `alpha` | L2 regularisation strength | 1e-4 |
| `shuffle` | Shuffle data each epoch (essential for SGD) | True |

---

## 10. Glossary

| Term | Definition |
|---|---|
| **Gradient** | Vector of partial derivatives of the cost function with respect to each parameter |
| **Learning Rate (α)** | Scalar controlling the step size of each parameter update |
| **Cost Function J(θ)** | Measure of prediction error; gradient descent minimises this |
| **Convergence** | When the cost function no longer decreases significantly between iterations |
| **Epoch** | One complete pass over the entire training dataset |
| **Batch Gradient Descent** | Uses all training examples to compute the gradient at each step |
| **Stochastic Gradient Descent** | Uses a single training example per gradient update |
| **Mini-Batch GD** | Uses a small subset (batch) of training examples per gradient update |
| **Feature Scaling** | Transforming features to a common range to improve gradient descent convergence |
| **Z-Score Normalisation** | Scaling feature to zero mean and unit variance |
| **Normal Equation** | Closed-form analytical solution to linear regression; no iteration required |
| **Saddle Point** | A point where the gradient is zero but is neither a minimum nor a maximum |
| **Design Matrix X** | Matrix where each row is a training example, each column is a feature (plus bias) |
| **Convex Function** | A function with a single global minimum; gradient descent is guaranteed to find it |

---

## 11. Further Reading

- **Andrew Ng — Machine Learning Specialisation (Coursera)**: The canonical course treatment of gradient descent for linear regression. Highly recommended for visual intuition.
- **Bishop, C. (2006). _Pattern Recognition and Machine Learning_. Springer.** — Chapter 3 covers linear regression from a probabilistic perspective.
- **Goodfellow, Bengio & Courville (2016). _Deep Learning_. MIT Press.** — Chapter 8 covers optimisation algorithms for deep learning, including momentum, AdaGrad, and Adam.
- **scikit-learn Documentation — `SGDRegressor`**: [https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.SGDRegressor.html](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.SGDRegressor.html)
- **scikit-learn User Guide — Stochastic Gradient Descent**: [https://scikit-learn.org/stable/modules/sgd.html](https://scikit-learn.org/stable/modules/sgd.html)
- **Ruder, S. (2016). An overview of gradient descent optimisation algorithms.** arXiv:1609.04747 — A thorough survey of all major GD variants and adaptive methods.
