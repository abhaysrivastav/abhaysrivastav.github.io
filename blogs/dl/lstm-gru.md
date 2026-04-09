---
layout: topic
title: "LSTM and GRU: Solving the Memory Problem in Sequence Modelling"
permalink: /blogs/lstm-gru/
---

# LSTM and GRU: Solving the Memory Problem in Sequence Modelling

The previous post on RNNs ended with an uncomfortable truth: vanilla RNNs are fundamentally bad at remembering things from more than about 10–20 steps ago. The culprit was the vanishing gradient — gradients shrinking exponentially as they flow backward through time, starving early time steps of any learning signal.

The sentence "The trophy did not fit in the suitcase because **it** was too big" requires the model to remember the context of the word "trophy" to correctly resolve the pronoun "it." A standard RNN frequently fails this because by the time it processes "it," the information about "trophy" has been washed away through repeated matrix multiplications.

Two architectures were designed specifically to fix this: **Long Short-Term Memory (LSTM)** networks, introduced by Hochreiter and Schmidhuber in 1997, and **Gated Recurrent Units (GRU)**, introduced by Cho et al. in 2014. Both use learnable gating mechanisms to decide what to remember, what to forget, and what to output. Together they dominated sequence modelling tasks for nearly a decade before the Transformer took over — and they remain the right tool in many practical settings today.

---

## 1. The Core Idea: Gating

The key insight shared by both LSTM and GRU is the **gate** — a neural network layer with a sigmoid activation whose output lies between 0 and 1. When a gate outputs 0, information is blocked. When it outputs 1, information passes through unchanged.

```
gate = sigmoid(W · [input, previous_state] + b)
```

By learning to control these gates, the network learns to protect important information over many time steps. This creates a direct path for gradients to flow backward without being multiplied through long chains of tanh derivatives — which is exactly what prevents the vanishing gradient.

---

## 2. LSTM — Long Short-Term Memory

LSTM adds a second state, the **cell state** `cₜ`, running like a highway alongside the hidden state `hₜ`. Information can travel along the cell state across hundreds of time steps with only minor, additive modifications — unlike the vanilla RNN, where information is squeezed through a tanh at every step.

