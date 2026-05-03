---
layout: topic
title: "Building a Visual Search System Like Pinterest: A Deep Dive into ML System Design"
permalink: /blogs/visual-search-system/
date: 2025-05-03
categories: [machine-learning, system-design, retrieval]
tags: [visual-search, embeddings, contrastive-learning, ann, faiss, scann]
author: Abhay
image: /blogs/assests/ml-img/visual-search-system/01_system_pipeline.jpg
excerpt: >
        Design a Pinterest-style visual search system end to end: embeddings, contrastive learning, evaluation, and
        scalable approximate nearest neighbor retrieval at billion-image scale.
---

# Building a Visual Search System Like Pinterest: A Deep Dive into ML System Design

> *"A picture is worth a thousand words — but only if you can find the right picture."*

---

## The Story Begins: You're Scrolling Pinterest at 2 AM

It's late. You're scrolling Pinterest and you spot a gorgeous mid-century modern chair in someone's living room photo. You don't know the brand. You don't know the name. All you have is... the image.

You tap the chair. Pinterest zooms in, and within milliseconds, a grid of visually similar chairs floods your screen — from Wayfair, from IKEA, from obscure Etsy sellers you'd never have found otherwise.

**That's a Visual Search System.** And in this post, we're going to design one from scratch — the same way you'd walk through it in a senior ML system design interview.

---

## Chapter 1: What Are We Actually Building?

A visual search system takes a **query image** (or a crop of an image) as input, and returns a **ranked list of visually similar images** from a database of billions.

The key design decisions:
- Results ranked by similarity (most similar first)
- No text queries, no videos — images only
- No personalization (same query → same results for everyone)
- ~100–200 billion images on the platform
- Training data constructed from **user interactions** (clicks, impressions)

Think of it like Google Image Search, but instead of typing "white Eames chair", you just *show* it one.

---

## Chapter 2: Framing It as an ML Problem

### The ML Objective

Our goal: **accurately retrieve images that are visually similar to a query image.**

This is a **ranking problem** — we're not predicting a class label, we're ordering a list by relevance. Think of it like how Google ranks search results, except our "query" is an image and our "relevance" is visual similarity.

### The Core Insight: Representation Learning

Here's the beautiful idea at the heart of this system:

> What if we could map every image to a point in space, such that **visually similar images end up close to each other**?

This is called **representation learning** (or embedding learning). We train a neural network to transform each image into an **embedding vector** — a list of numbers that represents the image's visual "essence."

![Embedding Space Diagram](/blogs/assests/ml-img/visual-search-system/02_embedding_space.jpg)

*In this N-dimensional space, a query image of a dog would be close to other dog images, and far from images of cars, trees, or houses. The system finds "nearest neighbors" in this space.*

**Real-world analogy:** Think of it like how Spotify's "Discover Weekly" works. Each song is mapped to a point in "music taste space." Songs with similar beats, tempo, and mood cluster together. Spotify finds songs near the ones you've liked. We're doing the exact same thing, but for images instead of songs.

---

## Chapter 3: The Data

### What Data Do We Have?

The system stores three types of data:

**Images** — with metadata:
<table>
        <thead>
                <tr>
                        <th>ID</th>
                        <th>Owner</th>
                        <th>Upload Time</th>
                        <th>Tags</th>
                </tr>
        </thead>
        <tbody>
                <tr>
                        <td>1</td>
                        <td>8</td>
                        <td>1658451341</td>
                        <td>Zebra</td>
                </tr>
                <tr>
                        <td>2</td>
                        <td>5</td>
                        <td>1658451841</td>
                        <td>Pasta, Food, Kitchen</td>
                </tr>
                <tr>
                        <td>3</td>
                        <td>19</td>
                        <td>1658821820</td>
                        <td>Children, Family, Party</td>
                </tr>
        </tbody>
</table>

**Users** — demographics (age, location, etc.)

