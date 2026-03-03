---
title: "Collision Preventation Package: CBF & QP"
date: 2026-02-12
math: true
summary: "Real-time safety filter with Control Barrier Functions."
tags: ["Robotics", "Control Theory", "Optimization", "C++", "Python", "ROS 2 Kilted", "Pinocchio"]
weight: 1
cover:
    image: "/images/cbf_qp.png"
    alt: "Dual Arm Safety Simulation"
    relative: true
    hiddenInSingle: true
---

## The Demo
{{<youtube NbHQxQ84qVs>}}
[![View Code on GitHub](https://img.shields.io/badge/View%20Code-GitHub-181717?style=for-the-badge&logo=github)](https://github.com/kyuwonweon/cbf_safety_layer)

## Project Overview
This project implements a **Real-Time Dynamic Collision Prevention System** for the 7-DOF Franka Emika Panda.

**Evolution from Previous Work:**
While my [Reactive Control project]({{< relref "reactive_control.md" >}}) used Artificial Potential Fields (APF) to create "soft" repulsive forces that could lead to local minima or oscillations, this system upgrades to **Control Barrier Functions (CBF)**. This approach treats safety as a **hard mathematical constraint**, guaranteeing collision freedom without altering the robot's path unless absolutely necessary.

**The Result:** A transparent safety filter that intercepts human or planner commands and modifies them via a Quadratic Program (QP) solver at **1kHz**, ensuring the robot remains within the safe set of states regardless of input aggression.

**Tech Stack:** ROS 2 Kilted, Python (Launch/Teleop), C++ (Solver), Pinocchio (Kinematics), ProxQP.

## Mathematical Foundation
The core of the safety layer is a constrained optimization problem.

**1. Distance Metric ($h(x)$)**
We approximate the robot as **Capsules** and obstacles as **Spheres**. The safety function $h(x)$ defines the distance to danger:
$$h(x) = d(C_{robot}, P_{obs}) - (r_{robot} + r_{obs} + \delta_{margin})$$
* $h(x) > 0$: Safe.
* $h(x) \le 0$: Collision.

**2. Control Barrier Function (CBF)**
To ensure forward invariance of the safe set (we never crash), we enforce the inequality:
$$\dot{h}(x) \geq -\alpha h(x)$$
Using the Jacobian $J$, we map this to joint velocities $\dot{q}$:
$$\frac{\partial h}{\partial q} \dot{q} \geq -\alpha h(x)$$

**3. Quadratic Program (QP)**
We solve for the optimal safe velocity $v^*$ that is closest to the user's desired velocity $v_{des}$:
$$\min_{v} \frac{1}{2} || v - v_{des} ||^2$$
$$\text{s.t.} \quad A_{cbf} v \leq b_{cbf}$$

## System Architecture (Nodes)
The system is built on a modular ROS 2 Kilted architecture:

* **`joy_node` (C++):** Interacts with the hardware driver to read raw Xbox controller states (axes/buttons).
* **`teleop_node` (Python):** Converts raw joystick inputs into a Twist or Joint Velocity command.
* **`safety_node` (C++):** The brain of the operation.
    * **Subscribes** to `/joint_states` (Real Robot) and `/cmd_vel` (Teleop).
    * **Computes** the CBF constraints using Pinocchio.
    * **Solves** the QP using ProxQP.
    * **Publishes** the safe command to the robot controller.
* **`robot_state_publisher`:** Broadcasts the URDF and TF tree, visualizing both the "Real Robot" and the "Ghost" (Commanded) robot.
* **`rviz2`:** Visualizes the geometric primitives (Capsules/Spheres) and the safety intervention in real-time.

## Key Challenges & Resolutions

**1. Visual "Ghosting"**
* **Challenge:** Safety geometry (capsules) trailed behind the robot during fast motions due to timestamp mismatches.
* **Resolution:** Synchronized visualization markers with the joint state update loop, ensuring 1:1 alignment between the physical robot and safety bubbles.

**2. Geometric Approximation**
* **Challenge:** Full mesh collision checking is too slow for 1kHz control.
* **Resolution:** Implemented a **Capsule-to-Point** distance algorithm. This reduces complex mesh interactions to simple algebraic line-segment math, drastically lowering computational cost.

## Future Work
* **Dual-Arm Collaboration:** Expanding the package functionality to support collision free path planning for two independent robots sharing a workspace.
* **Dynamic Obstacles:** Implementing peer-to-peer collision avoidance where each robot treats the other's links as moving obstacles.

## Citation
* **Reference:** Singletary, A., Kolathaya, S., & Ames, A. D. "Safety-Critical Kinematic Control of Robotic Systems." *arXiv preprint arXiv:2009.09100*, 2020. [PDF](https://arxiv.org/pdf/2009.09100.pdf)

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