![LSTM Cell](https://upload.wikimedia.org/wikipedia/commons/9/93/LSTM_Cell.svg)
*The LSTM cell — a single time step. The cell state cₜ (top horizontal line) acts as long-term memory. Four gating mechanisms control how information enters, leaves, and persists in the cell. Source: [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:LSTM_Cell.svg), CC BY-SA 4.0.*

### The Four Gates

**Forget Gate** — decides what fraction of the previous cell state `cₜ₋₁` to keep:

```
fₜ = sigmoid(Wf · [hₜ₋₁, xₜ] + bf)
```

Output near 1 → keep the old memory. Output near 0 → erase it. For example, when processing a new sentence after a full stop, the forget gate should learn to reset subject-related memory.

**Input Gate** — decides what new information to write into the cell state:

```
iₜ = sigmoid(Wi · [hₜ₋₁, xₜ] + bi)       ← how much to write
g̃ₜ = tanh(Wg · [hₜ₋₁, xₜ] + bg)          ← what to write (candidate)
```

The candidate `g̃ₜ` (from the main/gate network) is scaled by `iₜ` before being added. If `iₜ` is near 0, almost nothing new gets written.

**Cell State Update** — combines forget and input:

```
cₜ = fₜ ⊙ cₜ₋₁  +  iₜ ⊙ g̃ₜ
```

`⊙` is element-wise multiplication. This is an additive update — gradients can flow back through the `+` without shrinking. This is what gives LSTM its ability to maintain long-range dependencies.

**Output Gate** — decides what portion of the cell state to expose as the hidden state:

```
oₜ = sigmoid(Wo · [hₜ₋₁, xₜ] + bo)
hₜ = oₜ ⊙ tanh(cₜ)
```

The hidden state `hₜ` is a filtered view of the cell state — only the parts relevant to the current prediction.

### Worked Example

Suppose you are predicting the next word in "The cat, which chased the mouse across the floor, finally caught **___**." The model needs to track two things simultaneously: the subject "cat" (relevant for subject-verb agreement) and the object "mouse" (what the cat catches).

- **Forget gate:** After processing "which chased the mouse," the gate partially resets mouse-related context since it is now the object, not the subject.
- **Input gate:** When the model sees "caught," it writes high-relevance information about the expected object.
- **Cell state:** Maintains "cat" in long-term memory across 9 tokens.
- **Output gate:** At the blank, exposes cat-related and caught-object information to predict "it" or "the mouse."

A vanilla RNN would almost certainly lose "cat" across those 9 tokens. LSTM's additive cell state keeps it alive.

---

## 3. GRU — Gated Recurrent Unit

GRU simplifies LSTM by merging the cell state and hidden state into a single `hₜ`, and reducing four gates to two. Introduced in 2014, it typically achieves comparable performance to LSTM while being faster to train and requiring fewer parameters.

![GRU Cell](https://upload.wikimedia.org/wikipedia/commons/3/37/Gated_Recurrent_Unit%2C_base_type.svg)
*The GRU cell — a single time step. Two gates (Update and Reset) replace LSTM's four. There is no separate cell state; the hidden state carries both short- and long-term memory. Source: [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Gated_Recurrent_Unit,_base_type.svg), CC BY-SA 4.0.*

### The Two Gates

**Update Gate** — decides how much of the previous hidden state to carry forward (analogous to LSTM's combined forget + input):

```
zₜ = sigmoid(Wz · [hₜ₋₁, xₜ] + bz)
```

When `zₜ` is close to 1, the network copies the previous hidden state almost unchanged — effectively remembering for as long as needed. When `zₜ` is close to 0, the network replaces the hidden state with new information.

**Reset Gate** — decides how much of the previous hidden state to use when computing the candidate new state:

```
rₜ = sigmoid(Wr · [hₜ₋₁, xₜ] + br)
```

A reset gate near 0 allows the model to ignore previous state entirely — useful when starting a fresh clause or sentence.

**Candidate hidden state:**

```
h̃ₜ = tanh(Wh · [rₜ ⊙ hₜ₋₁, xₜ] + bh)
```

**Final hidden state update:**

```
hₜ = (1 − zₜ) ⊙ hₜ₋₁  +  zₜ ⊙ h̃ₜ
```

This is the convex interpolation between old state and candidate. When `zₜ = 1`, the hidden state is entirely replaced. When `zₜ = 0`, it is entirely preserved.

### LSTM vs GRU: Key Structural Difference

The fundamental difference is that LSTM separates long-term storage (`cₜ`) from short-term output (`hₜ`), giving it more expressive capacity. GRU collapses these into one vector, trading capacity for speed. In practice, on most tasks the gap is small — GRU trains ~30% faster per epoch and is the better default when compute is a constraint.

---

## 4. Stacked and Bidirectional Variants

Both LSTM and GRU can be composed into more powerful architectures.

![Stacked and Bidirectional](/blogs/assests/dl-img/stacked_bidirectional.png)
*Left: A 3-layer stacked RNN — each layer receives the hidden states of the layer below as input, building progressively higher-level representations. Right: A Bidirectional RNN — two independent passes (left-to-right and right-to-left) whose outputs are concatenated at each time step, giving the model full context in both directions.*

### Stacked (Deep) RNNs

In a stacked RNN, the hidden state of layer `l` at time step `t` becomes the input to layer `l+1` at the same time step:

```
hₜ¹ = RNN₁(xₜ,  hₜ₋₁¹)      ← Layer 1: processes raw input
hₜ² = RNN₂(hₜ¹, hₜ₋₁²)      ← Layer 2: processes Layer 1's output
hₜ³ = RNN₃(hₜ², hₜ₋₁³)      ← Layer 3: processes Layer 2's output
```

Lower layers tend to capture local syntactic patterns; higher layers capture longer-range semantic relationships. Two to four stacked LSTM/GRU layers is a common configuration in production NLP systems.

### Bidirectional RNNs

The limitation of a standard RNN is that it only sees context from the left — words that have already been processed. For many tasks (e.g., named entity recognition, machine translation encoding), knowing what comes after a word is just as important as knowing what came before.

A Bidirectional RNN runs two independent RNN passes on the same sequence:
- **Forward pass**: processes the sequence left-to-right, producing `h→ₜ`
- **Backward pass**: processes the sequence right-to-left, producing `h←ₜ`

At each time step, the outputs are concatenated:

```
ŷₜ = f([h→ₜ ; h←ₜ])
```

This gives every output position access to the full left and right context simultaneously. BERT, for example, is built on bidirectional transformers for exactly this reason — the bidirectional idea predates the Transformer by many years.

Note: Bidirectional RNNs can only be used when the full input sequence is available at once (e.g., text classification, encoding for translation). They cannot be used for autoregressive generation, where future tokens are not yet known.

---

## 5. Comparison: RNN vs LSTM vs GRU

![RNN vs LSTM vs GRU Comparison](/blogs/assests/dl-img/lstm_gru_rnn_comparison.png)
*Side-by-side comparison of the three architectures across key dimensions. Green/amber cells indicate strength; grey indicates relative weakness.*

The practical rule of thumb:

- **Vanilla RNN** — useful only as a baseline or for very short sequences. Not worth using in production.
- **LSTM** — the safe default for long-sequence tasks (language modelling, machine translation, time series forecasting). More parameters, more expressive.
- **GRU** — preferred when training speed matters or when sequences are of moderate length. Approximately 75% of LSTM's parameters with similar accuracy on most benchmarks.

---

## 6. Using LSTM and GRU in Practice (Keras)

Switching from a vanilla RNN to LSTM or GRU is a single line change in Keras:

```python
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Embedding, LSTM, GRU, Dense, Bidirectional
from tensorflow.keras.datasets import imdb
from tensorflow.keras.preprocessing.sequence import pad_sequences

vocab_size = 10000
max_len    = 200

(x_train, y_train), (x_test, y_test) = imdb.load_data(num_words=vocab_size)
x_train = pad_sequences(x_train, maxlen=max_len)
x_test  = pad_sequences(x_test,  maxlen=max_len)

# ── Option A: Single LSTM ──────────────────────────────────────
model_lstm = Sequential([
    Embedding(vocab_size, 64, input_length=max_len),
    LSTM(64),
    Dense(1, activation='sigmoid')
])

# ── Option B: Single GRU ──────────────────────────────────────
model_gru = Sequential([
    Embedding(vocab_size, 64, input_length=max_len),
    GRU(64),
    Dense(1, activation='sigmoid')
])

# ── Option C: Stacked Bidirectional LSTM ─────────────────────
model_bi = Sequential([
    Embedding(vocab_size, 64, input_length=max_len),
    Bidirectional(LSTM(64, return_sequences=True)),  # 2 × 64 = 128-dim output
    Bidirectional(LSTM(32)),
    Dense(1, activation='sigmoid')
])

# Training (same for all three)
model_bi.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
model_bi.fit(x_train, y_train, epochs=5, batch_size=128, validation_split=0.2)

loss, acc = model_bi.evaluate(x_test, y_test, verbose=0)
print(f"Test accuracy: {acc:.3f}")
# Typical: LSTM ~0.87, GRU ~0.87, Bidirectional LSTM ~0.89
```

### Return sequences

When stacking RNN layers, all layers except the final one must use `return_sequences=True` — this outputs the hidden state at every time step (shape: `batch × time × units`) instead of just the final step:

```python
# Stacked LSTM — note return_sequences=True on all but the last
model_stacked = Sequential([
    Embedding(vocab_size, 64, input_length=max_len),
    LSTM(128, return_sequences=True),   # outputs (batch, 200, 128)
    LSTM(64,  return_sequences=True),   # outputs (batch, 200, 64)
    LSTM(32),                           # outputs (batch, 32)
    Dense(1, activation='sigmoid')
])
```

### Dropout in LSTM/GRU

Keras LSTM and GRU support two kinds of dropout:
- `dropout` — applied to inputs at each time step
- `recurrent_dropout` — applied to the recurrent connections (the Wₕ matrix)

```python
LSTM(64, dropout=0.2, recurrent_dropout=0.2)
```

Recurrent dropout is particularly important — without it, stacked LSTMs tend to overfit aggressively.

---

## 7. Strengths and Limitations

| Aspect | LSTM | GRU |
|--------|------|-----|
| **Long-range dependencies** | Excellent — cell state acts as a protected highway | Very good — update gate provides similar protection |
| **Parameters** | 4 × (input_dim + hidden_dim) × hidden_dim per layer | 3 × (input_dim + hidden_dim) × hidden_dim per layer |
| **Training speed** | Slower due to more parameters | ~30% faster than LSTM |
| **Expressiveness** | Higher — separate cell state and hidden state | Slightly lower — merged state |
| **Empirical performance** | Marginally better on complex tasks | Comparable on most benchmarks |
| **Bidirectional** | Supported | Supported |
| **Parallelism** | Still sequential across time steps | Still sequential across time steps |
| **Compared to Transformer** | Much slower to train on long sequences | Much slower to train on long sequences |

Both LSTM and GRU share RNN's fundamental limitation: sequential computation. They cannot be parallelised across time steps the way a Transformer can. For very long sequences or very large datasets, the Transformer's ability to process all tokens in parallel makes it significantly faster to train — which is why it replaced LSTM/GRU as the dominant architecture for NLP from 2018 onward.

That said, LSTM and GRU retain advantages in settings with short sequences, limited data, or real-time inference requirements.

---

## 8. Glossary

**Cell state (cₜ)** — LSTM-specific. A separate memory vector that runs alongside the hidden state. Its additive update rule allows information to persist across hundreds of time steps without vanishing.

**Forget gate (fₜ)** — An LSTM gate that decides what fraction of the previous cell state to erase. Output near 0 clears memory; near 1 preserves it.

**Input gate (iₜ)** — An LSTM gate that decides how much new information to write into the cell state.

**Output gate (oₜ)** — An LSTM gate that controls how much of the cell state is exposed as the hidden state output.

**Update gate (zₜ)** — A GRU gate that decides how much of the previous hidden state to retain versus replace with new information. Combines the roles of LSTM's forget and input gates.

**Reset gate (rₜ)** — A GRU gate that decides how much of the previous hidden state to use when computing the candidate new state. Allows the model to "reset" context for a new sub-sequence.

**Candidate hidden state (h̃ₜ)** — In GRU, the proposed new hidden state, computed from the current input and a reset-scaled version of the previous state.

**Stacked RNN** — Multiple RNN layers where each layer's hidden states serve as input to the next. Builds hierarchical representations of the sequence.

**Bidirectional RNN** — Two RNN passes over the same sequence (forward and backward), whose outputs are concatenated. Each position has access to both past and future context.

**Return sequences** — A parameter in Keras/PyTorch that controls whether an RNN layer outputs the hidden state at every time step (`True`) or only the final step (`False`). Required `True` when stacking RNN layers.

**Recurrent dropout** — Dropout applied to the recurrent weight matrix Wₕ, regularising the temporal connections specifically.

---

## 9. Further Reading

- Hochreiter, S. & Schmidhuber, J. (1997). *Long Short-Term Memory.* Neural Computation. [doi.org/10.1162/neco.1997.9.8.1735](https://doi.org/10.1162/neco.1997.9.8.1735) — the original LSTM paper, 27 years old and still worth reading.
- Cho, K. et al. (2014). *Learning Phrase Representations using RNN Encoder–Decoder.* [arxiv.org/abs/1406.1078](https://arxiv.org/abs/1406.1078) — introduced GRU and the Encoder-Decoder architecture.
- Chung, J. et al. (2014). *Empirical Evaluation of Gated Recurrent Neural Networks on Sequence Modeling.* [arxiv.org/abs/1412.3555](https://arxiv.org/abs/1412.3555) — the paper that benchmarked GRU vs LSTM and showed they perform comparably.
- Greff, K. et al. (2017). *LSTM: A Search Space Odyssey.* IEEE TNNLS. [arxiv.org/abs/1503.04069](https://arxiv.org/abs/1503.04069) — systematic ablation of all LSTM components. The forget gate turns out to be the most important one.
- Colah, C. *Understanding LSTM Networks.* [colah.github.io](https://colah.github.io/posts/2015-08-Understanding-LSTMs/) — the clearest visual explanation of LSTM gates available anywhere.

---

*Image attributions: LSTM cell diagram from [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:LSTM_Cell.svg), CC BY-SA 4.0. GRU cell diagram from [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Gated_Recurrent_Unit,_base_type.svg), CC BY-SA 4.0. Stacked/Bidirectional RNN diagram and comparison chart are original illustrations.*
