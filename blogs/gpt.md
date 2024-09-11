# GPT
  - Large Language Models (like GPT measure the "likelihood" of text, given reference text.
  - They provide  a probability to a text sequence based on a corpus of text.
  - **Casual Language Modeling** measure the probability of the next word in a sequence.
  - **Masked Language Model** measure the probability of a word masked out in the text.
  - Option 1: Maximum Likelihood Estimation of *exact sequence* 
  - Option 2: Estimate the probability of the *sequence of words*, rather than the entire sentence.

## N-gram Language Model
  - Using a maximum likelihood estimation over an entire text is problematic due to sparsity.
  - Instead we model the individual words , allowing us to decompose the sequence into the product of conditional probabilities.

