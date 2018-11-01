---
layout: post
title: Action Recognition - Summary of Two Steam CNN paper
description: The authors developed a two stream convolutional newtork model for action recognition in videos. They investigated a different architecture based on two seperate recognition streams i.e, spatial and temporal recognition, which are then combined by late fusion.
---

## About Paper
- Title: Two-Stream Convolutional Networks for Action Recognition in Videos ([Two-Stream ConvNets](https://arxiv.org/abs/1406.2199))
- Submission date: 9 jun 2014


## Key Contributions

- Authors proposed a two-stream Convolutional Network architecture which incorporates spatial and temporal networks.
- Demonstrated a ConvNet trained on multi-frame dense optical flow able to achieve very good performace inspite of limited training data.
- They showed that multi-task learning, applied to two different action classification datasets, can be used to increase the amount of training data and improve the performance on both


## Action recognition in videos

Recognition of human actions in videos is a challenging task because this also involves temporal component of videos in addition to the image classification. In this paper, authors aimed at extending deep ConvNets to action recognition. Previously this task was addressed by inputing stacked video frames to the ConvNet. But it was observed that results were very poor compared to the hand-crafted features. 

Therefore authors investigated a different architecture based on two seperate recognition streams i.e, spatial and temporal recognition, which are then combined by late fusion.

<p align="center">
<img src="/assets/Images/two_stream/two-stream.png" alt="two-stream">
</p>

Videos can naturally be decomposed into spatial and temporal components. The spatial part in the individual frame appearance carries information about scenes and objects depicted in the video. The temporal part in the form of motion across the frames conveys the movement of the observer (the camera)
and the objects. These two parts are seperately implemented using a deep ConvNet, softmax scores of which are combined the late fusion. 

The two fusion methods are considered: averaging and training a multi-class linear SVM on stacked $$L_{2}$$-normalised softmax scores as features.

Spatial Convolutional network is essentially an image classification architecture which can be pre-trained on a large image classification dataset like ImageNet. The temporal convolutional network takes a stack of optical flow displacement fields between several consecutive video frames as input. The brief description of optical flow ConvNets is given below.

## Optical flow ConvNets

Authors described a ConvNet model that forms the temporal recognition stream of the full architecture. The input to the above ConvNet model is formed by stacking optical flow displacement fields between several consecutive frames. These inputs explicitly describes the motion between video frames, which makes the action recognition easier, as the network does not need to estimate motion implicitly. 

<p align="center">
<img src="/assets/Images/two_stream/optical-flow.png" alt="optical-flow">
</p>
The above set of images describe the optical flow, (a) and (b) images are the pair of two consecutive video frames with the area around a moving hand outlined with a cyan rectangle.
(c): The image displays a dense optical flow 
(d): horizontal component $$d^{x}$$ of the displacement vector field.
(e): vertical component $$d^{y}$$ of the displacement vector field.
We can observe (d) and (e) highlight the moving hand and bow. The input to a ConvNet contains a multiple flows.

The two variations of the optical flow-based input are described below:
<p align="center">
<img src="/assets/Images/two_stream/optical_flow_stack.png" alt="optical_flow_stack">
</p>
The above method describes the optical flow stacking. It can be seen as a set of displacement vector fields $$d_{t}$$ between the pairs of consecutive frames $$t$$ and $$t + 1$$. This method samples the displacement vectors d at the same location (u, v) in multiple frames.

<p align="center">
<img src="/assets/Images/two_stream/trajectories.png" alt="trajectories">
</p>   

The above method is an alternative motion representation. Here the method samples the vectors at the locations (u, v) - $$p_{k}$$ along the trajectory.


## Experiments

The above architectures were trained and evaluated on the below standard video actions benchmarks
+ [UCF-101](http://crcv.ucf.edu/data/UCF101.php)
    * 13k videos in total (180 frames/video on average)
    * annotated into 101 action classes
+ [HMDB-51](http://serre-lab.clps.brown.edu/resource/hmdb-a-large-human-motion-database/)
    * 6.8k videos in total
    * annotated into 51 action classes

The following are the observations from the results of experiments.
- Spatial ConvNet solely trained on the UCF-101 dataset leads to over-fitting, therefore authors opted for training the last layer on top of a pre-trained ConvNet.
- Experiments on temporal ConvNet resulted that optical flow stacking performs better than trajectory stacking.
- Temporal ConvNets significantly outperform the spatial ConvNets, which confirms the importance of motion importance for action recognition.
- Multi-task learning performs the best, as it allows the training procedure to exploit all available training data.
- Temporal and spatial recognition streams are complementary, as their fusion significantly improves on both (6% over temporal and 14% over spatial nets
- SVM-based fusion of softmax scores outperforms fusion by averaging



## Conclusion

The two recognition streams are complementary and the combination deep architecture significantly outperforms and is competitive with the state of the art shallow representations in spite of being trained on relatively small datasets.