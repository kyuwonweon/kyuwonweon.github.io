---
title: "PenPal: Robot Writing Assistant"
date: 2025-12-12
draft: false
tags: ["ROS 2", "Python", "Motion planning", "Franka Emika"]
summary: "Engineering a 7-DOF Franka Emika robot arm to answer questions by writing on hand-held whiteboards."
weight: 1
---

## The Demo
> *Insert a YouTube link here later like this:*
> `{{< youtube YOUR_VIDEO_ID >}}`

## Project Overview
**Role:** Motion Planning
**Tech Stack:** ROS 2 (Kilted), Python, MoveIt 2

PenPal uses a vision-guided robotic system to detect a whiteboard in the environment, read handwritten questions from the board using the Gemini vision-language model, generate concise answers, and physically writes responses back onto the board using a Franka Emika arm.

## Technical Challenges
### 1. Force Control & Friction
The hardest part was managing the friction of the marker against the whiteboard. We had to tune the impedance controller on the Franka Emika to apply constant pressure without triggering the safety stop.

### 2. Latency Integration
We reduced the round-trip latency between the LLM API and the robot controller by **200ms** using a custom Python node that pre-calculates the next character trajectory while the robot is drawing the previous one.

## Code Snippet
Here is the Python logic for our trajectory smoothing:

```python
def smooth_trajectory(waypoints):
    # Apply Spline interpolation to robot path
    return interpolated_path