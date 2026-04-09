---
layout: topic
title: "Transformer Architecture: The Engine Behind Modern AI"
permalink: /blogs/transformer-architecture/
---

# Transformer Architecture: The Engine Behind Modern AI

If you have used ChatGPT, GitHub Copilot, or Google Translate in the last few years, you have been on the receiving end of the Transformer — arguably the most impactful neural network architecture of the last decade. Originally introduced in the 2017 paper *"Attention Is All You Need"* by Vaswani et al., the Transformer completely replaced recurrent networks for sequence modeling tasks. 

This post walks through the full architecture from the ground up — positional encoding, the three types of attention, the encoder, the decoder, and how they connect.

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

Both the encoder and decoder are stacked N times (N = 6 in the original paper). Each layer refines the representation produced by the previous one.

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

![Positional Encoding](/blogs/assests/dl-img/positional_encoding.png)
*Left: The full positional encoding matrix — each row is a position, each column a dimension. The wave-like patterns are clearly visible. Right: PE values for selected dimensions across positions — lower dimensions oscillate fast (fine-grained position), higher dimensions oscillate slowly (coarse-grained position).*

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

1. **Q · Kᵀ** — Compute raw compatibility scores between every query and every key. This gives a matrix of shape `(seq_len × seq_len)`.
2. **÷ √dₖ** — Scale by the square root of the key dimension. Without this, dot products grow large in magnitude for high-dimensional vectors, pushing softmax into regions with near-zero gradients.
3. **softmax(...)** — Normalize across the key dimension so scores sum to 1. These are the attention weights.
4. **× V** — Weighted sum of values. Each output token is a blend of all values, weighted by how much it attended to each key.

![Scaled Dot-Product Attention](/blogs/assests/dl-img/scaled_dot_product_attention.png)
*Left: The computation graph of Scaled Dot-Product Attention. Right: An example attention heatmap for "The cat sat on mat" — brighter cells indicate stronger attention between token pairs.*

### Worked Example

Suppose you have the sentence "The cat sat" with `dₖ = 4`. After linear projection, the Q and K for the word "cat" might look like:

```
Q_cat = [0.2, 0.8, 0.1, 0.5]
K_the = [0.1, 0.3, 0.2, 0.4]   →  Q·K = 0.02+0.24+0.02+0.20 = 0.48
K_cat = [0.8, 0.9, 0.1, 0.7]   →  Q·K = 0.16+0.72+0.01+0.35 = 1.24
K_sat = [0.3, 0.5, 0.6, 0.2]   →  Q·K = 0.06+0.40+0.06+0.10 = 0.62
```

After scaling by `√4 = 2`:  `[0.24, 0.62, 0.31]`

After softmax: `[0.18, 0.52, 0.30]` — "cat" attends most to itself (0.52) and somewhat to "sat" (0.30), which is sensible because "cat" is the subject of "sat."

The output for "cat" is then `0.18 × V_the + 0.52 × V_cat + 0.30 × V_sat` — a context-aware representation of "cat."

---

## 5. Multi-Head Attention

A single attention operation captures one type of relationship between tokens. But language is rich — you want to simultaneously capture subject-verb relationships, pronoun coreference, modifier-noun relationships, and so on.

**Multi-head attention** runs `h` attention operations in parallel, each with different learned projections of Q, K, and V:

```
head_i = Attention(Q · Wᵢᴼ, K · Wᵢᴷ, V · WᵢᵛV)

MultiHead(Q, K, V) = Concat(head₁, ..., headₕ) · Wᴼ
```

Where `Wᵢᴼ`, `Wᵢᴷ`, `WᵢᵛV` are learned projection matrices per head, and `Wᴼ` is a final learned output projection.

Each head learns to focus on different aspects of the input. One head might track syntactic structure while another tracks long-range semantic dependencies. The original paper uses `h = 8` heads with `dₖ = d_model / h = 64`.

![Multi-Head Attention](/blogs/assests/dl-img/multi_head_attention.png)
*Multi-Head Attention: Q, K, V are projected into h different subspaces. Each head performs its own Scaled Dot-Product Attention independently. Outputs are concatenated and linearly projected to produce the final result.*

---

## 6. The Three Types of Attention

The Transformer uses attention in three distinct configurations, each serving a different purpose.

![Attention Types](/blogs/assests/dl-img/attention_types.png)
*The three configurations of attention: Self-Attention (encoder), Masked Self-Attention (decoder), and Cross-Attention (encoder-decoder bridge).*