**User-Image Interactions** — the gold mine:
<table>
        <thead>
                <tr>
                        <th>User ID</th>
                        <th>Query Image</th>
                        <th>Displayed Image</th>
                        <th>Position</th>
                        <th>Interaction Type</th>
                </tr>
        </thead>
        <tbody>
                <tr>
                        <td>8</td>
                        <td>2</td>
                        <td>6</td>
                        <td>1</td>
                        <td>Click</td>
                </tr>
                <tr>
                        <td>6</td>
                        <td>3</td>
                        <td>9</td>
                        <td>2</td>
                        <td>Click</td>
                </tr>
                <tr>
                        <td>91</td>
                        <td>5</td>
                        <td>1</td>
                        <td>2</td>
                        <td>Impression</td>
                </tr>
        </tbody>
</table>

These clicks tell us: *"When a user searched with image X and clicked on image Y, they believed Y was visually similar to X."* That's our implicit training signal.

---

## Chapter 4: Feature Engineering — Preparing Images for the Model

Before feeding an image into a neural network, we need to **preprocess** it:

- **Resizing:** Models need fixed-size inputs (e.g., 224×224 pixels)
- **Scaling:** Normalize pixel values to [0, 1]
- **Z-score normalization:** Scale to mean=0, variance=1 (helps training stability)
- **Consistent color mode:** Ensure all images are RGB (not CMYK or grayscale)

Think of preprocessing like standardizing resumes before HR reviews them — everyone gets the same format so the model can make fair comparisons.

---

## Chapter 5: Model Development

### Why Neural Networks?

Two reasons:
1. Neural networks handle **unstructured data** (images, text) naturally
2. Neural networks can **produce embeddings** — traditional ML models can't

### The Architecture: CNN or Transformer

We use a **CNN-based architecture** (like ResNet) or a **Transformer-based** one (like ViT — Vision Transformer):

```
Input Image (224×224)
    ↓
Convolution Layers (feature extraction)
    ↓
Fully Connected Layers
    ↓
Embedding Vector [0.1, 0.8, -1.0, -0.7, 0.0, ...]
```

The embedding vector is the compressed "fingerprint" of the image.

