---
layout: topic
title: "Recurrent Neural Networks: Teaching Machines to Think in Sequences"
permalink: /blogs/rnn/
---

# Recurrent Neural Networks: Teaching Machines to Think in Sequences

Every time you get a smart autocomplete suggestion while typing, or your phone transcribes speech accurately, there is a model somewhere working through your words one step at a time — not just reading the current word but keeping track of everything that came before it. That ability to process sequences with memory is exactly what Recurrent Neural Networks (RNNs) were built for.

Standard feedforward networks — the kind used for image classification or tabular prediction — take a fixed-size input and produce a fixed-size output. They have no notion of "before" or "after." Give them the same input twice and they give you the same output twice. That works perfectly for tasks where inputs are independent. But language, audio, time series, and video are fundamentally different: the meaning of each element depends on what came before it. The word "bank" means something entirely different depending on whether the previous word was "river" or "investment."

RNNs were designed to handle exactly this challenge. This post covers what they are, how they work, how they are trained, and where they hit their limits.

---

## 1. What is an RNN?

A Recurrent Neural Network is a neural network with a loop — its output at each step is fed back as input at the next step. This loop creates a form of memory: the network doesn't just see the current input, it sees the current input combined with a summary of everything it has seen before.

![RNN Folded and Unfolded](https://upload.wikimedia.org/wikipedia/commons/b/b5/Recurrent_neural_network_unfold.svg)
*Left: A compact RNN with a recurrent loop. Right: The same network "unrolled" across time steps — at each step t, the hidden state hₜ is computed from the current input xₜ and the previous hidden state hₜ₋₁. Source: [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Recurrent_neural_network_unfold.svg), CC BY-SA 4.0.*

The key equations are simple:

```
hₜ = tanh(Wₓ · xₜ + Wₕ · hₜ₋₁ + b)
ŷₜ = Wᵧ · hₜ
```

Where:
- `xₜ` is the input at time step t
- `hₜ` is the hidden state (memory) at time step t
- `hₜ₋₁` is the hidden state from the previous step
- `Wₓ`, `Wₕ`, `Wᵧ` are weight matrices — **shared across all time steps**
- `ŷₜ` is the output at time step t

The critical insight is weight sharing. The same `Wₓ`, `Wₕ`, and `Wᵧ` are used at every time step. This means the network applies the same transformation regardless of position in the sequence — just like how you use the same grammatical knowledge to parse any sentence, not a different grammar book for every word.

### The Recurrent Neuron

In a standard feedforward neuron, there is only one weight connecting input to output: `Wₓ`. In a recurrent neuron, there are two: `Wₓ` for the current input and `Wₕ` for the previous hidden state. The hidden state `hₜ` is essentially the neuron's working memory — it accumulates a compressed summary of the entire input sequence up to that point.

In practice, each layer contains multiple recurrent neurons (say, 128 or 256 units), and the number of units is a hyperparameter you choose based on the complexity of the task.

---

## 2. Five RNN Architectures

Not all sequence tasks have the same shape. Sometimes you read a sequence and produce one answer. Sometimes a single input generates a whole sequence. RNNs are flexible enough to handle all of these.

![Five RNN Types](/blogs/assests/dl-img/rnn_types.png)
*The five RNN input-output configurations. Color indicates the architecture type. Real-world examples are shown for each.*

**One-to-One (1:1)** is just a standard feedforward network — single input, single output. Technically an RNN degenerated to no recurrence. Useful as a baseline.

**Many-to-One (N:1)** reads a full sequence and produces a single output at the end. The entire sequence is compressed into the final hidden state. Classic use case: sentiment analysis, where you read a full review and predict positive/negative.

**One-to-Many (1:N)** takes a single input and generates a sequence. The input seeds the first hidden state; thereafter the network generates tokens autoregressively. Classic use case: image captioning, where a single image embedding generates a sentence.

**Many-to-Many (N:N, synchronous)** reads and writes at every step simultaneously. Each input position produces an output position. Classic use case: video labelling, where each frame gets a label.

**Many-to-Many (N:M, asynchronous)** separates reading and writing into two phases — an encoder reads the input, and a decoder generates the output. The lengths N and M can differ. Classic use case: machine translation. This is the architecture that directly evolved into the Encoder-Decoder (Seq2Seq) model, covered in the next post.

---

## 3. Training RNNs: Backpropagation Through Time (BPTT)

Training an RNN requires computing gradients of the loss with respect to the shared weights `Wₓ`, `Wₕ`, and `Wᵧ`. The standard algorithm for this is **Backpropagation Through Time (BPTT)**.

The idea is to "unroll" the RNN across time steps — treating each step as a layer of a very deep feedforward network — and then apply standard backpropagation through that unrolled structure.

![BPTT Diagram](/blogs/assests/dl-img/bptt.png)
*BPTT unrolls the RNN across T time steps. The forward pass (blue arrows) computes hidden states and losses left-to-right. The backward pass (red dashed arrows) propagates gradients right-to-left through time. All weight updates for Wₓ, Wₕ, Wᵧ are summed across all time steps because the weights are shared.*

### Step-by-step

**Forward pass:** For each time step t from 0 to T:
1. Compute `hₜ = tanh(Wₓ·xₜ + Wₕ·hₜ₋₁ + b)`
2. Compute output `ŷₜ = Wᵧ·hₜ`
3. Compute loss `Lₜ` comparing `ŷₜ` to the true label
4. Total loss `L = L₀ + L₁ + ... + Lᵀ`

**Backward pass (BPTT):** Starting from `L`, compute the gradient of `L` with respect to each weight. Because `Wₕ` appears at every time step, its gradient must be summed over all steps:

```
∂L/∂Wₕ = Σₜ ∂Lₜ/∂Wₕ
```

For each term `∂Lₜ/∂Wₕ`, the chain rule must trace back through all hidden states from t back to 0:

```
∂Lₜ/∂Wₕ = Σₖ₌₀ᵗ (∂Lₜ/∂hₜ) · (∂hₜ/∂hₖ) · (∂hₖ/∂Wₕ)
```

The term `∂hₜ/∂hₖ` is a product of t−k Jacobians, each involving `Wₕ` and the tanh derivative. This is where the trouble starts.

### Truncated BPTT

For very long sequences (thousands of tokens), full BPTT is both memory-intensive (you must store all hidden states) and numerically unstable. In practice, **truncated BPTT** is used: the sequence is split into fixed-length chunks, and BPTT is applied within each chunk. The hidden state from the end of one chunk is carried forward as the initial state of the next — maintaining continuity — but gradients are not propagated between chunks.

---

## 4. The Vanishing and Exploding Gradient Problem

This is the fundamental weakness of standard RNNs, and understanding it is essential to understanding why LSTM and GRU were invented.

During BPTT, the gradient flowing backward through t steps involves a product of t matrices:

```
∂hₜ/∂h₀ = Wₕᵗ · (product of t tanh derivatives)
```

The tanh derivative lies between 0 and 1. And `Wₕ` can be greater or less than 1. So this product either:

- **Shrinks exponentially** (vanishing gradient) if the dominant eigenvalue of `Wₕ` is < 1. The gradient reaching early time steps becomes numerically zero. The network cannot learn that something 20 steps ago was important.
- **Grows exponentially** (exploding gradient) if the dominant eigenvalue is > 1. Gradients become enormous, weights receive catastrophic updates, and training diverges.

![Vanishing and Exploding Gradients](/blogs/assests/dl-img/vanishing_gradient.png)
*Left: With w < 1, the gradient signal decays exponentially as it flows backward through time — early time steps effectively receive no learning signal. Right: With w > 1, the gradient grows uncontrollably. Both are plotted on a log scale to show the magnitude difference clearly.*

### Practical consequences

The vanishing gradient means standard RNNs cannot reliably learn dependencies spanning more than ~10–20 time steps. Ask an RNN to translate a long sentence and it will have largely forgotten the beginning of the sentence by the time it processes the end.

### Fixes

**Gradient clipping** addresses the exploding side: if the gradient norm exceeds a threshold (e.g., 1.0 or 5.0), it is rescaled:

```python
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
```

The vanishing side is harder. Clipping does nothing when gradients are already zero. The real solutions are architectural — LSTM and GRU, which are covered in the next blog post.

---

## 5. Strengths and Limitations

| Aspect | RNN |
|--------|-----|
| **Sequential data** | Natural fit — processes one step at a time with memory |
| **Weight sharing** | Same weights across all time steps — parameter efficient |
| **Variable length** | Handles sequences of any length |
| **Long-range dependencies** | Struggles beyond ~10–20 time steps due to vanishing gradient |
| **Parallelism** | Sequential by design — cannot process steps in parallel, slow to train |
| **Interpretability** | Hidden state is a black-box compressed summary |
| **Training stability** | Prone to exploding/vanishing gradients without careful tuning |

---

## 6. Using RNNs in Practice (Keras)

You almost never implement an RNN from scratch. Here is a complete example using Keras for sequence classification (e.g., sentiment analysis):

```python
import numpy as np
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Embedding, SimpleRNN, Dense
from tensorflow.keras.datasets import imdb
from tensorflow.keras.preprocessing.sequence import pad_sequences

# Load IMDB dataset — 10,000 most frequent words, sequences padded to 200
vocab_size = 10000
max_len = 200
(x_train, y_train), (x_test, y_test) = imdb.load_data(num_words=vocab_size)
x_train = pad_sequences(x_train, maxlen=max_len)
x_test  = pad_sequences(x_test,  maxlen=max_len)

# Build a simple RNN model
model = Sequential([
    # Convert word indices to 32-dim dense vectors
    Embedding(vocab_size, 32, input_length=max_len),
    # Recurrent layer: 64 units, return only the final hidden state
    SimpleRNN(64),
    # Binary classifier
    Dense(1, activation='sigmoid')
])

model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
model.summary()

# Train
history = model.fit(x_train, y_train,
                    epochs=5, batch_size=128,
                    validation_split=0.2)

# Evaluate
loss, acc = model.evaluate(x_test, y_test, verbose=0)
print(f"Test accuracy: {acc:.3f}")
# Typical output: ~0.78 — good baseline, but LSTM will do better
```

**Why does this only get ~78% accuracy?** Because IMDB reviews are often 200+ words long, and the vanilla RNN forgets most of the earlier context by the time it reaches the end. Switch to `LSTM(64)` or `GRU(64)` and accuracy jumps to ~87–89% with the same architecture — which is exactly what the next blog post is about.

For a many-to-many (sequence tagging) task, use `return_sequences=True` to get an output at every step:

```python
from tensorflow.keras.layers import TimeDistributed

model = Sequential([
    Embedding(vocab_size, 32, input_length=max_len),
    SimpleRNN(64, return_sequences=True),     # output at every step
    TimeDistributed(Dense(num_tags, activation='softmax'))  # tag each token
])
```

---

## 7. Comparison: RNN vs Standard Feedforward Networks

| Property | Feedforward NN | RNN |
|----------|---------------|-----|
| Input type | Fixed-size vector | Variable-length sequence |
| Memory | None | Hidden state carries sequence history |
| Weight sharing | Per-layer | Across all time steps |
| Order sensitivity | No | Yes — order of inputs matters |
| Parallelism | Full | Sequential only |
| Typical tasks | Classification, regression | NLP, speech, time series, video |

---

## 8. Glossary

**Hidden state (hₜ)** — The RNN's internal memory at time step t. A vector that summarizes all inputs seen from step 0 through t.

**BPTT (Backpropagation Through Time)** — The training algorithm for RNNs. The network is unrolled across time and standard backpropagation is applied to the resulting graph. Gradients are summed across all time steps for each shared weight.

**Truncated BPTT** — A practical variant of BPTT where gradients are only propagated through a fixed number of steps back, reducing memory and improving numerical stability.

**Vanishing gradient** — When gradients shrink exponentially during BPTT, making it impossible to learn long-range dependencies. The dominant failure mode of standard RNNs.

**Exploding gradient** — When gradients grow exponentially, causing large and destabilizing weight updates. Addressed by gradient clipping.

**Gradient clipping** — Rescaling the gradient vector when its norm exceeds a threshold, preventing exploding gradients.

**Teacher forcing** — A training technique where the ground-truth output from step t is fed as input at step t+1, rather than the model's own prediction. Stabilizes training but can cause a mismatch at inference time.

**Weight sharing** — Using the same weight matrices `Wₓ`, `Wₕ`, `Wᵧ` at every time step, regardless of sequence length. This is what makes RNNs parameter-efficient.

**Return sequences** — In Keras/PyTorch, `return_sequences=True` means the RNN outputs a hidden state at every time step. `return_sequences=False` (default) outputs only the final hidden state.

---

## 9. Strengths and Limitations Summary

The RNN is an elegant idea that handles the core challenge of sequential data well — but it has a fundamental ceiling. Its sequential computation prevents GPU parallelism, its vanilla form cannot learn long-range dependencies, and training it requires careful gradient management.

These limitations are what drove the field to develop LSTM (1997) and GRU (2014) — and eventually the Transformer (2017), which abandoned recurrence altogether in favour of attention. Understanding the RNN's strengths and weaknesses is the foundation for understanding why each of those architectures exists.

---

## 10. Further Reading

- Hochreiter, S. & Schmidhuber, J. (1997). *Long Short-Term Memory.* Neural Computation. [doi:10.1162/neco.1997.9.8.1735](https://doi.org/10.1162/neco.1997.9.8.1735) — the paper that solved the vanishing gradient problem.
- Cho, K. et al. (2014). *Learning Phrase Representations using RNN Encoder–Decoder.* [arxiv.org/abs/1406.1078](https://arxiv.org/abs/1406.1078) — introduced the GRU and the Encoder-Decoder architecture.
- Karpathy, A. *The Unreasonable Effectiveness of Recurrent Neural Networks.* [karpathy.github.io](http://karpathy.github.io/2015/05/21/rnn-effectiveness/) — an intuitive blog post showing what vanilla RNNs can do on character-level language modelling.
- Sherstinsky, A. (2020). *Fundamentals of RNN and LSTM Network.* [arxiv.org/abs/1808.03314](https://arxiv.org/abs/1808.03314) — a thorough mathematical treatment of both architectures.

---

*Image attributions: RNN unrolled diagram from [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Recurrent_neural_network_unfold.svg), CC BY-SA 4.0. RNN types diagram, BPTT diagram, and gradient plots are original illustrations.*
