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


### VGG16

#### Architecture 

- 64 filters of size 3x3 are used in both layers.
The input image size is 224x224x3 (RGB image with 3 channels).
The output after the first and second convolutional layers is 224x224x64. This is because padding is applied (same padding), so the spatial dimensions remain the same while the depth changes due to the 64 filters.
The output is then passed through a max pooling layer with a 2x2 filter and stride 2, which reduces the spatial dimensions by half.
After Pooling: The output becomes 112x112x64.

-Third and Fourth Convolutional Layers:

These layers use 128 filters of size 3x3.
After applying the two convolutional layers, the spatial dimensions remain 112x112, but the depth increases to 128 because of the 128 filters.
The output is then passed through another max pooling layer (2x2 filter, stride 2).
After Pooling: The output becomes 56x56x128.

- Fifth, Sixth, and Seventh Convolutional Layers:
These layers use 256 filters of size 3x3.
After applying these three convolutional layers, the spatial dimensions remain 56x56, and the depth increases to 256 due to the 256 filters.
The output is again passed through a max pooling layer (2x2 filter, stride 2).
After Pooling: The output becomes 28x28x256.

- Eighth to Thirteenth Convolutional Layers:
These layers use 512 filters of size 3x3.
There are six convolutional layers in this section, and the output dimensions remain 28x28x512 after the first three layers.
After the first set of three layers, max pooling is applied, reducing the dimensions to 14x14x512.
After the next three convolutional layers, the output remains 14x14x512, and a final max pooling is applied.
After Pooling: The output becomes 7x7x512.

- Fully Connected Layers:
After the final pooling layer, the output is flattened into a vector of size 25088 (since 7x7x512 = 25088).
This flattened vector is passed through two fully connected layers:
FC1: 4096 units.
FC2: 4096 units.
The final fully connected layer is a Softmax layer with 1000 units (for classification into 1000 categories, like in the ImageNet dataset).

| **Layer**                      | **Output Dimensions**   |
|---------------------------------|-------------------------|
| Input                           | 224x224x3               |
| Conv1 (3x3, 64 filters)         | 224x224x64              |
| Conv2 (3x3, 64 filters)         | 224x224x64              |
| Max Pooling (2x2)               | 112x112x64              |
| Conv3 (3x3, 128 filters)        | 112x112x128             |
| Conv4 (3x3, 128 filters)        | 112x112x128             |
| Max Pooling (2x2)               | 56x56x128               |
| Conv5 (3x3, 256 filters)        | 56x56x256               |
| Conv6 (3x3, 256 filters)        | 56x56x256               |
| Conv7 (3x3, 256 filters)        | 56x56x256               |
| Max Pooling (2x2)               | 28x28x256               |
| Conv8 (3x3, 512 filters)        | 28x28x512               |
| Conv9 (3x3, 512 filters)        | 28x28x512               |
| Conv10 (3x3, 512 filters)       | 28x28x512               |
| Max Pooling (2x2)               | 14x14x512               |
| Conv11 (3x3, 512 filters)       | 14x14x512               |
| Conv12 (3x3, 512 filters)       | 14x14x512               |
| Conv13 (3x3, 512 filters)       | 14x14x512               |
| Max Pooling (2x2)               | 7x7x512                 |
| Flatten                         | 25088                   |
| Fully Connected (FC1)           | 4096                    |
| Fully Connected (FC2)           | 4096                    |
| Output Layer (Softmax, 1000)    | 1000                    |


### ResNet

### Inception 