**Research Papers:**
- ResNet: [He et al., 2015 — Deep Residual Learning for Image Recognition](https://arxiv.org/abs/1512.03385)
- Vision Transformer (ViT): [Dosovitskiy et al., 2020 — An Image is Worth 16×16 Words](https://arxiv.org/abs/2010.11929)

### Model Training: Contrastive Learning

This is where the magic happens. We use **contrastive training** to teach the model what "similar" means.

![Contrastive Training Diagram](/blogs/assests/ml-img/visual-search-system/03_contrastive_training.jpg)

For each training example, we give the model:
- A **query image** (a dog)
- 1 **positive image** (another dog — visually similar)
- N-1 **negative images** (cat, house, tree — visually different)

The model must learn to **pull the query and positive closer** in embedding space, while **pushing negatives away.**

**How do we get positive images?** Three options:

| Method | Pros | Cons |
|--------|------|------|
| Human Judgment | High quality | Expensive, slow |
| User Clicks | Free, scalable | Noisy (users click wrong things) |
| Self-Supervision (SimCLR/MoCo) | No labels needed, scalable | May miss semantic similarity |

> **Real-world analogy:** Imagine teaching a dog to recognize its owner. You show it photos of the owner (positive), photos of strangers (negative), and reward it when it distinguishes correctly. Over thousands of examples, it learns the visual "essence" of the owner. That's contrastive learning.

**Research Papers:**
- SimCLR: [Chen et al., 2020 — A Simple Framework for Contrastive Learning](https://arxiv.org/abs/2002.05709)
- MoCo: [He et al., 2019 — Momentum Contrast for Unsupervised Visual Representation Learning](https://arxiv.org/abs/1911.05722)

### The Loss Function: How We Measure "Good" Embeddings

We use a **contrastive loss** computed in 3 steps:

1. **Compute similarities** — dot product or cosine similarity between query embedding and all other embeddings
2. **Softmax** — convert similarities to probabilities
3. **Cross-entropy** — penalize the model when it doesn't rank the positive image highest

```
Similarities → [0.95 (positive), 0.02, 0.01, 0.003]
After Softmax → [0.95, 0.02, 0.01, 0.003]  
Cross-entropy with Label [1, 0, 0, 0] → Loss
```

The model minimizes this loss over millions of training examples.

---

## Chapter 6: Evaluation — How Do We Know It's Working?

### Offline Metrics

We have an evaluation dataset where each query image has candidate images with **human-labeled similarity scores (0–5)**.

![Metrics Comparison](/blogs/assests/ml-img/visual-search-system/04_metrics_comparison.jpg)

**The candidates:**

- **MRR (Mean Reciprocal Rank)** — looks only at the rank of the first relevant item. Simple but misses overall ranking quality. *Discarded.*
- **Recall@k** — ratio of relevant items retrieved. Useless when there are millions of relevant items in a database of billions. *Discarded.*
- **Precision@k** — fraction of top-k results that are relevant. Ignores ranking order. *Discarded.*
- **mAP (Mean Average Precision)** — considers ranking, but only works for binary relevance. *Discarded.*
- **nDCG (Normalized Discounted Cumulative Gain)** ✅ — handles **graded relevance** (0–5 scores), rewards putting the best results first, normalized for fair comparison. **Winner.**

**nDCG Formula:**

$$DCG_p = \sum_{i=1}^{p} \frac{rel_i}{\log_2(i+1)}$$

$$nDCG_p = \frac{DCG_p}{IDCG_p}$$

Where IDCG is the ideal (perfect) ranking. A perfect system scores 1.0.

> **Real-world analogy:** Imagine rating a waiter's performance. If they bring your main course first, soup second, and appetizer third — that's a bad ordering. nDCG measures how close the model's ordering is to the ideal ordering, with heavier penalties for getting the most important items wrong.

**Research:** [Järvelin & Kekäläinen, 2002 — Cumulated Gain-Based Evaluation of IR Techniques](https://dl.acm.org/doi/10.1145/582415.582418)

### Online Metrics

Once deployed, we track:
- **Click-Through Rate (CTR)** = Clicked images / Total suggested images
- **Average time spent** on suggested images daily/weekly/monthly

A higher CTR means users are finding the results relevant. Time spent measures deeper engagement.

---

## Chapter 7: Serving — Making It Fast at Scale

This is where the system design gets real. With 100–200 billion images, how do you find the nearest neighbors to a query embedding **in milliseconds**?

![System Pipeline](/blogs/assests/ml-img/visual-search-system/01_system_pipeline.jpg)

### The Prediction Pipeline

When a user queries with an image:

1. **Embedding Generation Service** — preprocesses the query image and runs it through the trained model to get the query embedding vector
2. **Nearest Neighbor Service** — finds the most similar embeddings in the index
3. **Re-ranking Service** — applies business logic (remove NSFW, remove duplicates, remove private images)
4. **Results** delivered to user

### The Indexing Pipeline

Running in the background, continuously:
- All platform images are processed through the same model
- Their embeddings are stored in an **index table**
- When new images are uploaded, they're immediately indexed

### The Hard Part: Nearest Neighbor Search at Billion Scale

Finding the exact closest point in a 100B-point database would take **O(N × D)** time — impossibly slow.

Enter **Approximate Nearest Neighbor (ANN)** algorithms:

![ANN Algorithms](/blogs/assests/ml-img/visual-search-system/05_ann_algorithms.jpg)

**Three families of ANN algorithms:**

#### 1. Tree-Based ANN
Algorithms like KD-trees, R-trees, and **ANNOY** (Spotify's algorithm!) partition the embedding space into a tree. To search, we only traverse the relevant partition — like going to the correct floor of a library instead of searching every shelf.

- Paper: [Bentley, 1975 — Multidimensional Binary Search Trees (KD-tree)](https://dl.acm.org/doi/10.1145/361002.361007)

#### 2. Locality Sensitive Hashing (LSH)
LSH uses special hash functions that map **similar points to the same "bucket."** To search, we only look in the same bucket as the query point.

> **Analogy:** Like sorting songs by genre. If you want songs similar to a jazz track, you only search the jazz bucket — not the entire music library.

- Paper: [Indyk & Motwani, 1998 — Approximate Nearest Neighbors: Towards Removing the Curse of Dimensionality](https://dl.acm.org/doi/10.1145/276698.276876)

#### 3. Clustering-Based ANN (FAISS, ScaNN)
Cluster all embeddings into groups. To search, find the nearest cluster center, then only search within that cluster.

- **FAISS** (Meta): [Johnson et al., 2017 — Billion-scale similarity search with GPUs](https://arxiv.org/abs/1702.08734)
- **ScaNN** (Google): [Guo et al., 2020 — Accelerating Large-Scale Inference with Anisotropic Vector Quantization](https://arxiv.org/abs/1908.10396)

**Which to use?** For a system with 100–200 billion images, **clustering-based ANN (FAISS or ScaNN) is the pragmatic choice** — they scale to billions with sub-linear query time.

---

## Chapter 8: Putting It All Together

Here's the complete story of what happens when you search Pinterest by image:

```
You crop a chair from a photo
        ↓
[Preprocessing: resize to 224×224, normalize pixels]
        ↓
[Embedding Generation: ResNet/ViT → 128-dim vector]
        ↓
[ANN Search: FAISS finds top-1000 nearest embeddings in <10ms]
        ↓
[Re-ranking: remove private pins, near-duplicates, NSFW]
        ↓
[Top 50 visually similar results returned to you]
```

Meanwhile, every image ever uploaded to Pinterest has already gone through the indexing pipeline — their embeddings pre-computed and stored in a FAISS index, ready for instant lookup.

---

## Chapter 9: What Could Make It Even Better?

If you had extra time in an interview (or extra engineering resources), here's what's worth discussing:

- **Smart Cropping with Object Detection** — automatically detect and crop the interesting object before embedding, so you search for "the chair" not "the whole room"
- **Multi-modal Search** — enable text queries like "white Eames chair" to work alongside image queries (explored in Chapter 4 of the source material)
- **Graph Neural Networks** — learn better representations by modeling relationships between images
- **Active Learning** — intelligently decide which images to send for human labeling to get maximum training signal per dollar
- **Content Moderation** — detect and block inappropriate images from the index

---

## Summary: Key Design Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| ML Category | Ranking (Representation Learning) | Output is a ranked list |
| Model Architecture | CNN (ResNet) or ViT | Best for image inputs |
| Training Method | Contrastive Learning (SimCLR) | No explicit labels needed |
| Positive Sample Source | Self-supervision + Clicks | Scalable + signal quality |
| Offline Metric | nDCG | Handles graded relevance + ranking quality |
| Online Metric | CTR + Time Spent | Measures real user satisfaction |
| ANN Algorithm | FAISS / ScaNN | Scales to 100B+ images |

---

## References & Papers

1. ResNet — [Deep Residual Learning for Image Recognition](https://arxiv.org/abs/1512.03385) (He et al., 2015)
2. Vision Transformer (ViT) — [An Image is Worth 16×16 Words](https://arxiv.org/abs/2010.11929) (Dosovitskiy et al., 2020)
3. SimCLR — [A Simple Framework for Contrastive Learning of Visual Representations](https://arxiv.org/abs/2002.05709) (Chen et al., 2020)
4. MoCo — [Momentum Contrast for Unsupervised Visual Representation Learning](https://arxiv.org/abs/1911.05722) (He et al., 2019)
5. nDCG — [Cumulated Gain-Based Evaluation of IR Techniques](https://dl.acm.org/doi/10.1145/582415.582418) (Järvelin & Kekäläinen, 2002)
6. KD-Tree — [Multidimensional Binary Search Trees](https://dl.acm.org/doi/10.1145/361002.361007) (Bentley, 1975)
7. LSH — [Approximate Nearest Neighbors](https://dl.acm.org/doi/10.1145/276698.276876) (Indyk & Motwani, 1998)
8. FAISS — [Billion-scale Similarity Search with GPUs](https://arxiv.org/abs/1702.08734) (Johnson et al., 2017)
9. ScaNN — [Accelerating Large-Scale Inference with Anisotropic Vector Quantization](https://arxiv.org/abs/1908.10396) (Guo et al., 2020)
10. Contrastive Loss — [Dimensionality Reduction by Learning an Invariant Mapping](https://yann.lecun.com/exdb/publis/pdf/hadsell-chopra-lecun-06.pdf) (Hadsell et al., 2006)
11. ANNOY (Spotify) — [GitHub: spotify/annoy](https://github.com/spotify/annoy)

---

*This blog is based on Chapter 2 of "Machine Learning System Design Interview" by Ali Aminian & Alex Xu. Highly recommended if you're preparing for ML system design interviews.*

---

*If you found this useful, feel free to star the repo or connect with me on LinkedIn!*
