---
layout: topic
title: "NLP Fundamentals: From Text Preprocessing to Modern Language Models"
permalink: /blogs/nlpconcepts/
date: 2026-04-25
categories: [machine-learning, nlp, fundamentals]
tags: [nlp, tokenization, text-preprocessing, tf-idf, word2vec, embeddings, sequence-models, transformers, ner, language-modeling]
description: "A beginner-friendly guide to Natural Language Processing fundamentals, covering text preprocessing, tokenization, vectorization, embeddings, language models, sequence models, attention, transformers, NLP tasks, and evaluation."
image: /blogs/assests/word2vec.jpg
---

# NLP Fundamentals: From Text Preprocessing to Modern Language Models

Natural Language Processing (NLP) is the field of machine learning that helps computers understand, process, and generate human language. It powers search engines, chatbots, translation systems, sentiment analysis, spam filters, document classification, summarization, and modern large language models.

This guide covers the basic and fundamental ideas you should know before going deeper into advanced NLP, transformers, RAG, or LLM applications.

![Word2Vec embedding visualization](/blogs/assests/word2vec.jpg)

---

## What Is NLP?

NLP is about converting unstructured text into a form that machines can analyze and learn from.

Examples of NLP tasks:

| Task | Example |
|---|---|
| Text classification | Classify an email as spam or not spam |
| Sentiment analysis | Detect whether a review is positive or negative |
| Named Entity Recognition | Find names, companies, dates, and locations |
| Machine translation | Translate English to Hindi |
| Question answering | Answer questions from a document |
| Summarization | Produce a shorter version of a long article |
| Text generation | Generate paragraphs, code, or chat responses |

The central challenge is that language is ambiguous. The same word can mean different things depending on context.

```text
I deposited money in the bank.
I sat near the river bank.
```

Both sentences use the word `bank`, but the meaning is different.

---

## Core NLP Pipeline

A classic NLP pipeline looks like this:

```text
Raw text -> Cleaning -> Tokenization -> Normalization -> Feature extraction -> Model -> Prediction
```

For modern deep learning NLP, the pipeline may look like this:

```text
Raw text -> Tokenization -> Embeddings -> Neural network or Transformer -> Output
```

The exact pipeline depends on the task, but the core goal is always the same: represent language in a useful numerical form.

---

## Corpus, Document, Sentence, and Token

These four terms appear everywhere in NLP.

| Term | Meaning |
|---|---|
| Corpus | A collection of text data |
| Document | One text item inside a corpus |
| Sentence | A sentence inside a document |
| Token | A smaller unit such as a word, subword, or punctuation mark |

Example:

```text
Corpus: 10,000 customer reviews
Document: 1 review
Sentence: "The delivery was fast."
Tokens: ["The", "delivery", "was", "fast", "."]
```

---

## Tokenization

Tokenization is the process of breaking text into smaller units called tokens.

A token can be:

- A word
- A subword
- A punctuation mark
- A number
- A special symbol

Example:

```text
Text: "I don't like delays."
Tokens: ["I", "do", "n't", "like", "delays", "."]
```

Tokenization matters because every downstream NLP model works on tokens, not raw text.

### Word, Character, and Subword Tokenization

| Tokenization type | Example | Used when |
|---|---|---|
| Word-level | `playing` | Simple NLP pipelines |
| Character-level | `p`, `l`, `a`, `y`, `i`, `n`, `g` | Handling noisy text or unknown words |
| Subword-level | `play`, `ing` | Modern transformers and LLMs |

Subword tokenization is important because it handles rare words better. For example, a model may not know `unhappiness` as a full word, but it can understand pieces like `un`, `happi`, and `ness`.

---

## Text Preprocessing

Text preprocessing prepares raw text for modeling. The goal is to reduce noise while preserving useful meaning.

Common preprocessing steps:

| Step | What it does | Example |
|---|---|---|
| Lowercasing | Converts text to lowercase | `Hello` -> `hello` |
| Removing punctuation | Removes punctuation marks when not useful | `hello!` -> `hello` |
| Removing stop words | Removes frequent words like `the`, `is`, `and` | Useful for bag-of-words models |
| Stemming | Cuts suffixes crudely | `playing` -> `play` |
| Lemmatization | Converts word to dictionary base form | `better` -> `good` |
| Handling numbers | Keeps, removes, or normalizes numbers | `10,000` -> `<NUMBER>` |
| Handling URLs/emails | Replaces or removes web/email patterns | `abc@mail.com` -> `<EMAIL>` |

