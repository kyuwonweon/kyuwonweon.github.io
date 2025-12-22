---
title: "Jack in a Cup: Hybrid Dynamics Simulation"
date: 2025-12-15
draft: false
weight: 2
tags: ["Python", "SymPy", "Computational Dynamics", "Simulation"]
summary: "Simulating rigid body collisions and Lagrangian dynamics for a multi-body constrained system using symbolic derivation."
cover:
    image: "images/jack-cover.jpg"
    alt: "Jack in a Cup Simulation"
    relative: false
    hiddenInSingle: true
---

## Simulation Demo
> `{{< youtube uo32M8EQnbg >}}`

Link to the full code [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/11C2A4lP5c3aaAlLjqFaonxYHd9u5vhHk?usp=sharing)

## Project Overview
**Tech Stack:** Python, SymPy, NumPy, Plotly

This project models the dynamics of a Jack (a cross-shaped rigid body) bouncing inside a moving cup (square box). The system **combines continuous** differential equations of motion with discrete impact events.

## Technical Implementation

### 1. Symbolic Lagrangian Dynamics
I used **SymPy** to mathematically derive the Equations of Motion (EOM) from scratch using the Euler-Lagrange method.
* **SE(3) Formulation:** Modeled the 6-DOF configuration space using transformation matrices for the Cup frame and Jack frame.
* **Symbolic Derivation:** Automatically calculated the Mass Matrix and Coriolis/Gravity terms by taking the Jacobian of the Lagrangian with respect to the state vectors.

### 2. Impact & Collision Handling
The core challenge was handling the **discrete jumps** in velocity when the Jack hits the Cup walls.
* **Geometric Constraints:** Defined boundary conditions for all 4 walls of the cup relative to the 4 tips of the jack.
* **Impulse Solver:** Implemented an impact update law that solves for the instantaneous change in velocity and the constraint force by solving the system of linear equations derived from the impact Hamiltonian.