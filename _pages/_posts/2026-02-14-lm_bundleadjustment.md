---
layout: post
title: Optimization for Bundle Adjustment
date: 2025-8-04 01:59:00
description: Gauss–Newton and Levenberg–Marquardt Algorithms
tags: optimization images
categories: sample-posts
thumbnail: assets/img/9.jpg
images:
  lightbox2: true
  photoswipe: true
  spotlight: true
  venobox: true
---

## Motivation

Bundle Adjustment is a core optimization problem in 3D computer vision and robotics. It jointly estimates and refines camera poses and 3D point structures by minimizing reprojection error between projected points and image pixels.

Due to the nonlinear nature of the pinhole camera model, the problem is typically solved using **Gauss–Newton** and **Levenberg–Marquardt (LM)**, which behave like second-order optimization methods without explicitly computing second derivatives.

---

## The Bundle Adjustment Problem

### Notation Reference Table

| Symbol                 | Meaning                                       | Dimension / Space               |
| ---------------------- | --------------------------------------------- | ------------------------------- |
| $C_i$                  | Pose of camera $i$ (rotation + translation)   | $SE(3)$                         |
| $R_i$                  | Rotation matrix of camera $i$                 | $3 \times 3$, $SO(3)$           |
| $t_i$                  | Translation vector of camera $i$              | $\mathbb{R}^3$                  |
| $T_i$                  | Homogeneous camera pose matrix                | $4 \times 4$, $SE(3)$           |
| $X_j$                  | 3D world point $j$                            | $\mathbb{R}^3$                  |
| $u_{ij}$               | Observed 2D pixel of point $j$ in camera $i$  | $\mathbb{R}^2$                  |
| $\pi(\cdot)$           | Camera projection function                    | $\mathbb{R}^3 \to \mathbb{R}^2$ |
| $r_{ij}$               | Reprojection residual for observation $(i,j)$ | $\mathbb{R}^2$                  |
| $\theta$               | All optimization parameters (poses + points)  | $\mathbb{R}^{6M + 3N}$          |
| $\Delta \theta$        | Parameter update vector                       | $\mathbb{R}^{6M + 3N}$          |
| $J$                    | Jacobian of residuals w.r.t. parameters       | Sparse matrix                   |
| $H$                    | Hessian of the cost function                  | $(6M + 3N) \times (6M + 3N)$    |
| $J^T J$                | Gauss–Newton Hessian approximation            | Same as $H$                     |
| $\lambda$              | Levenberg–Marquardt damping parameter         | $\mathbb{R}^+$                  |
| $\xi$                  | Pose perturbation (twist coordinates)         | $\mathbb{R}^6$                  |
| $\hat{\xi}$            | Lie algebra matrix form of twist              | $\mathfrak{se}(3)$              |
| $\exp(\hat{\xi})$      | Exponential map to update pose                | $SE(3)$                         |
| $U, W, V$              | Block Hessian matrices (camera, cross, point) | Block matrices                  |
| $S = U - W V^{-1} W^T$ | Schur complement                              | $6M \times 6M$                  |
| $b$                    | Right-hand side of normal equations           | Vector                          |

---

### Problem Setup

**Given:**

- A set of camera poses $\{C_i\}$
- A set of 3D points in world coordinates $\{X_j\}$
- 2D observation measurements $u_{ij}$

## [Lightbox2](https://lokeshdhakar.com/projects/lightbox2/)

<a href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/1/img-2500.jpg" data-lightbox="roadtrip"><img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/1/img-200.jpg" /></a>
<a href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/2/img-2500.jpg" data-lightbox="roadtrip"><img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/2/img-200.jpg" /></a>
<a href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/3/img-2500.jpg" data-lightbox="roadtrip"><img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/3/img-200.jpg" /></a>

---

## [PhotoSwipe](https://photoswipe.com/)

<div class="pswp-gallery pswp-gallery--single-column" id="gallery--getting-started">
  <a href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/2/img-2500.jpg"
    data-pswp-width="1669"
    data-pswp-height="2500"
    target="_blank">
    <img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/2/img-200.jpg" alt="" />
  </a>
  <!-- cropped thumbnail: -->
  <a href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/7/img-2500.jpg"
    data-pswp-width="1875"
    data-pswp-height="2500"
    data-cropped="true"
    target="_blank">
    <img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/7/img-200.jpg" alt="" />
  </a>
  <!-- data-pswp-src with custom URL in href -->
  <a href="https://unsplash.com"
    data-pswp-src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/3/img-2500.jpg"
    data-pswp-width="2500"
    data-pswp-height="1666"
    target="_blank">
    <img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/3/img-200.jpg" alt="" />
  </a>
  <!-- wrapped with any element: -->
  <div>
    <a href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/6/img-2500.jpg"
      data-pswp-width="2500"
      data-pswp-height="1667"
      target="_blank">
      <img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/6/img-200.jpg" alt="" />
    </a>
  </div>
</div>

---

## [Spotlight JS](https://nextapps-de.github.io/spotlight/)

<!-- Group 1 -->
<div class="spotlight-group">
    <a class="spotlight" href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/1/img-2500.jpg">
        <img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/1/img-200.jpg" />
    </a>
    <a class="spotlight" href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/2/img-2500.jpg">
        <img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/2/img-200.jpg" />
    </a>
    <a class="spotlight" href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/3/img-2500.jpg">
        <img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/3/img-200.jpg" />
    </a>
</div>
<!-- Group 2 -->
<div class="spotlight-group">
    <a class="spotlight" href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/4/img-2500.jpg">
        <img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/4/img-200.jpg" />
    </a>
    <a class="spotlight" href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/5/img-2500.jpg">
        <img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/5/img-200.jpg" />
    </a>
    <a class="spotlight" href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/6/img-2500.jpg">
        <img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/6/img-200.jpg" />
    </a>
</div>

---

## [Venobox](https://veno.es/venobox/)

<a class="venobox" data-gall="myGallery" href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/1/img-2500.jpg"><img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/1/img-200.jpg" /></a>
<a class="venobox" data-gall="myGallery" href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/2/img-2500.jpg"><img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/2/img-200.jpg" /></a>
<a class="venobox" data-gall="myGallery" href="https://cdn.photoswipe.com/photoswipe-demo-images/photos/3/img-2500.jpg"><img src="https://cdn.photoswipe.com/photoswipe-demo-images/photos/3/img-200.jpg" /></a>
