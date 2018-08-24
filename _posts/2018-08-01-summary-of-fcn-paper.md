---
layout: post
title: Summary of FCN paper
description: This paper introduced the idea of converting classification networks into fully convolutional networks that produce coarse outputs. Then these coarse outputs are connected to dense pixels for pixelwise prediction. The ideas the authors used in improving semantic segmentation are follows 1. Adapting Classifiers for dense prediction 2. Upsampling using deconvolution
---

## Paper
- Title: Fully Convolutional Networks for Semantic Segmentation([FCN](https://arxiv.org/abs/1411.4038))
- Submission date: 14 Nov 2014

## Achievements
The FCN models are tested on the following datasets, the results reported are compared to the previous state-of-the-art methods.

- [PASCAL VOC 2012](http://host.robots.ox.ac.uk/pascal/VOC/voc2012/) 
    * achieved the best results on mean intersection over union (IoU) by a relative margin of 20%
- [NYUDv2](https://cs.nyu.edu/~silberman/datasets/nyu_depth_v2.html) dataset
    * achieved mean IoU by 18.8% relative improvement
- [SIFT](https://github.com/shelhamer/fcn.berkeleyvision.org/blob/master/data/sift-flow/README.md) Flow dataset
    * achieved the best results on pixel accuracy by a relative margin of 8.39%


## Key Contributions
This paper introduced the idea of converting classification networks into fully convolutional networks that produce coarse outputs. Then these coarse outputs are connected to dense pixels for pixelwise prediction.

The ideas the authors used in improving semantic segmentation are follows

1. Adapting Classifiers for dense prediction
2. Upsampling using deconvolution

## Architecture

This paper cast the Imagenet classifiers into fully convolutional layers and augmented them with in-network upsampling and pixel-wise loss. The learned representation of Imagenet classifiers are fine-tuned for semantic segmentation. 

<p align="center">
<img src="/assets/Images/fcn/image1.png" alt="fcn">
</p>

<!-- ### From classifier to dense FCN -->
Authors adapted and extended the then best classfication networks on Imagenet. They compared the performance of each network by inference time and mean IoU on the validation set of PASCAL VOC.
<p align="center">
<img src="/assets/Images/fcn/image2.png" alt="fcn">
</p>

The 32 pixel stride at the final prediction layer (of unmodified classfication networks) limits the scale of detail in the upsampled output. They addresed this issue by adding skips between layers to fuse coarse, semantic information and fine, location information. See the architecture below: FCN-32, FCN-16 and FCN-8 architectures have stride of 32, 16 and 8 respectively.

<p align="center">
<img src="/assets/Images/fcn/architecture2.png" alt="fcn">

<!-- <small><a href="https://medium.com/@wilburdes/semantic-segmentation-using-fully-convolutional-neural-networks-86e45336f99b">Source</a></small> -->
</p>


<!-- Then continuing in this fashion by fusing predictions from pool3 with a 2 x upsampling of predictions fused from pool4 and conv7, building the net FCN-8s.
 -->
## Conclusion
Extending the classification networks to semantic segmentation, and modifying these architectures with multi-resolution layer combinations dramatically improved the then state-of-the-art results.



