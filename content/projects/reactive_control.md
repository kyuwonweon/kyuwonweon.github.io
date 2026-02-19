---
title: "Reactive Control & Obstacle Avoidance"
date: 2026-01-05
draft: false
math: true
summary: "Real-time kinematic control with Artificial Potential Fields"
tags: ["Robotics", "Control Theory", "Artificial Potential Fields", "Python", "MuJoCo"]
weight: 3
cover:
    image: "images/apf.mp4"
    alt: "APF Demo Video"
    relative: false
    hiddenInSingle: true
---
## The Demo
{{<youtube ni3_9UhF_6E>}}
[![View Code on GitHub](https://img.shields.io/badge/View%20Code-GitHub-181717?style=for-the-badge&logo=github)](https://github.com/kyuwonweon/franka_reflex)

## Project Overview

This project implements a **Reactive Controller** for the 7-DOF Franka Emika Panda robot. This system uses **Artificial Potential Fields (APF)** to calculate control commands made from scratch in real-time (100Hz). This approach allows the robot to instantly adapt to dynamic environments, dodging obstacles while reaching for the target position.

## Background
1.  **The Goal:** The target position naturally pulls the robot's end effector.
2.  **The Obstacles:** Obstacles generate a repulsive "energy hill" that pushes the robot away. This force activates only when the robot breaches a safety threshold. It grows exponentially as the robot gets closer to an obstacle, creating a virtual bumper. 
3.  **The Result:** The robot acts as a particle moving through this energy landscape, naturally flowing around obstacles to reach the goal without explicit path planning.


## Challenge & Resolution

**The Challenge: Instability and Oscillation**
A major difficulty in implementing APF is tuning the gain parameters (zeta and eta).
* High gains allowed the robot to overpower the repulsion field and reach the target, but caused violent high-frequency oscillation when forces conflicted.
* Low gains resulted in smooth motion but left the robot "stuck" in local minima, unable to push through the safety threshold of the obstacle or overcome its own posture constraints.

**The Resolution: Global Damping and Gain Scheduling**
To stabilize the system without sacrificing strength:
1.  **Global Damping:** Implemented a velocity scaling factor. This allowed for high stiffness gains while dissipating the kinetic energy that caused vibrations.
2.  **Safety Clamping:** Capped the maximum repulsive force to prevent mathematical singularities when the robot operated extremely close to obstacle surfaces.
3.  **Dynamic Braking:** Designed a non-linear braking profile that smoothly decelerated the robot within the final 2cm of the target.

## Simulation Results

The simulation, built in **MuJoCo** and wrapped in **ROS 2**, demonstrates the robot successfully navigating a "glancing blow" scenario. The robot curves around a central obstacle to reach a target, adjusting its path instantaneously as the obstacle pushes against the repulsion field.


## Key Learning Outcomes

* **Artificial Potential Fields (APF):** Developed a strong intuition for designing reactive control. Learned to shape robot behavior mathematically by constructing virtual energy landscapes—defining attraction basins for goals and exponential repulsion fields for safety.
* **Tuning Non-Linear Systems:** Explored the trade-off between **Stiffness** (Accuracy) and **Stability** (Smoothness). Discovered that high-gain systems require robust "shock absorbers" (global damping) to prevent high-frequency oscillations when conflicting forces meet.


## Citation
https://www.cs.cmu.edu/~motionplanning/lecture/Chap4-Potential-Field_howie.pdf
