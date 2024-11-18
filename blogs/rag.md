# RAG Model 
RAG Model is hybrid model that has two components:
  1) Retrieval Model
  2) Generation Model

In practice, when RAG model is prompted to generate some text or answer of any question , it first retrieves the relevent information from the knowledge base and provides the augmented. It then uses this context as a direct input to guide the generative process, due to this it also uses real world data and dont fully rely on training data.This dynamic approch allows RAG model to produce more accurate and updated information.  

![ RAG :Source Educative.io](assests/RAG.JPG) Source:Educative.io 

RAG model has certain challenges: 
  1) RAG model heavily depends on knowledge base , so any problem in knowledge base will affect the model performence. So regular update of knowledge base is required.
  2) Intergrating the Retreival model with generation model together to work well is not easy.
  3) If the data size, scaling an RAG model is an issue in terms of speed and efficiency.



