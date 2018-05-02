---
layout: post
title: "Monocular Dense Reconstruction"
---

Authors: Sanjay Pushparajan, Vivek Kalyan

[[Github]](https://github.com/bowsplinter/mono-reconstruction)

## Motivation

With the rising commoditisation of mobile augmented reality, there is an increasing need to model the geometric world. This allows for more advanced features such as object occlusion, pose tracking, semantic segmentation and eventually world-scale localisation. With this in mind, the goal of this project is to implement monocular dense reconstruction on mobile devices for small objects.

While there are many solutions that involve uploading high resolution photos for cloud processing, advances in mobile processors and state-of-the-art work combining depth prediction using machine learning and traditional structure-from-motion pipelines are making local mesh reconstruction a reality.

## Methodology

To get the 3D mesh from images, there are generally 3 main steps. The first is to create a sparse point cloud by tracking features across images. From this sparse point cloud, a dense point cloud is estimated using depth predictions coupled with the features. Finally, a point cloud to mesh algorithm is used to get a 3D mesh. A texture map can also be generated from input frames and assigned to the mesh.

## Implementation
There are many open source implementations of reconstruction, however none of them cater for mobile. Therefore, we looking to create a full end-to-end pipeline to do reconstruction in the mobile phones. We chose iOS as its ecosystem is more mature, but the concepts can easily be applied to Android. There are 3 main components of our work. A major complexity in our project is integrating these 3 parts together, which has little precedence in terms of previous works, making the undertaking a challenging one.

### Sparse Point Cloud
The first component is in Swift which is used to get the camera intrinsics and extrinsic parameters. These are computed using the phone’s hardware and transformed in a  way that can be read by openCV. Once the point clouds/mesh have been generated, they are rendered using Metal (which is the iOS graphics library).

The second component is in Objective-C. This serves as an bridge between the Swift code and the C++ implementation. The bridge is designed to call the required C++ function by making the appropriate call in Swift. This allows the information to flow between the two runtimes.

The third and final component is the C++ component. This is where the majority of the calculations occur. The openCV library is used for some of its primitives and functions. However, note that despite openCV having a SFM module, our work implements SFM instead of using the library.

### CNN-depth prediction
A CNN that was trained on self-driving car environments was used to predict the depth map of images.

## Results

**Feature Detection**

<img class="center-block" style="padding: 10px 30px;" src="/img/gnv-feature-detection.png">

**Feature Matching**

<img class="center-block" style="padding: 10px 30px;" src="/img/gnv-feature-matching.png">

**Triangulation**

<img class="center-block" style="padding: 10px 30px;" src="/img/gnv-match-triangulation.png">

**Point Cloud**

<img class="center-block" style="padding: 10px 30px;" src="/img/gnv-point-cloud.png">

**CNN Depth Prediction**

<img class="center-block" style="padding: 10px 30px;" src="/img/gnv-cnn-slam.png">