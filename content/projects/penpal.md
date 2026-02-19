---
title: "PenPal: VLM-Guided Writing Robot"
date: 2025-12-12
draft: false
tags: ["ROS 2", "Python", "Motion planning", "Franka Emika", "MoveIt 2"]
summary: "7-DOF Franka answers questions by writing on hand-held whiteboards"
weight: 2
cover:
    image: "images/penpal-cover.jpg"
    alt: "PenPal Robot"
    relative: false
    hiddenInSingle: true
---

## The Demo
{{<youtube xrn48SnPuLs>}}
[![View Code on GitHub](https://img.shields.io/badge/View%20Code-GitHub-181717?style=for-the-badge&logo=github)](https://github.com/kyuwonweon/penpal)

## Project Overview
**Role:** Motion Planning & Control  
**Tech Stack:** ROS 2 (Kilted), Python, MoveIt 2, Franka Emika Panda

PenPal is a vision-guided robotic system that bridges the gap between Large Language Models (LLMs) and physical actuation. It reads handwritten questions, generate an answer and physically writes answers in real-time. My role focused on the **hardware interface layer**: ensuring the robot could autonomously grasp a marker, safely approach a dynamic surface, and execute precise writing strokes without triggering safety stops.

## Key Technical Contributions

### 1. Hybrid Motion Planning Architecture
I engineered a two-stage control framework to handle the distinct requirements of "free space" vs. "contact" motion:
I utilized sampling-based planners (OMPL) for global tasks like "Pick Up Pen" and "Return to Home," navigating complex joint configurations to avoid self-collisions.
* **Fine Motion (Cartesian):** Implemented a custom Cartesian interpolator for the actual writing phase. This ensured linearity and consistent motion during strokes, which standard planners often fail to guarantee.

### 2. Integration Testing & Validation
To prove the system could handle physical contact before full deployment, I built a **standalone integration test suite**.
* Developed a parameterized test node that commanded the robot to draw geometric primitives (circles, squares, arrows) in the air and on surfaces.
* This validated that our "Writing Mode" logic held up under sustained contact, proving the system was ready for the variability of VLM-generated text.

### 3. Dynamic Safety ("Orange Zone" Logic)
Writing requires *collision*, but robots are designed to *stop* on collision.
* I implemented a dynamic client that interacts with the Franka reflex controller.
* **The Logic:** The system monitors the proximity to the whiteboard. Millimeters before contact, it creates an "Orange Zone" where torque thresholds are raised. Instantly upon retreat, thresholds drop back to nominal levels, ensuring human safety is never compromised during rapid movements.

## Challenge & Solution

### The "Wrist vs. Tip" TF Mismatch
**The Problem:** During early tests, the robot would approach the whiteboard at unpredictable, "weird" angles—often tilting the marker away from the surface. This occurred because the motion planner was solving for the robot's **Flange (Wrist)** frame, ignoring the orientation of the marker tip.

**The Solution:**
* Defined a precise static transform from the wrist to a end effector frame.
* Updated the MoveIt planning group to solve specifically for `marker_tip` orientation constraints.
* **Result:** The robot decoupled the wrist rotation from the writing task, ensuring the marker always touched the board perfectly perpendicular ($90^{\circ}$), regardless of the arm's configuration.