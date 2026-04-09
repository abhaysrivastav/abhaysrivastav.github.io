---
layout: topic
title: "Transformer Architecture: The Engine Behind Modern AI"
permalink: /blogs/transformer-architecture/
---

# Transformer Architecture: The Engine Behind Modern AI

If you have used ChatGPT, GitHub Copilot, or Google Translate in the last few years, you have been on the receiving end of the Transformer — arguably the most impactful neural network architecture of the last decade. Originally introduced in the 2017 paper *"Attention Is All You Need"* by Vaswani et al., the Transformer completely replaced recurrent networks for sequence modeling tasks. Understanding how it works is no longer optional for anyone serious about ML.

This post walks through the full architecture from the ground up — positional encoding, the three types of attention, the encoder, the decoder, and how they all connect.

---

## 1. The Problem Transformers Solve

Before Transformers, sequence-to-sequence tasks (like translation) were handled by RNNs, LSTMs, or GRUs. These models processed tokens one at a time, passing a hidden state from step to step. That design had two fundamental weaknesses:

**Vanishing gradients.** During backpropagation, gradients shrink as they travel backward through hundreds of time steps. The model effectively forgets long-range dependencies — the beginning of a paragraph has little influence on the end.

**No parallelism.** Because each step depends on the previous hidden state, you cannot process the sequence in parallel. Training on long sequences was painfully slow.

The Transformer solves both problems in one move: it throws away recurrence entirely and replaces it with **attention** — a mechanism that lets every token in a sequence directly attend to every other token, all at once, in parallel.

---

## 2. The Big Picture

Before diving into components, here is the full architecture at a glance.

![Transformer Architecture](/blogs/assests/dl-img/transformer_architecture.png)
*Full Transformer Architecture — Encoder (left) processes the input sequence; Decoder (right) generates the output sequence. The cyan arrow shows encoder output flowing as Keys and Values into the decoder's cross-attention layer.*

The architecture has two halves:

- **Encoder** — reads the entire input sequence and produces a rich, context-aware representation of it.
- **Decoder** — generates the output sequence token by token, using both what it has generated so far and the encoder's representation of the input.

Both halves are stacked N times (N = 6 in the original paper). Each layer refines the representation produced by the previous one.

---

## 3. Embeddings and Positional Encoding

### Word Embeddings

Every token in the input is first converted to a dense vector (embedding) of dimension `d_model` (512 in the original paper). These embeddings are learned during training and capture semantic relationships between words.

### The Positional Encoding Problem

Attention has no built-in sense of order. If you shuffle the words in a sentence, the attention mechanism produces identical outputs (just permuted). A Transformer needs to know that "dog bites man" is different from "man bites dog."

The fix is **positional encoding** — a vector added to each embedding that encodes the position of the token in the sequence. The original paper uses fixed sinusoidal functions:

```
PE(pos, 2i)   = sin(pos / 10000^(2i / d_model))
PE(pos, 2i+1) = cos(pos / 10000^(2i / d_model))
```

Where `pos` is the position and `i` is the dimension index. Even dimensions use sine, odd dimensions use cosine. Different frequencies at different dimensions mean every position gets a unique signature.

![Positional Encoding Visualization](/blogs/assests/dl-img/positional_encoding.png)
*Left: The full positional encoding matrix — each row is a position, each column a dimension. The sinusoidal wave patterns are clearly visible. Right: PE values for four selected dimensions across 50 positions — lower dimensions oscillate fast (fine-grained position), higher dimensions oscillate slowly (coarse-grained position).*

The elegant property of this design is that `PE(pos + k)` can be expressed as a linear function of `PE(pos)` — so the model can learn to attend by relative positions, not just absolute ones.

The final input to the first encoder layer is:

```
Input = WordEmbedding(token) + PositionalEncoding(position)
```

---

## 4. Scaled Dot-Product Attention

This is the heart of the Transformer. Everything else is scaffolding around this core operation.

### Queries, Keys, and Values

Attention works with three matrices derived from the input:

