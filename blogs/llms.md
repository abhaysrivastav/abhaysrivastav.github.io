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
   - 


## N-gram Language Model
  - Using a maximum likelihood estimation over an entire text is problematic due to sparsity.
  - Instead we model the individual words , allowing us to decompose the sequence into the product of conditional probabilities.

