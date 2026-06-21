---
title: "Lower-Limb Pediatric Exoskeleton: Shirley Ryan AbilityLab"
date: 2026-03-28
math: true
summary: "Assistive pediatric exoskeleton research for gait rehabilitation in collaboration with Shirley Ryan AbilityLab."
tags: ["Exoskeleton", "Rehabilitation", "Control", "ROS 2", "Python", "Mechanical Design", "Biomechanics"]
weight: 3
cover:
    image: "/videos/exoskeleton.webm"
    alt: "Lower-Limb Exoskeleton"
    relative: true
    hiddenInSingle: true
---

## Project Overview

This project is part of ongoing assistive robotics research at **Shirley Ryan AbilityLab**, focused on developing a lower-limb pediatric exoskeleton for gait rehabilitation. My contributions span two areas: **feedforward motor control** to reduce resistive drag from back-EMF and rotor inertia, and **mechanical design** of the hip joint assembly to accommodate pediatric anatomy and natural gait kinematics.

**Hardware:** AKE60-8 KV80 brushless motor (CubeMars) · Novanta Everest CORE servo drive

---

## Control Architecture

The exoskeleton runs a layered software stack bridging high-level ROS 2 controllers down to the Novanta Everest servo drive.

The **ROS 2 layer** hosts a library of controllers — including force-feedback, oscillatory, and state-machine walking modes — that output desired torque commands. These commands travel down through a robot model layer (which converts degrees to encoder ticks) and a hardware abstraction layer over SPI to reach the drive.

The **Novanta Everest CORE** handles low-level closed-loop control. Internally it runs a cascaded PID pipeline: position → velocity → torque → current → voltage. A state machine safety check gates all setpoints before they reach the MUX and setpoint manager, which supports linear, trapezoidal, S-curve, and PVT motion profiles.

Communication between the ROS 2 host and the microcontroller uses UDP sockets (`udp2ROS.py` publishes `robot_state` and `ctrl_state` topics; `ROS2udp.py` builds 20-float control messages and 4-float robot messages). TCP is used separately for tuning and calibration commands.

---

## Back-EMF & Inertia Feedforward

### Problem

At the drive level, the AKE60-8 motor exhibits two sources of resistive drag that fight the patient's natural leg motion:

1. **Back-EMF drag** — as the rotor spins, it generates a voltage opposing motion (Ke·ω). Even when the drive commands zero current, manual rotation of the shaft induces a back-EMF current that by Lenz's law creates a braking torque.
2. **Rotational inertia** — at direction reversals, the rotor resists acceleration changes, creating a torque deficit during the transition.

Both effects are especially problematic in a transparency-mode exoskeleton, where the goal is to make the device feel as if it isn't there.

### Approach

A feedforward term is injected at the torque junction every control tick, computed from the drive's real-time velocity and acceleration feedback:

$$
\tau_{ff} = (K_{bemf} \cdot \omega) + (K_{acc} \cdot \alpha)
$$

$$
\tau_{cmd} = \tau_{des} + \tau_{ff}
$$

| Gain | Value | Units |
|---|---|---|
| `BACKEMF_FF_GAIN` | 1.0 | A/(rev/s) |
| `ACC_FF_GAIN` | 0.1 | A/(rev/s²) |

```python
def _compute_feedforward(self):
    if self._backemf_ff_enabled:
        self._tau_backemf_ff = self.BACKEMF_FF_GAIN * self.servo.velocity
        self._tau_acc_ff     = self.ACC_FF_GAIN     * self.servo.acceleration
    else:
        self._tau_backemf_ff = 0.0
        self._tau_acc_ff     = 0.0

    self._tau_commanded = self._tau_external + self._tau_backemf_ff + self._tau_acc_ff
    self.servo.set_tau_offset(self._tau_commanded)
```

---

## Experimental Results

Three phases were tested:

- **Phase 1** — No feedforward (baseline)
- **Phase 2** — Back-EMF feedforward only
- **Phase 3** — Back-EMF + acceleration feedforward

**Test 1: Manual rotation (transparency mode)**

With position/velocity gains zeroed and FF off, rotating the shaft by hand produces back-EMF braking current. Adding the feedforward term cancels the induced current, making the joint feel free to rotate.

- Velocity–current slope: **−0.0066 → −0.0031 A/(deg/s)** (Phase 1 → Phase 2)

**Test 2: Sinusoidal position trajectory**

With position and velocity feedback active, a sinusoidal trajectory was commanded. The feedforward reduces the current the drive must supply to sustain the commanded motion.

- Velocity–current slope reduction: **16.3%** (0.0053 → 0.0045 A/(deg/s))
- Acceleration–current slope reduction: **14.0%** (0.00046 → 0.00040 A/(deg/s²))

Across both tests, mean current draw decreased monotonically from Phase 1 → Phase 2 → Phase 3, confirming that back-EMF and inertia compensation together reduce the resistive load felt by the patient.

---

## Mechanical Design

The hip joint assembly was redesigned around four pediatric-specific requirements:

- **Low inertia** — minimizing distal mass to reduce metabolic cost and improve transparency
- **Hip abduction/adduction freedom** — ±30° range of motion to accommodate natural pediatric gait
- **Discrete link length adjustment** — quick-change slot mechanism to fit different leg lengths without tools
- **Self-aligning mechanism** — passive compliance to prevent kinematic misalignment between the robot joint axis and the patient's anatomical hip axis

### Key Design Features

**90° Torsional Spring (Hip Ab/Ad Joint)**
A torsional spring allows the lateral linkage to passively follow the patient's hip during abduction and adduction while returning to neutral alignment when unloaded. A **30° hard stop** limits maximum abduction to protect the joint and the patient.

**Discrete Link Length Adjustment**
A slotted bracket with indexed hole positions enables rapid length adjustment across the femoral link. This allows the exoskeleton to be reconfigured between patients without requiring custom hardware.

**Tech Stack:** SolidWorks · Python · ROS 2 · Novanta Everest SDK · CubeMars AKE60-8