Preprocessing should be task-aware. For sentiment analysis, punctuation and casing may carry meaning. For legal or medical NLP, removing words too aggressively can harm accuracy.

---

## Stemming vs Lemmatization

### Stemming

Stemming removes word endings using simple rules.

```text
connected -> connect
connecting -> connect
connection -> connect
```

It is fast but crude. Sometimes it creates invalid words.

### Lemmatization

Lemmatization uses vocabulary and grammar to return the correct base word.

```text
running -> run
better -> good
was -> be
```

Lemmatization is slower but more accurate.

| Method | Speed | Accuracy | Output quality |
|---|---|---|---|
| Stemming | Fast | Lower | May produce invalid roots |
| Lemmatization | Slower | Higher | Produces valid base words |

---

## Stop Words

Stop words are very common words such as:

```text
the, is, am, are, was, to, of, and, in
```

Removing stop words can help older feature-based models like Bag of Words or TF-IDF. But stop word removal is not always good.

Example:

```text
This movie is good.
This movie is not good.
```

If `not` is removed, the meaning changes completely. For modern transformer models, stop words are usually kept because the model learns how to use context.

---

## N-Grams

An n-gram is a sequence of `n` tokens.

| Type | Example from "New Delhi is beautiful" |
|---|---|
| Unigram | `New`, `Delhi`, `is`, `beautiful` |
| Bigram | `New Delhi`, `Delhi is`, `is beautiful` |
| Trigram | `New Delhi is`, `Delhi is beautiful` |

N-grams help capture short phrases. For example, `New Delhi` should often be treated as one meaningful unit instead of two unrelated words.

---

## Part-of-Speech Tagging

Part-of-Speech (POS) tagging assigns grammatical roles to words.

```text
Sentence: The cat sleeps.
The -> Determiner
cat -> Noun
sleeps -> Verb
```

POS tagging helps with parsing, information extraction, named entity recognition, and grammar-aware NLP tasks.

---

## Named Entity Recognition

Named Entity Recognition (NER) identifies important entities in text.

```text
Apple hired engineers in Bengaluru on Monday.

Apple -> Organization
Bengaluru -> Location
Monday -> Date
```

Common entity types include:

- Person
- Organization
- Location
- Date
- Time
- Money
- Product
- Medical terms
- Legal references

NER is useful for document understanding, search, analytics, and information extraction.

---

## Numericalization: Converting Text to Numbers

Machine learning models need numbers, not raw text. Numericalization is the process of converting tokens into numerical representations.

Common methods:

| Method | Idea | Limitation |
|---|---|---|
| One-hot encoding | One vector position per word | Very sparse, no meaning similarity |
| Count vectorization | Count word frequency | Ignores word order and meaning |
| TF-IDF | Weigh important words higher | Still sparse and context-independent |
| Word embeddings | Dense vectors learned from data | Older embeddings give one meaning per word |
| Contextual embeddings | Meaning changes with context | More expensive, used in transformers |

---

## One-Hot Encoding

One-hot encoding represents each word as a vector with one `1` and the rest `0`s.

```text
Vocabulary: [cat, dog, apple]
cat   -> [1, 0, 0]
dog   -> [0, 1, 0]
apple -> [0, 0, 1]
```

Problem: one-hot vectors do not capture similarity. `cat` and `dog` are as different as `cat` and `apple`.

---

## Bag of Words

Bag of Words represents text using word counts.

```text
Sentence 1: I love NLP
Sentence 2: I love machine learning

Vocabulary: [I, love, NLP, machine, learning]
Sentence 1 -> [1, 1, 1, 0, 0]
Sentence 2 -> [1, 1, 0, 1, 1]
```

Bag of Words is simple and useful for classical ML models, but it ignores word order and context.

---

## TF-IDF

TF-IDF stands for **Term Frequency - Inverse Document Frequency**.

It gives higher weight to words that are frequent in a document but rare across the full corpus.

```text
TF-IDF = Term Frequency x Inverse Document Frequency
```

Intuition:

- Words like `the` appear everywhere, so they get low weight.
- Words like `refund`, `crash`, or `delivery` may be more meaningful, so they get higher weight.

TF-IDF is strong for search, classification, and baseline models.

---

## Word Embeddings

Word embeddings represent words as dense vectors. Similar words get similar vectors.

```text
king  -> [0.21, -0.44, 0.73, ...]
queen -> [0.19, -0.40, 0.71, ...]
```

Embeddings solve a major problem of one-hot encoding: they capture semantic similarity.

Classic embedding methods include:

- Word2Vec
- GloVe
- FastText

