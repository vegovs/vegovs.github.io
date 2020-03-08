---
layout: post
title: "TEK5030 - Computer Vision"
author: Vegard Bergsvik Øvstegård
date: 07-03-2020
description: "Kompendium TEK5030"
geometry: margin=2cm
tags: [TEK5030]
image:
    feature: tek5030.jpg
---

<details><summary markdown="span"><code>Contents</code></summary>
* TOC
{:toc}
</details>

# From keypoints to correspondences

## Local patches

* Covariant feature point detectors
    * Location (x, y), scale σ and orientation θ.


* Normalize local patches surrounding keypoints
    * Canonical scale
    * Canonical orientation
        ![](../images/tek5030/tek5030_w4_4.png)

# Estimating canonical orientation

* Find dominant orientation of the image patch
    * This is given by xmax, the eigenvector of M corresponding to λmax  (the larger eigenvalue)
    * Rotate the patch according to this angle

* Orientation from Histogram of Gradients (HoG)
    ![](../images/tek5030/tek5030_w4_5.png)

## Local patches

* Covariant feature point detectors
    * Location (x, y), scale σ and orientation θ.


* Normalize local patches surrounding keypoints
    * Canonical scale
    * Canonical orientation
        ![](../images/tek5030/tek5030_w4_4.png)

* Covariant feature point detectors
    * Affine transformation A

* Normalize local patches surrounding keypoints
    * Canonical affine transformation
        ![](../images/tek5030/tek5030_w4_6.png)

## Overview of point feature matching

1. Detect a set of distinct feature points
2. Define a patch around each point
3. Extract and normalize the patch
4. Compute a local descriptor
5. Match local descriptors

![](../images/tek5030/tek5030_w4_7.png)

## Feature descriptors

* Simplest descriptor: Vector of raw intensity values

* How to compare two such vectors?
    * Sum of squared differences (SSD)
        ![](../images/tek5030/tek5030_w4_8.png)

* Normalized correlation
    ![](../images/tek5030/tek5030_w4_9.png)


# Feature descriptors

## Histogram of Gradients (HOG) descriptors
![](../images/tek5030/tek5030_w4_11.png)

### SIFT detector
![](../images/tek5030/tek5030_w4_12.png)
![](../images/tek5030/tek5030_w4_13.png)

Patch at detected position, scale, orientation
![](../images/tek5030/tek5030_w4_16.png)

* Extract patch around detected keypoint
* Normalize the patch
to canonical scale and orientation
* Resize patch to 16x16 pixels
![](../images/tek5030/tek5030_w4_15.png)
* Compute the gradients
    - Unaffected by additive intensity change

* Apply a Gaussian weighting function
    - Weighs down gradients far from the centre
    - Avoids sudden changes in the descriptor with small changes in the window position
![](../images/tek5030/tek5030_w4_14.png)
* Divide the patch into 16 4x4 pixels squares
![](../images/tek5030/tek5030_w4_17.png)
* Compute gradient direction histograms over 8 directions in each square
    - Trilinear interpolation
    - Robust to small shifts, while preserving some spatial information
![](../images/tek5030/tek5030_w4_18.png)
* Concatenate the histograms to obtain
    a 128 dimensional feature vector
* Normalize to unit length
    - Invariant to multiplicative contrast change
* Threshold gradient magnitudes to avoid excessive influence of high gradients
    - Clamp gradients > 0.2
    - Renormalize

![](../images/tek5030/tek5030_w4_19.png)
![](../images/tek5030/tek5030_w4_20.png)
Example: Feature comparison
![](../images/tek5030/tek5030_w4_21.png)
![](../images/tek5030/tek5030_w4_22.png)
![](../images/tek5030/tek5030_w4_23.png)
SIFT summary

* Extract a 16x16 patch around detected keypoint
* Compute the gradients and apply a Gaussian weighting function
* Divide the window into a 4x4 grid of cells
* Compute gradient direction histograms over 8 directions in each cell
* Concatenate the histograms to obtain a 128 dimensional feature vector
* Normalize to unit length