| Matrix | Symbol | Meaning |
|--------|--------|---------|
| Query  | Q      | "What am I looking for?" |
| Key    | K      | "What do I have to offer?" |
| Value  | V      | "What information do I carry?" |

Think of it like a search engine. The Query is your search term. Keys are the indexed documents. The dot product between Q and K produces a relevance score. The Value is the actual content you retrieve, weighted by those scores.

### The Formula

```
Attention(Q, K, V) = softmax(Q · Kᵀ / √dₖ) · V
```

Step by step:

1. **Q · Kᵀ** — Compute raw compatibility scores between every query and every key. Produces a matrix of shape `(seq_len × seq_len)`.
2. **÷ √dₖ** — Scale by the square root of the key dimension. Without this, dot products grow large in magnitude for high-dimensional vectors, pushing softmax into regions with near-zero gradients.
3. **softmax(...)** — Normalize across the key dimension so scores sum to 1. These are the attention weights.
4. **× V** — Weighted sum of values. Each output token is a blend of all values, weighted by how much it attended to each key.

![Multi-Head Attention](/blogs/assests/dl-img/multi_head_attention.png)
*Multi-Head Attention: Q, K, V are projected into h different subspaces. Each head performs its own Scaled Dot-Product Attention independently. Outputs are concatenated and linearly projected to produce the final result.*

### Worked Example

Suppose you have the sentence "The cat sat" with `dₖ = 4`. After linear projection, the Q and K for "cat" might look like:

```
Q_cat = [0.2, 0.8, 0.1, 0.5]
K_the = [0.1, 0.3, 0.2, 0.4]   →  Q·K = 0.48
K_cat = [0.8, 0.9, 0.1, 0.7]   →  Q·K = 1.24
K_sat = [0.3, 0.5, 0.6, 0.2]   →  Q·K = 0.62
```

After scaling by `√4 = 2`:  `[0.24, 0.62, 0.31]`

After softmax: `[0.18, 0.52, 0.30]` — "cat" attends most to itself (0.52) and somewhat to "sat" (0.30), which makes sense because "cat" is the subject of "sat."

The output for "cat" is then `0.18 × V_the + 0.52 × V_cat + 0.30 × V_sat` — a context-aware representation.

Here is what that looks like as an attention weight heatmap:

![Attention Heatmap](/blogs/assests/dl-img/scaled_dot_product_attention.png)
*Left: The Scaled Dot-Product Attention computation graph. Right: Attention weight heatmap for "The cat sat on mat" — brighter cells = stronger attention. "cat" attends most strongly to itself and to "sat."*

---

## 5. Multi-Head Attention

A single attention operation captures one type of relationship between tokens. But language is rich — you want to simultaneously capture subject-verb relationships, pronoun coreference, modifier-noun relationships, and so on.

**Multi-head attention** runs `h` attention operations in parallel, each with different learned projections of Q, K, and V:

```
head_i = Attention(Q · WᵢQ, K · WᵢK, V · WᵢV)

MultiHead(Q, K, V) = Concat(head₁, ..., headₕ) · WO
```

Each head learns to focus on different aspects of the input. One head might track syntactic structure while another tracks long-range semantic dependencies. The original paper uses `h = 8` heads with `dₖ = d_model / h = 64`.

The right panel of the figure above shows this clearly: h heads run in parallel, each produces its own attention output, then all are concatenated and projected through a final linear layer.

---

## 6. The Three Types of Attention

The Transformer uses attention in three distinct configurations, each serving a different purpose.

![Three Attention Types](/blogs/assests/dl-img/attention_types.png)
*The three configurations of attention in the Transformer: Encoder Self-Attention (all-to-all), Masked Decoder Self-Attention (causal — future tokens blocked), and Cross-Attention (Q from decoder, K/V from encoder output).*

### 6.1 Encoder Self-Attention

In the encoder, Q, K, and V all come from the same sequence (the input). Every token attends to every other token — no restrictions. This is how the model builds context. The word "bank" in "river bank" versus "bank account" will produce different representations because the surrounding tokens it attends to carry different signals.