---

## Word2Vec

Word2Vec learns word vectors from context. It has two main architectures:

| Architecture | Input | Prediction target |
|---|---|---|
| CBOW | Surrounding context words | Middle word |
| Skip-Gram | Middle word | Surrounding context words |

### CBOW

CBOW predicts a target word from its surrounding words.

```text
Context: The ___ is barking
Target: dog
```

### Skip-Gram

Skip-Gram predicts surrounding words from the target word.

```text
Target: dog
Context: The, is, barking
```

Skip-Gram often performs better for rare words, while CBOW can be faster.

![Word2Vec](/blogs/assests/word2vec.jpg)

---

## GloVe

GloVe stands for **Global Vectors for Word Representation**.

While Word2Vec learns from local context windows, GloVe uses global word co-occurrence statistics across the corpus.

The intuition is that word meaning can be learned from how often words appear together.

Example:

```text
ice is likely to co-occur with cold
steam is likely to co-occur with hot
```

GloVe captures these statistical relationships in dense vectors.

---

## FastText

FastText improves word embeddings by representing words using subword units or character n-grams.

Example:

```text
playing -> play, lay, ayi, yin, ing
```

This helps with:

- Rare words
- Misspellings
- Morphologically rich languages
- Unknown words not seen during training

FastText can create a useful vector even for words missing from the training vocabulary.

---

## The Limitation of Static Embeddings

Word2Vec, GloVe, and FastText create one vector per word. That means the word has the same representation in every sentence.

Problem:

```text
I deposited cash in the bank.
The boat stopped near the river bank.
```

A static embedding gives `bank` one vector, even though the meanings are different.

This is why contextual embeddings became important.

---

## Sequential Data

Text is sequential. Word order matters.

```text
Dog bites man.
Man bites dog.
```

Both sentences contain the same words, but the meaning changes because the order changes.

Traditional Bag of Words models lose this order information. Sequence models were introduced to handle this.

---

## RNNs, LSTMs, and GRUs

Recurrent Neural Networks (RNNs) process text one token at a time while maintaining a hidden state.

```text
Token 1 -> hidden state -> Token 2 -> hidden state -> Token 3
```

RNNs can model sequences, but they struggle with long-range dependencies.

LSTMs and GRUs improve RNNs by using gates that control what information to keep, forget, or update.

| Model | Strength | Limitation |
|---|---|---|
| RNN | Basic sequence modeling | Struggles with long context |
| LSTM | Handles longer dependencies better | Slower than transformers |
| GRU | Simpler than LSTM | Still sequential |

---

## ELMo: Contextual Word Embeddings

ELMo stands for **Embeddings from Language Models**.

ELMo generates context-dependent word embeddings using bidirectional LSTMs. This means the same word can have different vectors depending on the sentence.

```text
bank in "river bank" != bank in "money bank"
```

This was a major step from static embeddings to contextual representations.

---

## Bidirectional Language Models

A bidirectional language model reads context from both directions:

- Left to right
- Right to left

This helps the model understand a word using both previous and future context.

Bidirectional modeling is useful for understanding tasks such as classification, entity recognition, and question answering. BERT later made this idea much more powerful using transformers.

---

## Attention

Attention allows a model to focus on the most relevant words when making a prediction.

Example:

```text
The animal did not cross the road because it was tired.
```

To understand `it`, the model should pay more attention to `animal` than to `road`.

Attention helps models handle long sentences better than basic RNNs.

---

## Transformers

Transformers are the foundation of modern NLP. They use self-attention instead of processing tokens strictly one by one.

Important transformer ideas:

| Concept | Meaning |
|---|---|
| Self-attention | Each token attends to other tokens in the sequence |
| Multi-head attention | Different attention heads learn different relationships |
| Positional encoding | Adds word order information |
| Feed-forward network | Processes token representations after attention |
| Layer normalization | Stabilizes training |

Transformers power models like BERT, GPT, T5, and many modern LLMs.

---

## BERT vs GPT

BERT and GPT are both transformer-based, but they are designed differently.

| Model | Architecture | Best for |
|---|---|---|
| BERT | Encoder-only transformer | Understanding tasks such as classification, NER, QA |
| GPT | Decoder-only transformer | Text generation and chat |
| T5 | Encoder-decoder transformer | Text-to-text tasks such as translation and summarization |

BERT learns by predicting masked words. GPT learns by predicting the next token.

---

## ULMFiT

ULMFiT stands for **Universal Language Model Fine-tuning for Text Classification**.

