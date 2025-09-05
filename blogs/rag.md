# RAG Model

Retrieval-Augmented Generation (RAG) is a technique that enhances generative AI models by combining them with a retrieval component. It works by retrieving relevant information from a large corpus of documents or knowledge base and then using this information to generate more accurate and contextually rich responses. RAG models leverage both the generative capabilities of AI and the precision of retrieval systems, enabling more informed and coherent outputs, especially in complex or knowledge-intensive tasks.

RAG Model is hybrid model that has two components:

1) Retrieval Model
2) Generation Model

In practice, when RAG model is prompted to generate some text or answer of any question , it first retrieves the relevent information from the knowledge base and provides the augmented query. It then uses this context as a direct input to guide the generative process, due to this it also uses real world data and dont fully rely on training data.This dynamic approch allows RAG model to produce more accurate and updated information.

![ RAG :Source Educative.io](assests/RAG.JPG) Source:Educative.io

## Retriever Architecture Overview

Most modern retrievers use two different serach techniqes :

* **Keyword Search** : Looks for documents containing the exact words found in the prompt.
* **Semantic Search** : Looks for the documents with similar meaning to the prompt.
* **Metadata Filtering**

Each search technique will be used to return a collection of documents. There will be many documents that will appear in both search technique. At this point, each list is filtered down based on their metadata.

Now the retriever has two filtered lists one generated with the keyword search and other with semantic search. The reretriever returns the top-ranked documents from this final list. The documents are sent along to be added to the augmented prompt.
