---
title: "Collision Prevention Package: CBF-QP"
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
<div class="video-showcase vertical">
  <div class="video-wrapper">
    <iframe src="https://www.youtube.com/embed/CJ427VDBa4E?autoplay=1&mute=1&loop=1&playlist=CJ427VDBa4E" frameborder="0" allow="autoplay; fullscreen; encrypted-media" allowfullscreen></iframe>
  </div>
  <div class="video-wrapper">
    <iframe src="https://www.youtube.com/embed/qM3LcE7e87Y?autoplay=1&mute=1&loop=1&playlist=qM3LcE7e87Y" frameborder="0" allow="autoplay; fullscreen; encrypted-media" allowfullscreen></iframe>
  </div>
</div>

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

**2. Energy-Aware Control Barrier Function (CBF)**
To ensure the robot can safely dissipate its momentum before an impact, we define a dynamic barrier function $B(q, \dot{q})$ that couples the distance to the robot's Kinetic Energy ($KE = \frac{1}{2}\dot{q}^T M(q) \dot{q}$), where $M(q)$ is the joint-space mass matrix:
$$B(q, \dot{q}) = \gamma h(q) - KE \geq 0$$

To guarantee forward invariance of the safe set (the robot always has enough distance to brake), we enforce the derivative inequality limit:
$$\dot{B}(q, \dot{q}) \geq -\alpha B(q, \dot{q})$$

Simultaneously, we apply a 2nd-order kinematic CBF to constrain the approach acceleration using the geometric Jacobian $J$:
$$J\ddot{q} \geq -k_p h(q) - k_d \dot{h}(q)$$

**3. Acceleration Quadratic Program (QP)**
Instead of solving blindly for velocity, we resolve a desired acceleration $\ddot{q}_{des} = K_p(\dot{q}_{des} - \dot{q}_{safe})$ and use a Dense QP Solver to find the optimal safe acceleration ($\ddot{q}^*$) that tracks the user's command without violating the dynamic energy limits:
$$\min_{\ddot{q}} \frac{1}{2} || \ddot{q} - \ddot{q}_{des} ||^2$$
$$\text{s.t.} \quad A_{cbf} \ddot{q} \leq b_{cbf} \quad \text{(Energy \& Kinematic Bounds)}$$
$$\quad \quad \ddot{q}_{min} \leq \ddot{q} \leq \ddot{q}_{max} \quad \text{(Hardware Slew-Rate Limits)}$$

The resulting safe acceleration is then numerically integrated ($\dot{q}_{safe} = \dot{q}_{safe} + \ddot{q}^* \Delta t$) and published to the physical hardware controllers.

## System Architecture (Nodes)
The system is built on a modular ROS 2 Kilted architecture:

* **`joy_node` (C++):** Interacts with the hardware driver to read raw Xbox controller states (axes/buttons).
* **`teleop_node` (Python):** Converts raw joystick inputs into a Twist or Joint Velocity command. It handles mode switching (e.g., enabling "Turbo" speed or resetting the robot).
* **`safety_node` (C++):** The brain of the operation.
    * **Subscribes** to `/joint_states` (Real Robot) and `/cmd_vel` (Teleop).
    * **Computes** the CBF constraints using Pinocchio.
    * **Solves** the QP using ProxQP.
    * **Publishes** the safe command to the robot controller.
* **`robot_state_publisher`:** Broadcasts the URDF and TF tree, visualizing both the "Real Robot" and the "Ghost" (Commanded) robot.
* **`rviz2`:** Visualizes the geometric primitives (Capsules/Spheres) and the safety intervention in real-time.

## Key Challenges & Resolutions

**1. Optimization Latency**
* **Challenge:** Initial debug builds resulted in loop times >3.5ms, causing control lag.
* **Resolution:** Migrated to `Release` builds with `-O3` flags, reducing QP solve time to **<0.5ms** for a robust 1kHz loop.

**2. Visual "Ghosting"**
* **Challenge:** Safety geometry (capsules) trailed behind the robot during fast motions due to timestamp mismatches.
* **Resolution:** Synchronized visualization markers with the joint state update loop, ensuring 1:1 alignment between the physical robot and safety bubbles.

**3. Geometric Approximation**
* **Challenge:** Full mesh collision checking is too slow for 1kHz control.
* **Resolution:** Implemented a **Capsule-to-Point** distance algorithm. This reduces complex mesh interactions to simple algebraic line-segment math, drastically lowering computational cost.


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