It showed that transfer learning works well for NLP. The idea is to start with a pretrained language model, adapt it to a target domain, and then fine-tune it for classification.

ULMFiT has three major steps:

1. General-domain language model pretraining
2. Target-task language model fine-tuning
3. Target-task classifier fine-tuning

![ULMFiT](/blogs/assests/ulmfit.png)

Important techniques introduced or popularized by ULMFiT include discriminative fine-tuning and slanted triangular learning rates.

---

## Common NLP Tasks

### Text Classification

Assign one or more labels to text.

Examples:

- Spam detection
- Topic classification
- Intent classification
- Toxic comment detection

### Sentiment Analysis

Detect emotion or opinion in text.

```text
"The product is excellent" -> Positive
"The product broke in two days" -> Negative
```

### Information Extraction

Extract structured data from unstructured text.

```text
Invoice total: $250
Due date: 20 May 2026
Vendor: ABC Pvt Ltd
```

### Question Answering

Answer a question using a passage, document, database, or knowledge base.

### Summarization

Create a shorter version of a longer text.

Types:

- Extractive summarization: selects important sentences from the original text
- Abstractive summarization: generates new sentences that capture the meaning

### Machine Translation

Translate text from one language to another.

---

## Evaluation Metrics in NLP

Different NLP tasks use different metrics.

| Task | Common metrics |
|---|---|
| Classification | Accuracy, precision, recall, F1-score |
| NER | Entity-level precision, recall, F1-score |
| Translation | BLEU, METEOR, COMET |
| Summarization | ROUGE, BERTScore, human review |
| Question answering | Exact match, F1-score |
| Language modeling | Perplexity |

### Precision, Recall, and F1

Precision answers: Of the predictions made, how many were correct?

Recall answers: Of the actual correct items, how many did we find?

F1-score is the balance between precision and recall.

### Perplexity

Perplexity measures how well a language model predicts text. Lower perplexity generally means the model is more confident and accurate in predicting the next token.

---

## Practical NLP Workflow

A practical NLP project usually follows this process:

1. Define the task clearly
2. Collect or label text data
3. Explore common words, labels, and errors
4. Clean and preprocess text
5. Choose a representation method
6. Train a baseline model
7. Evaluate using task-specific metrics
8. Inspect errors manually
9. Improve features, data, model, or prompts
10. Deploy and monitor performance

Always build a simple baseline first. For many problems, TF-IDF plus logistic regression is a strong starting point.

---

## Classical NLP vs Modern NLP

| Area | Classical NLP | Modern NLP |
|---|---|---|
| Representation | Bag of Words, TF-IDF | Embeddings, transformers |
| Models | Naive Bayes, SVM, logistic regression | BERT, GPT, T5, LLMs |
| Feature engineering | Manual | Mostly learned from data |
| Data need | Works with smaller data | Often needs more compute and data |
| Interpretability | Easier | Harder |

Classical NLP is still useful, especially for small datasets, fast baselines, and interpretable systems.

---

## Key Takeaways

- NLP converts language into numerical representations that models can process.
- Tokenization is the first major step in most NLP pipelines.
- Text preprocessing should match the task, not follow a fixed recipe blindly.
- Bag of Words and TF-IDF are strong classical baselines.
- Word2Vec, GloVe, and FastText create dense word vectors.
- Static embeddings cannot fully handle context-dependent meaning.
- RNNs, LSTMs, and GRUs model word order but struggle with long context.
- Attention and transformers are the foundation of modern NLP.
- BERT is strong for understanding tasks; GPT is strong for generation tasks.
- Evaluation must match the task and include manual error analysis.

---

## Suggested Learning Path

1. Learn tokenization, preprocessing, Bag of Words, and TF-IDF.
2. Practice classification with Naive Bayes or logistic regression.
3. Learn word embeddings: Word2Vec, GloVe, and FastText.
4. Study sequence models: RNN, LSTM, and GRU.
5. Learn attention and transformers.
6. Study BERT for understanding tasks.
7. Study GPT and LLMs for generation tasks.
8. Learn RAG for document-based question answering.

---

## Further Reading

- [Word2Vec Paper: Efficient Estimation of Word Representations in Vector Space](https://arxiv.org/abs/1301.3781)
- [GloVe: Global Vectors for Word Representation](https://nlp.stanford.edu/projects/glove/)
- [ULMFiT Paper](https://arxiv.org/abs/1801.06146)
- [Attention Is All You Need](https://arxiv.org/abs/1706.03762)
- [BERT Paper](https://arxiv.org/abs/1810.04805)
