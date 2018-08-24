---
layout: post
title: Summary of Vggnet Paper
description: The authors experimented with different level of depths in convolutional neural networks (ConvNets) and presented the improvement in error reduction with respect to the depth. More non-linear rectification layers (ReLUs) made the decision function more discriminative.
---
<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.2/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

> In this blog, for my notes as well as for the reference of others, I have written a small summary of the paper. 

### About Paper
- [Vggnet](https://arxiv.org/abs/1409.1556) paper title: Very deep convolutional networks for large scale Image Recognition
- Paper submission date: 4th sep 2014. 

### Achievements of the paper
In [ImageNet](http://www.image-net.org) competition 2014, authors secured
- First place in the localisation
- Second place in the classification 

### Key Contributions
- Experimented with different level of depths in convolutional neural networks (ConvNets)
- Presented the improvement in error reduction with respect to the depth 
- More non-linear rectification layers (ReLUs) makes the decision function more discriminative
- Though depth was increased, the number of parameters in ConvNets was not greater than previously proposed shallow networks
- Released two best performing [models](http://www.robots.ox.ac.uk/ ̃vgg/research/very_deep/) to facilitate furthur research 

### Model details
#### Architecture
- Input to the model is a fixed size $$224 \times 224$$ RGB image
- Preprocessing is subtracting the training set RGB value mean from each pixel
- Convolutional layers
    + Stride fixed to 1 pixel
    + padding is 1 pixel for $$3 \times 3$$
- Spatial pooling layers
    + This layers doesn't count to the depth of network by convention
    + Spatial pooling is done using max pooling layers
    + window size is $$2 \times 2$$ 
    + Stride fixed to 2
    + Convnets used 5 max pooling layers
    
#### Training
- Loss function is multinomial logistic regression
- Learning algorithm is mini-batch stochastic gradient descent(SGD) based on back-propogation with momentum
    + Batch size was 256
    + Momentum was 0.9
- Regularisation
    + L2 Weight decay (penalty multiplier was 0.0005)
    + Dropout for first two FC layers (set to 0.5)
- Learning rate
    + Intial: 0.01
    + decreased by 10 when validation set accuracy stopped improving
- Though greater number of parameters and depth compared to alexnet, the ConvNets required required less epochs for loss function to converge due to 
    + more regularisation by large depth and small convolutional kernels
    + pre-initialisation of certain layers
- Training image size
    + S is the smallest side of isotropically-rescaled image
    + Two approaches for setting S
        * Fix S, known as single scale training
            - Here S = 256 and S = 384
        * Vary S, known as multi-scale training
            - S from [S<sub>min</sub>, S<sub>max</sub>] where S<sub>min</sub> = 256, S<sub>max</sub> = 512
    + Then $$224 \times 224$$ image was radomly cropped from rescaled image per SGD iteration
