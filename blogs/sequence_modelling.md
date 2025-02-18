# RNN

A Recurrent Neural Network (RNN) is a type of neural network designed for processing sequential data. It features loops that allow information to be retained across time steps, making it effective at capturing temporal patterns. This capability makes RNNs particularly useful for applications such as time series forecasting, speech recognition, and natural language processing. More advanced variants, like Long Short-Term Memory (LSTM) and Gated Recurrent Unit (GRU) networks, have been developed to overcome the limitations of traditional RNNs, such as difficulty in learning long-term dependencies.

![RNN neuron](assests/RNN1.jpg)

- Regular neuron which has only one weight Wx to represent its contribution, a recurrent neuron has two weights Wx and Wy to represent the contribution of the input and previous output respectively.


### Flat Model

![alt text](flattenmodel.png)

-   Generally each layer has multiple recurrent neurons . In this layer each neuron is connected to the input and the previous output and number of units within each layer is a hyperparameter. 

- Recurrent neurons or cells maintain some kind of 
state or has a memory, which gets added or updated by the recurrent connections, which allows it to learn features from the sequential data

### Types of RNN 