### 6.2 Masked Decoder Self-Attention

In the decoder, self-attention is applied over the output sequence being generated, but with a critical constraint: token at position `t` can only attend to positions `0` through `t`. Future positions are set to `-∞` before softmax.

```
Mask matrix (upper triangle = -∞):
      The   cat   sat
The  [ 0    -∞    -∞  ]
cat  [ 0     0    -∞  ]
sat  [ 0     0     0  ]
```

This preserves the autoregressive property — the model cannot peek at tokens it has not generated yet.

### 6.3 Cross-Attention (Encoder-Decoder Attention)

This is the bridge between the two halves. Q comes from the decoder's current representation; K and V come from the encoder's output. Every decoder position can attend to every encoder position. In a translation task, when generating the Spanish word "encanta," the decoder's query attends most strongly to the English encoder tokens "love."

---

## 7. The Encoder

Each encoder layer consists of two sub-layers:

1. **Multi-Head Self-Attention**
2. **Position-wise Feed-Forward Network**

Each sub-layer is wrapped with a **residual connection** (`output = sublayer(x) + x`) and **Layer Normalization**. The full formula for one encoder layer:

```
x₁ = LayerNorm(x + MultiHeadAttention(x, x, x))
x₂ = LayerNorm(x₁ + FFN(x₁))
```

The **Feed-Forward Network** is a simple two-layer MLP applied identically to each position:

The feed-forward block is position-wise: the same two-layer MLP is applied independently to every token position.

```python
FFN(x) = max(0, x·W₁ + b₁)·W₂ + b₂
```

The inner dimension is typically 4× the model dimension (2048 for `d_model = 512`). The attention layers figure out *which* tokens to consider; the FFN processes *what* to do with that information.

This encoder layer is stacked N = 6 times. The final encoder output is a sequence of context vectors — one per input token — each carrying the full contextual meaning of that token.

---

## 8. The Decoder

The decoder has three sub-layers per layer:

1. **Masked Multi-Head Self-Attention** — attends to previously generated output tokens
2. **Multi-Head Cross-Attention** — attends to the encoder output
3. **Position-wise Feed-Forward Network**

Each wrapped with Add & Norm, just like the encoder. The decoder layer formula:

```
x₁ = LayerNorm(x + MaskedMultiHeadAttention(x, x, x))
x₂ = LayerNorm(x₁ + MultiHeadCrossAttention(x₁, enc_output, enc_output))
x₃ = LayerNorm(x₂ + FFN(x₂))
```

After N = 6 decoder layers, the output goes through a **Linear layer** (projects to vocabulary size) and **Softmax** (converts logits to token probabilities). The highest-probability token is selected and fed back as input for the next decoding step, until `[EOS]` is generated.

---

## 9. End-to-End: Information Flow

Here is a concrete walkthrough for translating "I love NLP" into Spanish:

**Encoder pass:**
1. Tokens "I", "love", "NLP" → embeddings + positional encodings.
2. Through 6 encoder layers, self-attention refines each token using full context. "love" knows it relates to both "I" and "NLP."
3. Encoder outputs 3 context vectors `(K, V)` — one per input token.

**Decoder pass:**
1. Decoder starts with `[SOS]`.
2. Masked self-attention → cross-attention to encoder `K, V` → FFN → softmax. First token: "Me."
3. Feed "Me" back in. Next: "encanta." Then "el NLP." Finally `[EOS]`.

The cross-attention layer is what makes this work — at every decoder step, it looks up which encoder tokens are most relevant for the next output word.

---

## 10. Strengths and Limitations

| Aspect | Transformer |
|--------|-------------|
| **Parallelism** | Full parallel computation — no sequential dependency |
| **Long-range dependencies** | Direct attention between any two tokens regardless of distance |
| **Scalability** | Scales extremely well with data and model size |
| **Context window** | Attention is O(n²) in sequence length — expensive for very long sequences |
| **Data hunger** | Requires large datasets to train from scratch |
| **Positional encoding** | Fixed sinusoidal encoding has limitations; later work (RoPE, ALiBi) improved this |
| **Interpretability** | Attention weights are partially interpretable but not a complete explanation |