### 6.1 Encoder Self-Attention

In the encoder, Q, K, and V all come from the same sequence (the input). Every token attends to every other token — no restrictions.

This is how the model builds context. The word "bank" in "river bank" versus "bank account" will produce different representations because the surrounding tokens it attends to carry different signals. After self-attention, each token's representation is enriched by the entire surrounding context.

### 6.2 Masked Decoder Self-Attention

In the decoder, we also apply self-attention over the output sequence being generated. But there is a critical constraint: during training, the model receives the full target sequence at once (teacher forcing). If it could attend to future tokens, it would trivially learn to copy — no real learning happens.

The solution is **masking**. Before the softmax step, attention scores for future positions are set to `-∞`, which become 0 after softmax. Token at position `t` can only attend to positions `0` through `t`.

```
                     Mask matrix (upper triangle = -∞):
      The   cat   sat
The  [ 0    -∞    -∞  ]
cat  [ 0     0    -∞  ]
sat  [ 0     0     0  ]
```

This preserves the autoregressive property at training time — the model can only use information from the past to predict the next token.

### 6.3 Cross-Attention (Encoder-Decoder Attention)

This is the bridge between the two halves of the architecture. In cross-attention:

- **Q** comes from the decoder's current representation
- **K and V** come from the encoder's output

Every decoder position can attend to every encoder position. This is how the decoder "reads" the encoded input while generating the output. In a translation task, when generating the Spanish word "encanta," the decoder's query attends most strongly to the English encoder tokens "love" — exactly the alignment you would want.

---

## 7. The Encoder

Now that we have the attention building blocks, the encoder's structure is straightforward.

Each encoder layer consists of two sub-layers:

1. **Multi-Head Self-Attention**
2. **Position-wise Feed-Forward Network**

Around each sub-layer, there are two additional operations:

**Residual connection (Add):** The input to each sub-layer is added to its output. Formally: `output = sublayer(x) + x`. This prevents vanishing gradients and makes it easier for the model to learn identity functions in early training.

**Layer Normalization (Norm):** Normalizes across the feature dimension of each token independently. This stabilizes training.

The full formula for one encoder layer:

```
x₁ = LayerNorm(x + MultiHeadAttention(x, x, x))
x₂ = LayerNorm(x₁ + FFN(x₁))
```

The **Feed-Forward Network** is a simple two-layer MLP applied identically to each position:

```python
FFN(x) = max(0, x·W₁ + b₁)·W₂ + b₂
```

The inner dimension is typically 4× the model dimension (2048 for `d_model = 512`). This is where the model stores factual and associative knowledge — the attention layers figure out *which* tokens to consider, the FFN processes *what* to do with that information.

This encoder layer is stacked N = 6 times. The final encoder output is a sequence of context vectors, one per input token, each shaped `(1, d_model)`. These context vectors encode not just the individual tokens but their full relational context.

---

## 8. The Decoder

The decoder has three sub-layers per layer (compared to two in the encoder):

1. **Masked Multi-Head Self-Attention** — attends to previously generated output tokens
2. **Multi-Head Cross-Attention** — attends to the encoder output
3. **Position-wise Feed-Forward Network**

Each sub-layer is again wrapped with Add & Norm. The decoder layer formula:

```
x₁ = LayerNorm(x + MaskedMultiHeadAttention(x, x, x))
x₂ = LayerNorm(x₁ + MultiHeadCrossAttention(x₁, enc_output, enc_output))
x₃ = LayerNorm(x₂ + FFN(x₂))
```

After N = 6 decoder layers, the output goes through:

- **Linear layer** — projects from `d_model` to vocabulary size (e.g., 32,000 dimensions)
- **Softmax** — converts logits to a probability distribution over the vocabulary

At inference time, the token with the highest probability (or a sampled token) is selected, appended to the output sequence, and fed back into the decoder for the next step. This continues until a special `[EOS]` token is generated.

---

## 9. Putting It All Together: Information Flow

To make the end-to-end flow concrete, here is what happens during a translation from English to Spanish for the sentence "I love NLP":

**Encoder pass:**
1. "I", "love", "NLP" are tokenized and converted to embeddings.
2. Positional encodings are added — each token now carries position information.
3. Through 6 encoder layers, self-attention refines each token's representation using the full context. "love" knows it relates to "I" and "NLP"; "NLP" knows it is the object being loved.
4. The encoder outputs 3 context vectors `(K, V)` — one per input token.

