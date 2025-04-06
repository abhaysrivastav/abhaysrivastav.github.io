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

# RAG

[Link to RAG Model](rag.md)


# LangChain 

  * Framework for building apps with large language models.
  * Integrates with models like GPT.
  * Enables context-aware, data-driven NLP application.

### Key features of LangChain:

  * Prompt Template
  * Memory
  * Agents
  * Tools


### LangChain works by:

  * Building Blocks
  * Combining Components
  * Executing Chains


The memory system in LangChain handles 2 actions:
  
  1) **Reading** : Retrieving relevent information from past interactions.
  
  2) **Writing** : Storing new information for future use.  

These actions are integrated in **chain execution process**, ensuring that each interactions is performed by past events and future interactions can be build upon which has already been established.

In Langchain, different types of memory systems used to manage and retain conversational context across interactions. These memory system decides how past interactions are stored. Each type of memory serves different purpose based on the need of the application. 

**Conversation Buffer Memory** stores all messages in a session. Provides access to the entire conversation.Everytime user interacts with the system both user query and response are stored in a buffer. When the application need to follow a new input, it can access to entire conversation history. Entire history is available in a variable. 

**Conversation Buffer Window Memory** stores the last K interactions. It is useful for recent context and discards older interactions. 

**Coversation Token Buffer Memory** manages context based on token count. Idieal for controlling text length within token limits. 

**Conversation Summary Memory** creates and maintains a summary of the conversation. Provides a condensed version of essential points.Avoid referencing every individual interactions.



**Chunking Strategy** refers to the methods used to divide large piece of data into smaller chunks.It goal is to breakdown the text in such a way that each chunk is still meaningful and retain enough context for the model to process it effectively. **Chunk size** is no of tokens in each chunk. It determines how large each piece of text will be when it splits. **Chunk overlap** is no of characters or tokens that are repeated between consecutive chunks.
**CharacterTextSplitter** is method of splitting text based on a Character.

**Recursive Character Text Splitting Process** using the seperators breakdown a text into smaller chunks by applying these separators in order of significance. 

1. First text split at paragraph and each paragraph becomes a chunk.

2. Second, if any chunk is still too large, the process splits those chunks further inti lines.

3. If the chunk is still large then , it divides based on spaces. 

4. If still large then into characters. 


## Instruction Tuned Model 

In LLMs primary training is unsupervised where the model learn from vast amount of text data without any specific task oriented guidance.LLMs may not be good in following the complex instructions unless they are not tuned in a specific way. Instruction tuned model are fine tuned model where they are trained on a specific instruction-response data. This process trained model to better follow the user's instructions. 

These models are better in **following specific commands**, understand the intent behind the user's instructions. It provides more accurate and contextual appropriate responses. These are more user friendly, accurate and task oriented than their counter parts.

These models are fine-tuned on supervised datasets, which contains instructions and correct responses. 

Application of Instruction tuned model:
  - Summerization
  - Translation 
  - Coding 
  - Answering specific queries.  

### How does instruction tuned model work ?

  - It works by fine-tuning LLMs on a labelled dataset of tasks that involve following instractions. Instruction datasets can be created by humans or another LLM.
  - Each sample in the dataset has 3 parts:
    - An instruction
    - Additional Context
    - Desired output
  The model learns to match its answers with the target outputs.

  - Adding more tasks to the instruction tuning process improves the model's performance, even on the task it has not seen before.


## Fine Tuned Model 

It involves updating all the parameters of a pre-trained model for task-specific dataset. This process talors the model for a specific task. Updating all the parameters in large language model is computational expensive.**Catatrophic Forgetting** is a significant challenge in full fine-tuning, where it may loose previously acquired knowledge when adopted for new task. It may degrade its general abilities. To handle this we can perform **Parameter-Efficient Fine Tuning**.

## Parameter-Efficient Fine Tuning

Is it the method that fine-tunes only a small subset of a model's parameters, rather the entire model to reduce computational cost and memory usage.  

