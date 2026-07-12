---
title: "Autonomous Mapping & Navigation: EKF SLAM"
date: 2026-03-20
math: true
summary: "ROS 2 pipeline for SLAM using an Extended Kalman Filter and unknown data association"
tags: ["SLAM", "State Estimation", "EKF", "C++", "ROS 2", "TurtleBot3"]
weight: 2
cover:
    image: "/videos/slam_cover.mp4"
    alt: "EKF SLAM RViz Visualization"
    relative: true
    hiddenInSingle: true
---

## Project Overview

This project implements **Simultaneous Localization and Mapping (SLAM)** pipeline from scratch for a differential-drive robot. 

<video width="100%" autoplay loop muted playsinline controls>
    <source src="/videos/SLAM SIM VIDEO.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

**The Core Problem:** If a robot positioning is calculated solely based on odometry(counting its wheel rotations), microscopic wheel slips will cause its internal sense of location to drift exponentially over time. SLAM solves this by letting the robot use its sensors to build a map of its surroundings, which is used to correct its own drifting location.

Instead of relying on perfect simulation data, this system processes raw, noisy LiDAR point clouds, extracts geometric features, and uses an **Extended Kalman Filter (EKF) with Unknown Data Association** to statistically fuse sensor data and odometry into a highly accurate single source of truth.

**Tech Stack:** ROS 2, C++, Armadillo (Matrix Math), Eigen3, TF2, RViz2.

[![View Code on GitHub](https://img.shields.io/badge/View%20Code-GitHub-181717?style=for-the-badge&logo=github)](https://github.com/ME495-Navigation/slam-kyuwonweon)

<div class="video-showcase vertical">
  </div>


## System Architecture

The pipeline is entirely custom-built and broken into three distinct layers:

### 1. Core Kinematics (`turtlelib`)
A standalone C++ geometry engine handling 2D rigid body transformations ($SE(2)$) and the forward/inverse kinematics of a differential drive robot.

### 2. Perception Pipeline (`landmarks`)
* **Inputs:** Subscribes to `/laser` for raw 2D LiDAR scans.
* **Processing:** 1. **Clustering:** Groups adjacent laser hits using a Euclidean distance threshold.
    2. **Circle Fitting:** Passes the clusters through an algebraic circle-fitting algorithm to estimate the exact center $(x, y)$ and radius $r$ of cylindrical obstacles in the environment.
* **Outputs:** Publishes clean `visualization_msgs::MarkerArray` cylinder coordinates, transforming a noisy room of dots into distinct mathematical landmarks.

### 3. State Estimation (`slam`)
This node tracks a massive $12 \times 12$ (or larger) state vector matrix containing both the robot's pose and the coordinates of every landmark it has ever seen. It continuously loops through Prediction (odometry) and Correction (LiDAR) to bound positional error.


## Mathematical Foundation

### 1. The Prediction Step (Motion Model)
When the robot moves, we update the state vector $q_{t}$ (containing robot pose $\theta, x, y$ and landmark coordinates) based on wheel encoder data. Because the robot moves in arcs, we use exact arc integration to update the pose:

$$q_{t} = f(q_{t-1}, u_t)$$

Simultaneously, we expand our uncertainty (Covariance Matrix, $\Sigma$) using the motion model Jacobian ($A$) and process noise ($Q$):
$$\Sigma_{t} = A \Sigma_{t-1} A^T + Q$$

### 2. Unknown Data Association
When the LiDAR detects a cylinder, the robot doesn't inherently know which cylinder it is looking at. We use the **Mahalanobis Distance** to mathematically determine if the detected obstacle is a new discovery or a previously mapped landmark, taking our current uncertainty into account:

$$d_m = (z - \hat{z})^T S^{-1} (z - \hat{z})$$

Where $z$ is the actual measurement, $\hat{z}$ is the predicted measurement, and $S$ is the innovation covariance. If $d_m$ falls below a strict threshold, we associate the measurement with an existing landmark. If not, we initialize a new landmark in the state vector.

### 3. The Correction Step (Measurement Model)
Once a landmark is identified, we calculate the difference between where the robot *thinks* the landmark should be and where the LiDAR *actually* sees it (the innovation). We compute the Kalman Gain ($K$) to decide how much to trust the sensor vs. the odometry:

$$K = \Sigma_t H^T (H \Sigma_t H^T + R)^{-1}$$

Finally, we pull the robot's estimated position back toward reality and shrink our uncertainty map:
$$q_{t} = q_{t} + K(z - \hat{z})$$
$$\Sigma_{t} = (I - KH)\Sigma_{t}$$


## Results

To prove the algorithm's effectiveness, the simulation visualizes three distinctly colored robots simultaneously:

* **Red (Ground Truth):** The absolute truth of where the robot is.

* **Blue (Odometry):** The odometry estimate based only on wheel spins.

* **Green (SLAM):** The EKF-corrected estimate.

**Performance:**
<video width="100%" autoplay loop muted playsinline controls>
    <source src="/videos/SLAM with real robot.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>
Over a sustained run in an unknown environment, pure wheel odometry accumulated severe drift—offsetting the robot by over $14$cm in X, $36$cm in Y, and approximately $40^\circ$ in heading. 

The EKF SLAM implementation successfully bounded this drift. By continuously correcting its internal map, the SLAM algorithm maintained near-perfect alignment with the ground truth, resulting in a final positional error of just **4 millimeters**.


<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.js" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/contrib/auto-render.min.js" crossorigin="anonymous"
    onload="renderMathInElement(document.body, {
      delimiters: [
        {left: '$$', right: '$$', display: true},
        {left: '$', right: '$', display: false},
        {left: '\\(', right: '\\)', display: false},
        {left: '\\[', right: '\\]', display: true}
      ],
      throwOnError : false
    });">
</script>