## Binary descriptors

* Extremely efficient construction and comparison
* Based on pairwise intensity comparisons
    - Sampling pattern around keypoint
    - Set of sampling pairs
    - Feature descriptor vector is a binary string:
* Matching using Hamming distance:

![](../images/tek5030/tek5030_w4_24.png)
![](../images/tek5030/tek5030_w4_25.png)
![](../images/tek5030/tek5030_w4_26.png)

Binary descriptors

* Often achieves very good performance compared to SIFT/SURF
* Much faster than SIFT/SURF

![](../images/tek5030/tek5030_w4_27.png)

# Feature matching
## Overview of point feature matching
1. Detect a set of distinct feature points
2. Define a patch around each point
3. Extract and normalize the patch
4. Compute a local descriptor
5. Match local descriptors

## Distance between descriptors
* Define distance function that compares two descriptors
    - L1 distance (SAD):
        ![](../images/tek5030/tek5030_w4_29.png)
    - L2 distance (SSD):
        ![](../images/tek5030/tek5030_w4_30.png)
    - Hamming distance:
        ![](../images/tek5030/tek5030_w4_31.png)

## At which threshold do we get a good match?
![](../images/tek5030/tek5030_w4_32.png)

## Evaluating matching performance
AUC: Area Under the Curve
![](../images/tek5030/tek5030_w4_33.png)

## Matching strategy

* Compare all
* Take the closest
    - Or k closest
    - And/or within a (low) thresholded distance

* Choose the N best putative matches

![](../images/tek5030/tek5030_w4_34.png)
![](../images/tek5030/tek5030_w4_35.png)

## Which matches are good?
### Nearest Neighbour Distance Ratio
* For a descriptor $$f_a$$ in $$I_a$$, take the two closest descriptors $$f_b1$$ and $$f_b^2$$ in $$I_b$$
* Perform ratio test: $$\frac{d(fa, fb1)}{d(fa, fb2)}$$
    - Low distance ratio: $$f_b^1$$ can be a good match
    - High distance ratio: $$f_b^1$$ can be an ambiguous or incorrect match
![](../images/tek5030/tek5030_w4_36.png)
![](../images/tek5030/tek5030_w4_37.png)

## Example: Holmenkollen
![](../images/tek5030/tek5030_w4_38.png)
## Cross check test

* Choose matches $$(fa, fb)$$ so that
    - $$fb$$ is the best match for fa in $$Ib$$
    - And fa is the best match for $$fb$$ in Ia

![](../images/tek5030/tek5030_w4_39.png)
![](../images/tek5030/tek5030_w4_40.png)
## Matching algorithms
* Comparing all features works well for small sets of images
    - Brute force: BFMatcher in OpenCV
* When the number of features is large, an indexing structure is required
    - For example a k-d tree
    - Training an indexing structure takes time, but accelerates matching
    - FlannBasedMatcher in OpenCV
## Summary
* Matching keypoints
    - Comparing local patches in canonical scale and orientation
* Feature descriptors
    - Robust, distinctive and efficient
* Descriptor types
    - HoG descriptors
    - Binary descriptors
* Putative matching
    - Closest match, distance ratio, cross check
* Next lecture
    - Matches that fit a model


# Estimating homographies from feature

## Homographies induced by central projection

![](../images/tek5030/tek5030_w4_41.png)

* Point-correspondences can be determined automatically
* Erroneous correspondences are common
* Robust estimation is required to find $$H$$

    ![](../images/tek5030/tek5030_w4_42.png)

## Estimating the homography between overlapping images

![](../images/tek5030/tek5030_w4_43.png)

* Establish point correspondences $$u_i ↔  u'_i$$
    - Find key points $$u_i ∈ Img_1$$ and $$u'_i ∈ Img_2$$
    - Represent key points by suitable descriptors
    - Determine correspondences $$u_i ↔ u'_i$$ by matching descriptors
    - Some wrong correspondences are to be expected

