---
layout: post
title: Summary of Faster R-CNN Paper
description: The authors developed a faster r-cnn newtork model for object detection. This network builds on the top of previously introduced R-CNN network to efficiently classify object proposals using deep convolutional networks.
---

## Paper
- Title: Faster R-CNN ([fasterRCNN](https://arxiv.org/abs/1506.01497))
- Submission date: 4 jun 2015

## Key Contributions

- Introduced a region proposal network (RPN) that shares full-image convolutional features with detection network, thus enabling cost-free region proposals.
- The RPN is a trained end-to-end to generate high-quality region proposals, which are used by Fast R-CNN for detection.
- we furthur merge RPN and Fast R-CNN into a single network by sharing their convolutional features - using the recent popular terminology of neural networks with "attention" mechanisms, the RPN component tells the unified network where to look.
- For the very deep VGG-16 model, our detection system has a frame rate of 5fps (including all steps) on a GPU. 

## Achievements
- In ILSVRC and COCO 2015 competitions, Faster R-CNN and RPN are the basis of several 1st-place entries in the tracks of Imagenet detection, Imagenet localization, COCO detection, and COCO segmentation. 

## Datasets used
we also report results 
- on the MS COCO dataset
- investigate the improvements on PASCAL VOC using the COCO data.
-  Evaluation on 
PASCAL VOC detection benchmarks
1. PASCAL VOC 2007 detection benchmark 
        - 5000 trainval images
        - 5000 test images
        - 20 object categories
 2. PASCAL VOC 2012 detection benchmarks

## Object detection networks

Before this paper, advances in object detection are driven by region proposal methods and Region based Convolutional neural networks (R-CNN). Previous state-of-the-art object detection networks depended on region proposal algorithms to hypothesize object locations. Then advanced object detection networks like SPPnet and Fast R-CNN have reduced their running time, exposing region proposal computation as a bottleneck.

The most popular region proposal methods like selective-search and EdgeBoxes were being used in efficient object detecion methods. Even Fast R-CNN method uses selective-search for generating region proposals. The region proposal step consumes the same time as the detecion network.

In this paper, authors showed that computing proposals with deep convolutional neural netwoks led to a elegant and effective solution where proposal computation is nearly a cost-free given the detection network's computation. For this, they introduced Region Proposal Networks (RPNs) that share convolutional layers with the state-of-the-art object detection network are introduced. An RPN is a fully convolutional network that simultaneously predicts object bounds and object scores at each position.

## Region Proposal Networks

Authors observed that convolutional feature maps used by region based detectors, like Fast R-CNN can also be used for generating region proposals.

On top of these convolutional features, they constructed an RPN by adding a few additional convolutional layers that simultaneously regress region bounds and objectness scores at each location on a rectangular grid.

RPN takes an image of any size as input. This ouputs a set of rectangular object proposals each with objectness score. Our ultimate goal is to share computation with a Fast R-CNN object detection network, we assume that both nets share a common set of convolutional layers. RPNs are designed to efficiently predict region proposals with a wide range of scales and aspect ratios.

## Faster R-CNN
To unify RPNs with Fast R-CNN object detection networks, we propose a training scheme that alternates between fine-tuning for the region proposal task and then fine-tuning for object detection, while keeping the proposals fixed. This scheme converges quickly and produces a unified network with convolutional features that are shared between both tasks. 
Thus the detection system, Faster R-CNN consists of two modules, i.e..,
    1. Deep fully convolutional network that proposes regions (RPN).
    2. Second module is the Fast R-CNN detector that uses the proposed regions.
Entire system is a single and unified network for object detection. RPN module tells the Fast R-CNN module where to look.

<p align="center">
<img src="/assets/Images/faster-rcnn/faster_rcnn.png" alt="tab_contents">
</p>
 
## Anchor boxes

Authors introduced novel "anchor" boxes that serve as references at multiple scales and aspect ratios. 

To generate region proposals, slide a small network over the convolutional feature map output by last convolutional layer. 

This small network takes an input n x n spatial window of the input feature map. Each sliding window is mapped to a lower dimensional feature (256-d for ZF and 512-d for VGG, with ReLU). This feature is fed into two sibling fully-connected layers - a box regression layer (reg) and a box classification layer (cls). At each sliding-window location, we simultaneously predict multiple region proposals, where the number of maximum possible proposals for each location is denoted as k. so, the reg layer has 4k outputs encoding the coordinates of k boxes, and the cls layer outputs 2k scores that estimate probability of object or not. An anchor is centered at the sliding window and is associated with scale and aspect ratio. For a convolutional feature map of a size W x H, there are W*H*k anchors in total.


<p align="center">
<img src="/assets/Images/faster-rcnn/anchors.png" alt="tab_contents">
</p>


## Training Networks
Four ways of training networks with features shared are mentioned below.
1. Alternating training
    - First train a RPN.
    - Use these region proposals to train Fast R-CNN.
    - The network fine-tuned by Fast R-CNN is then used to initialize RPN, and these process is iterated.
2. Approximate joint training
    - During training, RPN and Fast R-CNN networks are merged into one network.
    - In each SGD itearation, the forward pass generates region Proposals which are treated just like fixed, pre-computed proposals when training a Fast R-CNN detector.
    - The backward propogation takes place as usual, where for shared layers the backward propogated signals from both the RPN loss and the Fast R-CNN loss are combined.
3. Non-approximate joint training
    - The bounding boxes predicted by RPN are also functions of the input.
    - The RoI pooling layer in Fast R-CNN accepts the convolutional features and also the predicted bounding boxes as input, so a theoritically valid backpropogation solver should also involve gradients w.r.t the box-coordinates.
    - These gradients are ignored in the above join training solution.
    - In non-approximate joint training solution, we need an RoI pooling layer that is differentiable w.r.t the box coordinates. 
    - This is an nontrivial problem and a solution can be given by an "RoI warping" layer.
4. 4-step alternate training
    - we train RPN, This network is initialized with Imagenet pre-trained model and fine-tuned end-to-end for RP task.
    - we train a seperate detection network by Fast R-CNN using the proposals generated by 1 RPN. The detection network is initialized by the ImageNet-pre-trained model. At this point, the two networks do not share convolutional layers.
    - we use the detector network to initialize RPN training. And we fix the shared convolutional layers and only fine-tune the layers unique to RPN.
    Now two networks share convolutional layers.
    - Now keeping the shared convolutional layers fixed, we fine-tune the unique layers of Fast R-CNN.

## Implementation details 
- Some RPN proposals highly overlap with each other
- To reduce redundancy, we adopt non-maximum suppression (NMS) on the proposal regions based on their cls scores. 
- After NMS, we use top-N ranked proposal regions for detection.
- For ImageNet-pre-trained, 
        - use "fast version" of ZF-net
            + 5 convolutional layers
            + 3 fully connected layers
        - public VGG-16 model
            + 13 convolutional layers
            + 3 fully connected layers

## Observations     
- RPNs with Fast R-CNN produce detection accuracy better than the strong baseline of selective-search with Fast R-CNNs.
- Our method waives nearly all computational burdens of selective-search at test-time, the effective running-time for proposals is just 10 milliseconds. 
- Though using expensive very deep models, our detection method still has a frame rate of 5fps (including all steps) on a GPU, and thus is a practical object detecion system in terms of both speed and accuracy.
- Zeiler and Fergus model (ZF) 
    + 5 shareable convolutional layers.
- Simonyan and Zisserman model (VGG-16)
    + 13 shareable convolutional layers.
- Using RPN yields much faster detection system than using either SS or EB because of shared convolutional computations, the fewer proposals also reduce region-wise fully connected layers cost.
- The following are the observations from Ablation experiments:
    - It showed the effect of sharing convolutional layers between RPN and Fast R-CNN detection network. 
    - Next, we disentangle the RPN's influence on training Fast R-CNN detection network.
    - Next, we sepearately investigate the role of RPN's cls and reg outputs by tuning off either of them at test-time.
        + It shows that cls scores account for the accuracy of the highest ranked proposals.
        + The anchor boxes, thus having multiple scales and multiple aspect ratios, are not sufficient for accurate detection.
    - We also evaluated the effects of more powerful networks on the proposal quality of RPN alone. 
        

## Performance of VGG

|Method| #proposals |data | mAP (%)|
|:-----:|:----:|:-----------:|
|SS|2000|07|66.9|
|SS|2000|07+12|70.0|
|RPN + VGG, unshared|300|07|68.5|
|RPN + VGG, shared|300|07|69.9|
|RPN + VGG, shared|300|07+12|73.2| 
|RPN + VGG, shared|300|COCO+07+12|78.8|

|Method| #proposals |data | mAP (%)|
|:-----:|:----:|:-----------:|
|SS|2000|12|65.7|
|SS|2000|07++12|68.4|
|RPN + VGG, shared|300|12|67.0|
|RPN + VGG, shared|300|07++12|70.4| 
|RPN + VGG, shared|300|COCO+07++12|75.9|

## Conclusion

We evaluated our method on PASCAL VOC detection benchmarks where RPNs with Fast R-CNNs produced detection accuracy better than the strong baseline of selective-search with Fast R-CNNs.