# GPT

  - Generative Pre-Training i.e GPT model used transformers as they provide more structured memory for handing long-term dependencies.
  - GPT consist of 2 stages:
    
    -  Learning a high capicity language model on a large unlabeled corpus of text.

    - Fine-tuning stage. where we adapt the model.

  - The first stage is learning a high capacity language model on a large unlabeled Corpus of text.

  - Second, fine-tuning stage where we adapt the model, that is the learn 
parameters to a target task using the corresponding supervised objective.

  - This model is a 12-layer decoder-only transformer with masked self-attention heads.

  - The reason to use only decoder is that, the feature of the self-attention layer preventing positions from attending to subsequent positions by masking fits well for language modeling task, in which we predict the next word using the previous words.

# LLMs – Characteristics

  1. General purpose models :
     - Handles a wide range of NLP tasks.
     - Trained on diverse text datasets. 

2. Learn and memorizes:
    - LLMs understand syntax and semantics.
    - Memorizes details from training data.
    - Provides factual information based on learned data. 
3. Stateless : 
     - LLMs process each input independently.
     - No memory of previous interaction.
     - Each query is treated as seperate instance.
4. Stochastic:
   - LLMs exhibit variability in their outputs.
    - Responses can differe with the same input. 
    - Ensures diverse and contextually appropriate responses. 

# Parameters

 Model Parameters
These are learned during training and represent the actual "knowledge" of the model.

**Weights and Biases**
These are the primary parameters.
Found in the layers of the neural network (attention layers, feedforward layers, etc.).
For example, GPT-3 has 175 billion such parameters.

**Hyperparameters**
These are set before training and control the architecture and learning process:

**1. Model Architecture Parameters**
Number of layers (depth) – e.g., 12, 24, 96 transformer blocks.
Hidden size (width) – e.g., 768, 1024, 12288 (dimensionality of embeddings).
Number of attention heads – e.g., 12, 16, 96.
Intermediate feedforward size – typically 4× hidden size.
Vocabulary size – size of the token set (e.g., 50k tokens).
Position embeddings – to capture word order.

**2. Training Hyperparameters**
Learning rate
Batch size
Optimizer type – e.g., Adam, AdamW
Dropout rate
Weight decay
Gradient clipping
**3. Tokenizer-related Parameters**
Type of tokenizer – e.g., Byte-Pair Encoding (BPE), WordPiece, SentencePiece.
Max sequence length – maximum number of tokens the model can process (e.g., 512, 2048, 8192).

# Top-k Vs. Top-p Sampling:

* At each step of text generation, the model predicts a probability distribution over the entire vocabulary (all possible words). Top-k sampling selects the k most likely words from this distribution and then resamples from only those k words.

* Instead of selecting a fixed number of words (like Top-k), Top-p sampling selects the smallest set of most likely words whose cumulative probability exceeds a threshold p.


# Temperature - Impact of Probabilities distribution

* Its a hyperparameter that controls the **randomness** or **creativity** of a language model. It adjust the model confidence in selecting the next word in sequence.

* Essentially, it modifies how "confident" the model is in its prediction. 

* In case of Low Temperature Values the model becomes more confident and deteministic. 

* In case of High Temperature Value model becomes more diverse and creative. 

# GenAI Project life cycle

1) Identify the use case.
  - Text Generation.
  - Coversational AI
  - Text Summarization
  - Sentiment Analysis
  - Question Answering
  - Machine Translation
  - Text to Speech
  - Code Generation
  - Image Generation
  - Audio Generation
  - Text to Image


2) Choose a foundation Model.
3) Prompting or Tuning
4) Evalution
5) Deployment





```
Research Paper 1 : Language Models are Few-Shot Learners 
Link :   https://arxiv.org/abs/2005.14165

Research Paper 2 : GPT-4 Technical Report
Link : https://arxiv.org/pdf/2303.08774

Research Paper 3 : BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding
Link : https://arxiv.org/pdf/1810.04805
```
## N-gram Language Model
  - Using a maximum likelihood estimation over an entire text is problematic due to sparsity.
  - Instead we model the individual words , allowing us to decompose the sequence into the product of conditional probabilities.