**Decoder pass:**
1. The decoder starts with just the `[SOS]` (start of sequence) token.
2. In each decoder layer, masked self-attention over generated tokens → cross-attention to encoder `K, V` → FFN.
3. The linear + softmax projects to vocabulary probabilities. "Me" is selected (highest probability).
4. "Me" is appended to the output, fed back in. Now the decoder generates "encanta" conditioned on "[SOS] Me" and the full encoder context.
5. This repeats until `[EOS]` is generated: "Me encanta el NLP."

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
| **Interpretability** | Attention weights are partially interpretable but not a complete explanation of model behavior |

---

## 11. Quick Reference: Key Hyperparameters

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

You do not need to build a Transformer from scratch. HuggingFace's `transformers` library gives you pretrained models with a few lines of code.

```python
from transformers import pipeline

# Translation
translator = pipeline("translation_en_to_fr", model="Helsinki-NLP/opus-mt-en-fr")
result = translator("I love machine learning.")
print(result[0]['translation_text'])
# Output: "J'adore l'apprentissage automatique."

# Text generation with a GPT-style decoder-only Transformer
generator = pipeline("text-generation", model="gpt2")
output = generator("The Transformer architecture works by", max_length=50)
print(output[0]['generated_text'])

# Sentence embeddings (encoder output)
from transformers import AutoTokenizer, AutoModel
import torch

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
model = AutoModel.from_pretrained("bert-base-uncased")

inputs = tokenizer("The cat sat on the mat", return_tensors="pt")
with torch.no_grad():
    outputs = model(**inputs)

# outputs.last_hidden_state: shape (1, seq_len, 768)
# Each row is the encoder's context vector for that token
encoder_output = outputs.last_hidden_state
print(f"Encoder output shape: {encoder_output.shape}")
# torch.Size([1, 9, 768])
```

---

## 13. Glossary

**Attention weight** — A scalar between 0 and 1 representing how much one token attends to another. Produced by softmax over Q·Kᵀ scores.

**Context vector** — The output of an attention layer for a given query. A weighted sum of Value vectors encoding information from attended tokens.

**Cross-attention** — Attention where Queries come from the decoder and Keys/Values come from the encoder. Bridges the two halves of the model.

**d_model** — The dimensionality of all embeddings and hidden representations in the model.

**dₖ** — The dimension of each attention head's Query and Key vectors. Equal to `d_model / h`.

**Feed-forward network (FFN)** — A two-layer MLP applied identically and independently to each token position after the attention sub-layer.

**Layer normalization** — Normalization applied across feature dimensions of a single token, stabilizing training.

**Masked attention** — Attention where future positions are masked (set to -∞) before softmax, enforcing left-to-right generation in the decoder.

**Multi-head attention** — Running `h` attention operations in parallel over different linear projections of Q, K, V, then concatenating outputs.

**Positional encoding** — A fixed or learned vector added to token embeddings to inject sequence order information.

**Query / Key / Value (Q, K, V)** — The three projections of the input used in attention. Q is what you are looking for, K is what tokens offer, V is what they contain.

**Residual connection** — Adding the sub-layer input directly to its output (`x + sublayer(x)`), enabling gradients to flow without vanishing.

**Scaled dot-product attention** — The core attention operation: `softmax(Q·Kᵀ / √dₖ) · V`.

**Self-attention** — Attention where Q, K, and V all come from the same sequence, allowing every token to attend to every other token.

**Teacher forcing** — A training technique where the ground-truth previous token is fed as input to the decoder at each step, rather than the model's own prediction.

---

## 14. Further Reading

- Vaswani et al. (2017). *Attention Is All You Need.* [arxiv.org/abs/1706.03762](https://arxiv.org/abs/1706.03762) — the original paper. Read the architecture section carefully; it is well-written and precise.
- Alammar, J. *The Illustrated Transformer.* [jalammar.github.io](https://jalammar.github.io/illustrated-transformer/) — the best visual walkthrough available. If any part of this post was unclear, this will clarify it.
- HuggingFace. *Transformers documentation.* [huggingface.co/docs/transformers](https://huggingface.co/docs/transformers) — practical entry point for working with pre-trained Transformer models.
- Huang et al. (2022). *Are Transformers Effective for Time Series Forecasting?* [arxiv.org/abs/2205.13504](https://arxiv.org/abs/2205.13504) — a useful sanity check on where Transformers do and do not dominate.
