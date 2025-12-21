---
title: "PenPal: VLM-Guided Writing Robot"
date: 2025-12-12
draft: false
tags: ["ROS 2", "Python", "Motion planning", "Franka Emika"]
summary: "Engineering a 7-DOF Franka Emika robot arm to answer questions by writing on hand-held whiteboards."
weight: 1
cover:
    image: "images/penpal-cover.jpg"
    alt: "PenPal Robot"
    relative: false
    hidden: true
---

## The Demo
> `{{< youtube xrn48SnPuLs >}}`

## Project Overview
**Role:** Motion Planning & Control

**Tech Stack:** ROS 2 (Kilted), Python, MoveIt 2, Franka Emika Panda

PenPal is a vision-guided robotic system that reads handwritten questions from a whiteboard and physically writes answers back in real-time.

I engineered a hybrid control framework. While Cartesian planning was used for the actual writing strokes, I developed a custom constraint-based motion planner to handle the complex approach and retreat maneuvers required to grab the pen and approach the board.

## Key Contributions

### 1. Dynamic Safety & Collision Management
A critical challenge was allowing the robot to touch the whiteboard (collision) while writing, but preventing dangerous collisions elsewhere. I engineered a dynamic safety system that modifies the robot's reflex thresholds in real-time.

* **"Orange Zone" Logic:** Implemented a service client (`SetFullCollisionBehavior`) that dynamically raises the torque/force thresholds just before the pen touches the board and lowers them back to sensitive levels during free-space motion.
* **Integration Testing:** Developed a standalone integration test node to visualize and validate complex trajectories (Circles, Squares, Arrows) in Rviz before executing them on hardware.

### 2. Constraint-Based Motion Planning
Instead of relying on standard `move_group.go()` calls, I implemented a custom `MotionPlanRequest` generator. This allowed for precise control over the planning volume and solved the issue of the robot getting stuck in local minima when moving near the whiteboard surface.
* **Tolerance Handling:** Defined strict `PositionConstraint` and `OrientationConstraint` bounding boxes (SolidPrimitives) to ensure the end-effector (TCP) remained perpendicular to the writing surface during approach.
* **Safety Scaling:** Dynamically adjusted `max_velocity_scaling_factor` (0.1) and `max_acceleration_scaling_factor` (0.1) during the delicate approach phase to prevent overshoot.


## Code Snippet: Dynamic Collision Tuning
This snippet from my integration test node demonstrates how I dynamically adjusted safety thresholds for the "Writing Phase" to prevent false-positive safety stops due to friction.

```python
async def integration_test(node, ctl):
    """
    Orchestrates the writing sequence with dynamic safety parameters.
    """
    # 1. Define High Thresholds (For Writing Contact)
    # Allows higher torque resistance (friction) without triggering safety stop
    high_req = SetFullCollisionBehavior.Request()
    high_req.lower_torque_thresholds_nominal = [20.0] * 7
    high_req.upper_torque_thresholds_nominal = [50.0] * 7 # Increased for contact
    
    node.get_logger().info('Setting High Thresholds for Writing Phase')
    await collision_service.call_async(high_req)
    await asyncio.sleep(2.0)

    # 2. Execute Trajectory
    for traj in seq:
        logger.info(f'Executing trajectory {traj.label}...')
        await ctl.execute_trajectory(traj, speed=0.01)