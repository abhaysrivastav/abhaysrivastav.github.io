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

**One-to-One (1:1)**: This is a standard feedforward neural network used for non-sequential data.

**Many-to-One (N:1)**: This type processes multiple inputs to produce a single output, such as in sentiment analysis.

**One-to-Many (1-N)**: This setup uses a single input to generate multiple outputs, such as in image captioning.

**Many-to-Many (N-N)**: This configuration handles multiple inputs and produces multiple outputs, which is common in machine translation.

**Many-to-Many (N-M)**: This flexible structure allows for varying sequence lengths in both inputs and outputs, useful in applications like video analysis.

**Long Short-Term Memory (LSTM)**

**Gated Recurrent Unit (GRU)**

**Character Prediction**

**Stacked RNNs**

**Bidirectional RNNs**

### Training RNN BPTT

![bptt](assests/bptt.jpg)

- It performs forward propagation from left to right. So, for the first time step the input X0 is multiplied with the weight WX.The bias term is added as well and the activation function is applied on this value to produce Y of 0, which is then compared with the actual output to calculate the error E of 0. Similarly, for the next time step input X1 is multiplied with WX and the previous output is multiplied with the WY and the activation function is applied on the sum of these two values and the bias term to produce the output Y of 1, which is used to calculate the loss for that time step.

- This way the loss terms are calculated for every time step and the cumulative loss is the loss for the entire sequence. The next step is to perform back propagation in which the gradients of the error is calculated with respect to the weights WX and WY.

### LSTM(Long Short Term Memory)

- To address the memory issue, added 2 states: 
    1) Long term state (c) 
    2) Short term state(h)

Forgets or filters not so important old memories and update the old memory and form  new memories.

- LSTM dont have pre-defined logic instead but it uses neural network for it to learn what state to retain or forget. For this it uses 4 gates: 
    1) Main Gate 
    2) Forget Gate 
    3) Input Gate 
    4) Output Gate

Each gate above is fully connected neural network.

![lstm](assests/lstm.jpg)

-   Main neural network produces the output based on the input and previous state of cell and updates the long term memory.

- Forget Gate determines how much of the long-term memory needs to be forgotten or retained.

- Input Gate figures out the important part of the input and concatenates that to the long-term state.

- Output Gate decides how much of the updated long-term memory should be considered as part of the output cell.