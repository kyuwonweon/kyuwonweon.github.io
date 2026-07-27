---
title: "Packaging Automation: Siemens Healthineers"
date: 2026-07-24
math: false
summary: "A servo-driven robotic folding station that automates shipping-box assembly on a manufacturing packaging line, reclaiming 41 hours/month of manual labor."
tags: ["Automation", "Manufacturing", "Mechanical Design", "Embedded Systems", "MicroPython", "Mechatronics"]
weight: 1
cover:
    image: "/videos/automation/prototype_video.mp4"
    alt: "Packaging Automation Folding Station"
    relative: true
    hiddenInSingle: true
---

## Project Overview

This project was developed during my manufacturing internship at **Siemens Healthineers**, where a packaging line was folding shipping boxes entirely by hand: roughly 10 seconds per box across 15,000 units a month, tying up about **41 hours/month** in repetitive labor and repetitive-motion strain. The goal was to design and prototype a robotic station that could fold the box automatically and free that time for higher-value work.

<video width="100%" autoplay loop muted playsinline controls>
    <source src="/videos/automation/prototype_video.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

The solution follows a simple **Feed → Fold → Drop** pipeline: boxes are loaded from a stack, the station erects the box and folds the bottom flaps into a sealed shape, and the finished box is moved off to a bin.

---

## Mechanical Design

The folding station is built around a small set of key components:


<img src="/images/boxfoldingcad.png" alt="Packaging Automation Folding Station CAD" width="100%">

1. **Frame** — 2020 T-slot aluminum rails on a 3D-printed base with alignment features
2. **Servo motor** — drives the folding actuation
3. **Rack & pinion** — converts servo rotation into the linear stroke needed for bottom folding
4. **Vacuum pump + suction cup** — holds the box securely in position during folding
5. **Base enclosure** — houses the electronics and mounting hardware

Three actuation mechanisms were paired to the motion each fold step demands: a **linkage** drives box erection and bottom-folding, a **rack & pinion** executes the side-tab tuck, and a **rotary** actuator completes the tuck-in tab fold.

<video width="100%" autoplay loop muted playsinline controls>
    <source src="/videos/automation/squaring.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

<video width="100%" autoplay loop muted playsinline controls>
    <source src="/videos/automation/tuck_in.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

<video width="100%" autoplay loop muted playsinline controls>
    <source src="/videos/automation/tucktab.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

---

## Control System
<img src="/images/automationschematic.png" alt="Packaging Automation Folding Station CAD" width="100%">

The station runs on **MicroPython** on a Raspberry Pi Pico 2. Servos are driven over I2C through a PCA9685 16-channel PWM driver, with the pump and solenoid valve switched through an optocoupled relay so the Pico's GPIO never carries load current directly. Power is split across two isolated rails — 6V for the servos and driver, 12V for the pump and valve — sharing a common ground, with actuation sequenced rather than simultaneous to keep peak draw within each rail's rating.

A small command interface (`Run [stage]`, `Set <servo> <value>`, `Pump on/off`) drives an 8-step execution sequence per box:

1. Form the square
2. Fold side tabs in
3. Half bottom fold (60°)
4. Fold tuck tab in
5. Pull side-tab fixtures out
6. Second bottom fold to seat the tuck tab (80°)
7. Pull the tuck flap out
8. Final full bottom fold

The full sequence completes in about **3 seconds per box**.

---

## Future Work for the remainder of Internship

* **Feeding mechanism** — prototype a servo crank-slider to automate loading boxes into the station
* **Drop mechanism** — automate moving the finished box off to the bin
* **Industrialization** — move the design from a rapid prototype to a mechanism reliable enough for continuous use on an industrial manufacturing floor

<video width="100%" autoplay loop muted playsinline controls>
    <source src="/videos/automation/futurework.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>
