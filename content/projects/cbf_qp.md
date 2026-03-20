---
title: "Robot Collision Prevention Package: CBF-QP"
date: 2026-03-11
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

## Project Overview
<iframe width="100%" height="450" src="https://www.youtube.com/embed/68CW9JTyOSY?autoplay=1&mute=1&loop=1&playlist=68CW9JTyOSY" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
This project implements a **Real-Time Dynamic Collision Prevention System** for the 7-DOF Franka Emika Panda. 

**Evolution from Previous Work:**
While my [Reactive Control project]({{< relref "reactive_control.md" >}}) used Artificial Potential Fields (APF) to create soft repulsive forces that could lead to local minima or oscillations, this system upgrades to **Control Barrier Functions (CBF)**. This approach treats safety as a **hard mathematical constraint**, guaranteeing collision freedom without altering the robot's path unless absolutely necessary.

**Tech Stack:** ROS 2 Kilted, Python (Launch/Teleop), C++ (Solver), Pinocchio (Kinematics), ProxQP.

[![View Code on GitHub](https://img.shields.io/badge/View%20Code-GitHub-181717?style=for-the-badge&logo=github)](https://github.com/kyuwonweon/cbf_safety_layer)


<div class="video-showcase vertical">
  <div class="video-wrapper">
    <iframe src="https://www.youtube.com/embed/CJ427VDBa4E?autoplay=1&mute=1&loop=1&playlist=CJ427VDBa4E" frameborder="0" allow="autoplay; fullscreen; encrypted-media" allowfullscreen></iframe>
  </div>
  <div class="video-wrapper">
    <iframe src="https://www.youtube.com/embed/qM3LcE7e87Y?autoplay=1&mute=1&loop=1&playlist=SAKC8gtJvGA" frameborder="0" allow="autoplay; fullscreen; encrypted-media" allowfullscreen></iframe>
  </div>
</div>

Two robotic arms moving in commanded velocities with and without my collision prevention layer

## System Architecture

### Core Node: `/safety_node_cpp`
* **Inputs (Intention & State):** Subscribes to `/joint_states_source` (1kHz hardware states) and `/teleop_vel` (the desired user or global planner velocity commands). It also listens to `/shared_obstacle` for dynamic hazard tracking.
* **Internal Processing Pipeline:**
    1. **Kinematic Update:** Uses **Pinocchio** to compute forward kinematics, frame placements, joint Jacobians, and the Mass Matrix via CRBA.
    2. **Geometric Evaluation:** Models the robot using 8 overlapping capsules. Calculates point-to-segment and segment-to-segment distances algebraically to completely bypass the latency of heavy mesh-collision libraries.
    3. **Constraint Formulation:** Constructs the dense matrices ($C$, $l$, $u$) for Floor, Dynamic Obstacles, Inter-Robot collisions, and Self-Collisions.
    4. **Optimization:** Solves the constrained Quadratic Program using **ProxQP**.
* **Outputs (Action):** Publishes the mathematically verified safe velocity commands to the hardware controller, alongside RViz2 visualization markers.

![My Robot Description](/images/cbf_blockdiagram.svg)

## Mathematical Foundation

Unlike standard kinematic safety filters, this implementation treats safety as a dynamic constraint. It accounts for the robot's physical inertia and kinetic energy to guarantee it always has the stopping power required to avoid an impact.

**1. The Energy-Aware Barrier Function ($h(x)$)**
Let $h(q)$ be the geometric distance between the robot capsule and a hazard. We define a dynamic barrier function $B(q, \dot{q})$ that couples this distance to the robot's kinetic energy ($E_k$):
$$E_k = \frac{1}{2}\dot{q}^T M(q) \dot{q}$$
$$B(q, \dot{q}) = \gamma \max(h(q), -0.05) - E_k$$

To ensure the robot does not enter an unavoidable collision state, we enforce the derivative inequality $\dot{B} \ge -\alpha B$, which bounds the allowable kinetic energy relative to the distance to the obstacle.

**2. High-Order Kinematic Constraints**
Because the robot is commanded at the acceleration level internally, we apply a PD-style High-Order CBF (HOCBF) to constrain the approach rate. Using the geometric Jacobian $J(q)$, the distance constraint is bounded by:
$$l_{kin} = -k_p h(q) - k_d \dot{h}(q)$$
$$l_{kin} \le J(q) \ddot{q} \le \infty$$

**3. The Dense Quadratic Program (QP)**
The solver maps the user's desired velocity into a desired acceleration $\ddot{q}_{des} = k_p(\dot{q}_{des} - \dot{q}_{safe})$ and finds the optimal safe acceleration ($\ddot{q}^*$) using ProxQP. 

$$\min_{\ddot{q}} \frac{1}{2} || \ddot{q} - \ddot{q}_{des} ||^2$$

**Subject to:**
* $L_{cbf} \le C_{cbf} \ddot{q} \le U_{cbf}$  (Energy and Geometry Bounds)
* $a_{min} \le \ddot{q} \le a_{max}$  (Hardware Acceleration Limits)
* $\dot{q}_{min} \le \dot{q} \le \dot{q}_{max}$  (Hardware Velocity Limits)
* $q_{min} \le q \le q_{max}$  (Hardware Joint Limits)

The optimal acceleration is integrated numerically ($v_{safe} = v_{safe} + \ddot{q}^* \Delta t$) and clamped to hardware limits before publishing.


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