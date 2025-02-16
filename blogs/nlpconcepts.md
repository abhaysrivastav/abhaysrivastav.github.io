## Tokenization

The process of breaking down these documents into set of sentences and each sentence into words is called the Tokenization.  The tokens could be individual words punctuations, or part of short forms of the words. For instance, don’t could be broken down into do and n apostrophe t.

## Text Pre-processing Tasks
- Stop words removal
- Stemming : Stemming is a crude process of severing the suffixes of the word without checking whether the word after chopping is a valid dictionary form of a word or not.
- Lemmatization : lemmatization is a more refined way of converting the words to the base form, as it checks the dictionary after the process to ensure the word is a proper vocabulary term.

Besides these pre-processing steps there are many more pre-processing steps, such as creating Bi-grams that is instead of considering New Delhi as two words, we consider it as single word. Then we have Tri-grams etc. Part of speech tagging, which is useful for name entity recognition application.

## Numericalization 

Now that, we have the tokens and have performed the pre-processing steps, we need to encode or numericalize the output tokens into numbers. There are various ways of encoding the token:

- One-Hot Encoding
- Count Vectorization
- TF-IDF (Term Frequency Inverse Document Frequency) Vectorization
- Word Embedding: Learning word embeddings involves using algorithms like Word2Vec and GloVe 

### Word2Vec
Word2Vec Model has two different variants. Continuous Bag-of-Words that is CBOW and Skip-Gram.In CBOW architecture the input layer is the context, that is the surrounding words that is both to the left and the right of the target word and is used to predict the target that is the middle word.In Skip-Gram Architecture, the input layer is the middle word and is used to predict the context that is the words to the left and the right of the target word.
![Word2Vec](assests/word2vec.jpg)

-  From a large corpus when we capture multiple contexts using different sentences, we could expect the word embeddings of similar words to be similar.
-  Typically, the embeddings are represented in 50 or higher dimensions, which makes it difficult to visualize the embeddings.However, we could use the **dimension reduction** technique and reduce the dimension to 2
or 3 and plot it to check, whether we can observe any relationship between the words