---

## 11. Key Hyperparameters at a Glance

| Hyperparameter | Original Paper | Meaning |
|----------------|---------------|---------|
| `d_model` | 512 | Embedding and model dimension |
| `h` | 8 | Number of attention heads |
| `dₖ = dᵥ` | 64 | Dimension per head (`d_model / h`) |
| `d_ff` | 2048 | Inner dimension of FFN |
| `N` | 6 | Number of encoder/decoder layers |
| `dropout` | 0.1 | Applied after each sub-layer |

---

## 12. Using Transformers in Practice (HuggingFace)

```python
from transformers import pipeline

# Translation (encoder-decoder Transformer)
translator = pipeline("translation_en_to_fr", model="Helsinki-NLP/opus-mt-en-fr")
result = translator("I love machine learning.")
print(result[0]['translation_text'])
# Output: "J'adore l'apprentissage automatique."

# Text generation (decoder-only Transformer — GPT-style)
generator = pipeline("text-generation", model="gpt2")
output = generator("The Transformer architecture works by", max_length=50)
print(output[0]['generated_text'])

# Encoder output / sentence embeddings (BERT-style)
from transformers import AutoTokenizer, AutoModel
import torch

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
model = AutoModel.from_pretrained("bert-base-uncased")

inputs = tokenizer("The cat sat on the mat", return_tensors="pt")
with torch.no_grad():
    outputs = model(**inputs)

# shape: (1, seq_len, 768) — one 768-dim context vector per token
encoder_output = outputs.last_hidden_state
print(f"Encoder output shape: {encoder_output.shape}")
# torch.Size([1, 9, 768])
```

---

## 13. Glossary

**Attention weight** — A scalar (0 to 1) representing how much one token attends to another. Produced by softmax over Q·Kᵀ scores.

**Context vector** — Output of an attention layer for a given query. A weighted sum of Value vectors.

**Cross-attention** — Attention where Q comes from the decoder and K/V from the encoder. Bridges the two halves.

**d_model** — The dimensionality of all embeddings and hidden representations.

**dₖ** — Dimension per attention head. Equal to `d_model / h`.

**Feed-forward network (FFN)** — A two-layer MLP applied identically and independently to each token position.

**Layer normalization** — Normalization across the feature dimension of a single token, stabilizing training.

**Masked attention** — Attention where future positions are masked (-∞) before softmax, enforcing left-to-right generation.

**Multi-head attention** — h attention operations in parallel over different linear projections, concatenated at the output.

**Positional encoding** — A vector added to token embeddings to inject sequence order information.

**Q / K / V** — Query, Key, Value. Q is what you look for, K is what tokens offer, V is what they contain.

**Residual connection** — Adding sub-layer input to its output: `x + sublayer(x)`.

**Self-attention** — Q, K, V all from the same sequence — every token attends to every other.

**Teacher forcing** — Feeding ground-truth previous tokens to the decoder during training instead of its own predictions.

---

## 14. Further Reading

- Vaswani et al. (2017). *Attention Is All You Need.* [arxiv.org/abs/1706.03762](https://arxiv.org/abs/1706.03762)
- Alammar, J. *The Illustrated Transformer.* [jalammar.github.io](https://jalammar.github.io/illustrated-transformer/) — the best visual companion to this post.
- HuggingFace. *Transformers documentation.* [huggingface.co/docs/transformers](https://huggingface.co/docs/transformers)
- Huang et al. (2022). *Are Transformers Effective for Time Series Forecasting?* [arxiv.org/abs/2205.13504](https://arxiv.org/abs/2205.13504)

---

*Image attributions: Full architecture and Scaled Dot-Product / Multi-Head Attention diagrams from Vaswani et al. (2017) via [Wikimedia Commons](https://commons.wikimedia.org), [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). FFN module diagram from [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Transformer_architecture_-_FFN_module.png), CC BY-SA 4.0. Positional encoding heatmap, attention heatmap, and attention types diagram are original illustrations.*
