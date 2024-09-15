# Diffusion Model 

Diffusion Model is a Generative model , that means It is capable of generating the data. It generates the the data through the process of adding the noise into data and then removing it. It takes the sample of pure noise and iteratively transform it into data like images or audio. It works in following 2 steps: 

## Forward Diffusion Process :
  Process that begin with clean data sample(like an image), and we gradually add the noise into data over multiple steps. This process transforms the data close to  pure Gausian Noise. This process is modeled as *Markov Chain* , where each step is small amount of noise.  

## Reverse Diffusion Process:
   The main task of Diffusion Model is to learn the reverese process that means denoising the data step by step. Typicallly. a neural network (often a U-Net) is trained to perform this denoising task. 