* Estimate the homography $$H$$ such that $$u'_i = Hu_i ∀ i$$
    - Robust estimation with RANSAC
    - Improved estimation based on RANSAC inliers

* This homography enables us to compose the images into a
    larger image
    - Image mosaicing
    - Panorama

## Adaptive RANSAC

Objective: To robustly fit a model $$y = f(x;\alpha)$$ to a data set $$S$$ containing outliers

Algorithm
1. Let $$N = ∞, S_{IN} = ∅$$ and $$ #iterations = 0$$
2. while $$N > #interations$$ repeat 3-5
3. Estimate parameters $$\alpha_{tst}$$ from a random n-tuple from S
4. Determine inlier set $$S_t$$ i.e. data points within a distance $$i$$ of the model $$y = f(x;\alpha_{txt})$$
5. If $$\|S_{txt}\| > \|S_{IN}\|$$, set $$S_{IN} = S_{txt}, \alpha = \alpha_{tst}, \omega = \frac{\|S_{IN}\|}{\|S\|}$$ and $$N = \frac{log(1-p)}{log(1-\omega^n)}$$ with $$p = 0.99$$ Increase $$#iteratons$$ by 1

## Estimating the homography
* Estimating the homography in a RANSAC scheme requires
    1. **A basic homography estimation method for 𝒏𝒏 point-correspondences**
    2. A way to determine the inlier set of point-correspondences for a given homography

* The homography has 8 degrees of freedom, but it is custom to treat all 9 entries of the matrix
    as unknowns instead of setting one of the entries to 1 which excludes all potential solutions
    where this entry is 0

## Basic homography estimation
![](../images/tek5030/tek5030_w4_43.png)

Observe that the third row in A is a linear combination of the first and second row

![](../images/tek5030/tek5030_w4_44.png)

Hence every correspondence $$u_i ↔ u'_i$$contribute with 2 equations in the 9 unknown entries

Basic homography estimation

* Since H (and thus h) is homogeneous, we
    only need the matrix A to have rank 8 in
    order to determine h up to scale

* It is sufficient with 4 point correspondences
    where no 3 points are collinear

* We can calculate the non-trivial solution to
    the equation $$Ah = 0$$ by SVD
    * $$svd(A) = USV^T$$

* The solution is given by the right singular
    vector without a singular value which is the
    last column of V, i.e. $$h = v_9$$

    ![](../images/tek5030/tek5030_w4_45.png)

* The basic DLT algorithm is never used with more than 4 point-correspondences
* This is because the algorithm performs better when all the terms of 𝐴𝐴 has a similar scale
    - Note that some of the terms will always be of scale 1
* To achieve this, it is common to extend the algorithm with a normalization and a

    denormalization step

    ![](../images/tek5030/tek5030_w4_46.png)

* Estimating the homography in a RANSAC scheme requires
    1. A basic homography estimation method for 𝑖𝑖 point-correspondences
    2. **A way to determine the inlier set of point-correspondences for a given homography**

    ![](../images/tek5030/tek5030_w4_49.png)
    ![](../images/tek5030/tek5030_w4_48.png)

## Robust homography estimation

![](../images/tek5030/tek5030_w4_47.png)

* Finally we would typically re-estimate $$H$$ from all correspondences in $$S_{IN}$$
    - Normalized DLT
    - Minimize $$ϵ = ∑ϵ_i$$ in an iterative optimization method like Levenberg Marquardt

## Image mosaicing

![](../images/tek5030/tek5030_w4_50.png)
* Let us compose these two images into a larger image

    ![](../images/tek5030/tek5030_w4_51.png)
* Find key points and represent by descriptors

    ![](../images/tek5030/tek5030_w4_52.png)
* Establish point-correspondences by matching descriptors
* Several wrong correspondences

    ![](../images/tek5030/tek5030_w4_53.png)
* Establish point-correspondences by matching descriptors
* Several wrong correspondences


