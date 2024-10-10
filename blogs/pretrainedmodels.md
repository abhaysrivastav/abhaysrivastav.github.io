# Pre-Trained Model Architectures

### AlexNet

#### Architecture

- 5 Convolutional Layers: These layers extract features like edges, textures, and patterns from the images.
- 3 Fully Connected (FC) Layers: The last three layers are fully connected, similar to traditional neural networks, and they make the final classification decision.
- ReLU Activation: It introduced the use of the ReLU (Rectified Linear Unit) activation function instead of the sigmoid or tanh, speeding up training.
- Dropout: Dropout was introduced to prevent overfitting. It randomly drops units (neurons) during training, which ensures the model doesn’t rely too much on any specific neurons.
- Max Pooling: This technique reduces the spatial dimensions of the output feature maps, helping the model focus on important features while reducing computational complexity.

| **Layer**         | **Output Dimensions** |
|-------------------|-----------------------|
| Input             | 227 x 227 x 3         |
| Conv1             | 55 x 55 x 96          |
| Pool1             | 27 x 27 x 96          |
| Conv2             | 27 x 27 x 256         |
| Pool2             | 13 x 13 x 256         |
| Conv3             | 13 x 13 x 384         |
| Conv4             | 13 x 13 x 384         |
| Conv5             | 13 x 13 x 256         |
| Pool5             | 6 x 6 x 256           |
| Flatten           | 9216                  |
| FC1               | 4096                  |
| FC2               | 4096                  |
| FC3               | 1000 (Softmax Output) |


### ResNet

### VGG

### Inception 

