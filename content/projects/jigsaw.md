---
title: "Jigsaw Puzzle Solving Robot"
date: 2026-05-20
math: true
summary: "Autonomous robot that detects, picks, and places jigsaw puzzle pieces using computer vision and manipulation."
tags: ["Robotics", "Computer Vision", "Manipulation", "Python", "ROS 2"]
weight: 5
cover:
    image: "/videos/jigsaw_cover.mp4"
    alt: "Jigsaw Puzzle Solving Robot"
    relative: true
    hiddenInSingle: true
---

## Project Overview

This project implements a **Jigsaw Puzzle Solving Robot** that combines computer vision, robotic manipulation, and motion planning to identify, pick, and place puzzle pieces into their correct locations.

<video width="100%" autoplay loop muted playsinline controls>
    <source src="/videos/puzzle solver.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

[![View Code on GitHub](https://img.shields.io/badge/View%20Code-GitHub-181717?style=for-the-badge&logo=github)](https://github.com/kyuwonweon/jigsaw-puzzle-solver)

**The Core Problem:** Solving a jigsaw puzzle requires both perception and manipulation. A robot must first determine *which* puzzle piece it is observing, then physically grasp the piece and place it at the correct location. Unlike structured industrial parts, puzzle pieces often have similar shapes, arbitrary orientations, and cluttered arrangements, making reliable identification challenging.

This project was developed collaboratively with **Halley Zhong**. I led development of the **computer vision pipeline**, while Halley designed the **robotic manipulation system**, including the vacuum end-effector, hardware integration, and pick-and-place execution.

Instead of relying on labeled object positions, the system performs real-time puzzle piece recognition using template generation, contour extraction, feature matching, and shape analysis from a live Intel RealSense camera feed. The resulting piece identity is then used to guide robotic pick-and-place operations.

**Tech Stack:** Python, OpenCV, ROS 2, Intel RealSense, Interbotix PX100 Robot Arm, NumPy.

---

## Vision System Architecture

### 1. Vision-Based Template Generation

A template generation pipeline creates a database of puzzle piece references directly from camera observations.

* **Inputs:** RealSense RGB images of individual puzzle pieces.
* **Processing:**

  * HSV color segmentation to isolate puzzle pieces from the workspace.
  * Morphological filtering to remove noise and fill segmentation gaps.
  * Contour extraction to identify valid puzzle-piece boundaries.
  * Alpha-mask generation to create transparent puzzle-piece templates.
* **Outputs:** Numbered puzzle-piece template images stored for future matching.

This process eliminates the need for manually cropped reference images and enables rapid creation of puzzle datasets.

### 2. Real-Time Piece Recognition

The perception pipeline continuously processes incoming camera frames and identifies visible puzzle pieces.

* **Inputs:** Live RGB stream from Intel RealSense.
* **Processing:**
 
  1. HSV thresholding for foreground segmentation.
  2. Contour filtering based on area and geometric constraints.
  3. Feature extraction using SIFT descriptors.
  4. Template matching using brute-force nearest-neighbor matching.
  5. Ratio-test filtering to reject ambiguous correspondences.
* **Outputs:** Puzzle-piece IDs and bounding-box locations.

The system is robust to moderate rotation, scale variation, and illumination changes due to the use of feature-based matching rather than direct pixel comparison.


## Results

The completed system successfully identifies puzzle pieces from a live camera feed and enables robotic manipulation using those classifications. One major limitation of this project was human's intervention in manually pushing in the puzzle piece due to the positional accuracy of 8mm and repeatability of 5mm of the PincherX robot.

**Computer Vision Performance**

* Real-time puzzle-piece detection and classification.
* Robust matching under varying orientations.
* Automatic template generation from camera observations.
* Reliable recognition of all 12 puzzle pieces.


## Future Improvements

* **Hardware Upgrade** Switch from Pincher X to Franka Emika for superior positional accuracy and more workspace. 
* **Puzzle Complexity** Test with puzzles featuring more pieces, similar traits, and more challenging geometric designs.
* **Autonomous Learning** Transition from template matching to self-learned assembly logic, allowing the robot to solve puzzles independently.