[ Resources ](<resources/Mandatory Readings_ Week 6.pdf>)


In PEFT, there is a technique called **Adapters** that enables task-specific adaptations by adding and training small modules within the model, making it both computationally and storage efficient while preserving the model's ability to generalize well.

Adapters are small modules or layers inserted within each layer of a pre-trained model.Typically within the each layer of transformation.In Training process , instead of updating all the parameters , we only need to train parameters related to these adapters only and parameters for rest of the model remain frozen. 

During fine-tuning, the input passes through the pre-trained layers as usual, but at specific points where adapters are inserted, they are trained on the new task. This allows the model to adapt to the new task without affecting the rest of the model. As a result, it requires less memory and computational power since only the adapters, not the entire model, need to be stored.

### LoRA (Low Rank Adaptation)

Technique to enable efficient fine-tuning of large transformers.It uses low-rank matrix factorization.The goal is to *reduce the number of parameters that need to be updated* during the fine-tuning making it less expensive.In Transformer Model, most of the computation happens in Self attention layer and Feedforward layer. 

In Transformer models, LoRA (Low-Rank Adaptation) is typically applied to the Attention and Feed-Forward layers. These layers are computationally expensive and contain a large number of parameters, making them the ideal candidates for parameter-efficient fine-tuning techniques like LoRA.

1. Attention Layers (Self-Attention or Multi-Head Attention):
The attention layers are the most parameter-heavy components in a Transformer.

Specifically, LoRA is used to fine-tune the projection matrices involved in:

  - Query
  - Key 
  - Value

2. Feed-Forward Layers (MLP layers):
Transformers also have Feed-Forward Neural Networks (FFNs) after each attention block.

These FFNs are linear transformations with large weight matrices.

LoRA decomposes these weight matrices as well, applying the same principle:

𝑊′=𝑊+𝐴×𝐵

Fine-tuning these matrices helps the model adapt to new tasks without updating the entire model.
Model uses its merged weight matrices W' in place of original weight matrices W. Process of creating W' 
from W is known as re-parameterization. Here A and B are learnable parameters. W' has the same dimentions as of W. 
So it keeps the model architecture remains unchanged. 

Why Not Use LoRA Everywhere?
LoRA is mainly beneficial in layers where weight matrices are large and computationally expensive to fine-tune.

Embedding layers or LayerNorm layers usually don’t benefit from LoRA because:

  - They have relatively fewer parameters.

  - They are less critical in terms of task-specific adaptation.


## Evalution of LLMs

* Qualitative evaluation through human verification.
* Quantitative Evalution through Metrics:
    
    * **ROUGE (mainly used for summerization task)** : Set of metrics used to evaluate the quality of summaries generated by machine learning model in the context 
    of NLP. It compares the machine generated summeries to one or more reference summary (mainly created by human) and then find the overlap in term of words and its sequence.
    **ROUGE-N** measures the overlap of N-grams between the generated summary and reference summaries.

    * BLEU
    * METEOR 

## Biases

Biases in data, particularly in terms of data representation, refer to the issue where certain groups are depicted in a
less favorable or inaccurate manner compared to others, even if there is a sufficient amount of data for each group.
This means that while there may be ample data available for various groups, the way these groups are represented
can still be skewed or biased, affecting the fairness and accuracy of analyses and outcomes


## Hallucination

AI hallucination refers to a phenomenon where a large language model (LLM), such as a generative AI chatbot or
computer vision tool, produces outputs that are nonsensical or inaccurate because it perceives patterns or objects that
don’t actually exist or are imperceptible to humans.

AI hallucinations arise from issues like overfitting, data bias, or complex model behavior.


----------------
```
Research Paper 1 : Language Models are Few-Shot Learners 
Link :   https://arxiv.org/abs/2005.14165

Research Paper 2 : GPT-4 Technical Report
Link : https://arxiv.org/pdf/2303.08774

Research Paper 3 : BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding
Link : https://arxiv.org/pdf/1810.04805


```