![](../images/tek5030/tek5030_w4_54.png)
* Estimate homography $$Hũ = ũ'$$
- OpenCV
    ```c++
    #include "opencv2/calib3d.hpp"
    cv::findHomography(srcPoints, dstPoints, CV_RANSAC);
    ```

- Matlab
    ```matlab
    tform = estimateGeometricTransform(srcPoints,dstPoints,’projective’);
    ```
    ![](../images/tek5030/tek5030_w4_54.png)
* Represent the images in common coordinates  (Note the additional translation!)
- OpenCV
    ```c++
    #include "opencv2/calib3d.hpp"
    cv::warpPerspective(img1, img2, H, output_size);
    ```
- Matlab
    ```matlab
    img2 = imwarp(img1,tform);
    ```

* Now we can compose the images


![](../images/tek5030/tek5030_w4_55.png)
Blending with a ramp + histogram equalization

### SVD
![](../images/tek5030/tek5030_w4_56.png)
![](../images/tek5030/tek5030_w4_57.png)
#  Orientation

* A term describing the relationship between coordinate frames
* Orientation ↔ Rotation
![Image](../images/tek5030/week5/orientation_rotation.png)
![Image](../images/tek5030/week5/camera_earth.png)


* The orientation of the camera frame $$F_c$$ with respect to the world
frame $$F_w$$ can be represented by an orthonormal rotation matrix
![Image](../images/tek5030/week5/orientation.png)

![Image](../images/tek5030/week5/orientation_2.png)

* Special orthogonal group
![Image](../images/tek5030/week5/special_orthogonal_group.png)

* Construction from orthonormal basis vectors
![Image](../images/tek5030/week5/cont_from_ortho.png)

## Principal rotations
![Image](../images/tek5030/week5/principal_rotations.png)

## Action on points

* The matrix $$R_{ab}$$ represents the orientation of $$F_b$$ relative
to $$F_a$$, but it is also a point transformation from $$F_b$$ to $$F_a$$
given that the frames have the same origin

* A point $$x$$ can be transformed from $$F_b$$ to $$F_a$$ by $$x^a = R_{ab}x^b$$
![Image](../images/tek5030/week5/action_on_points.png)

## Composition

* We can chain together consecutive orientations

* If $$R_{ab}$$ is the orientation of $$F_a$$ relative to $$F_a$$ and $$R_{bc}$$ is the
orientation of $$F_c$$ relative to $$F_b$$, then the orientation of $$F_c$$
relative to $$F_a$$ is given by $$R_{ac} = R_{ab}R_{bc}$$
![Image](../images/tek5030/week5/composition.png)
![Image](../images/tek5030/week5/note.png)
* Issue with numerical precision
![Image](../images/tek5030/week5/numerical_precision.png)
* Normalization
![Image](../images/tek5030/week5/normalizaiton.png)

## Other representations

### Euler angles

* Any orientation can be decomposed into a sequence of three principal rotations
![Image](../images/tek5030/week5/2020-03-08-15:50.png)
* The orientation can be represented by the three angles $$(\theta_1,\theta_2,\theta_3)$$  known as Euler angles
![Image](../images/tek5030/week5/euler.png)
* Several sequences can be used
    - ![Image](../images/tek5030/week5/2020-03-08-15:52.png) 
    - To understand Euler angles, we must know the sequence they came from!
    - All sequences have singularities, i.e. orientations where the angles of the sequence are not unique
    - Problematic if we want to recover Euler angles from a rotation matrix

* (roll, pitch, yaw) is often used in navigation to represent the orientation of a vehicle
* The orientation is often described relative to a local North-East-Down (NED) coordinate
frame $$F_w$$ in the world situated directly below the body frame $$F_b$$
* Then the yaw angle is commonly referred to as «heading» since it corresponds to the compass direction
    - North corresponds to 0°, east 90° and so on 

![Image](../images/tek5030/week5/2020-03-08-15:54.png)

* The roll-pitch-yaw sequence $$R_zR_yR_x$$ is singular when $$\theta_2 = \frac{\pi}{2}$$
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-39-52.png)

* (roll, pitch, yaw) is practical for vehicles not
    - Most airplanes, cars and ships

* (roll, pitch, yaw) provides an intuitive understanding about the orientation

### Axis angle

* Euler’s rotation theorem states that the most general
motion of a rigid body with one point fixed is a rotation
about an axis through that point

* So we can represent any orientation by a pair $$(v, \phi)$$ where
$$v = [v_1, v_2, v_3]T$$ is the axis of rotation and $$\phi$$ is the angle of rotation
* This representation is intuitive, but typically not used for computations
* The corresponding rotation matrix is
$$ R_{ab} = cos\phi1 + (1 - cos\phi) vv^T + sin\phi\hat{v} $$
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-45-23.png)

### Unit quaternions
* Quaternions are 4D complex numbers ![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-46-25.png) defined by![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-47-09.png)
* Norm:
        ![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-47-28.png) 

* Unit quaternions $$(\|q\| = 1)$$ is a popular representation for orientation/rotation
* The complex terms are closely related to the axis of rotation, while the real term is closely related to the angle of rotation

![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-48-18.png)

* Composition $$q_{ac} = q_{ab}q_{bc}$$ is very efficient
    - 16 multiplications and 12 additions
    - Matrix multiplication: 27 multiplications and 18 additions
    - Limited numerical precision ⇒ Normalization (divide by $$\|q\|$$ )


* Inverse of unit quaternions![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-51-55.png) 
* Action on point a 𝐱𝐱𝑎𝑎 can be expressed as a product $$p^b = q_{ab}p^aq_{ab}$$
where points are represented as quaternions with zero real
term

![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-53-00.png)

* The rotation matrix corresponding to the unit quaternion
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-53-37.png)

## Pros and cons
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-53-52.png)

## Summary
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-55-14.png)

# Pose

* A term describing the relationship between coordinate frames
* Pose = {Position, Orientation}
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-56-50.png)

* The pose of the camera frame $$F_c$$  with respect to the world frame
$$F_w$$ can be represented by the Euclidean transformation matrix
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-16-59-15.png)
where $$R_{wc} \in SO(3)$$ is a rotation matrix and $$t_{wc}^w \in R^3$$ is a
translation vector given in world coordinates

![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-17-01-14.png)
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-17-02-31.png)

* In illustrations we often represent the pose as an arrow similar to that of the translation vector

## Pose - Invers
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-17-03-37.png)

## Pose - Action on points
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-17-03-57.png)
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-17-04-21.png)

## Pose - Composition
* We can chain together consecutive poses by compounding transformation matrices
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-17-04-44.png)

## Example - Camera on a vehicle in the world
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-17-06-17.png)

## Example - Image rotation about center
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-17-07-51.png)
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-17-08-01.png)

## Summary
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-17-07-32.png)

# The perspective camera model

A mathematical model that describes the
viewing geometry of pinhole cameras

It describes how the perspective projection
maps 3D points in the world to 2D points in
the image

Combined with a distortion model, the
perspective camera model can describe the
viewing geometry of most cameras

![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-18-10-38.png)

* The pose of the world frame relative to the camera frame,
denoted by $$T_{cw}$$, is also a point transformation from $$F_w$$ to $$F_c$$

* General perspective camera model
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-18-11-48.png)


* By multiplying $$\Pi_0$$ with $$T_{wc}$$ we get a very compact  expression that is commonly used to represent the perspective camera model

![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-18-14-42.png)

* We refer to $$K$$ as the *intrinsic* part and $$[R_{cw} t_{cw}^c]$$ as the
*extrinsic* part of the perspective camera model

* The matrix $$K[R_{cw} t_{cw}^c]$$ is often denoted by $$P$$ and referred to as camera’s projection matrix
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-18-16-46.png) 

* Alternative formulation
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-18-17-51.png)
where we have used that
![Image](../images/tek5030/week5/tek5030_w5_2020-03-08-18-18-00.png)

