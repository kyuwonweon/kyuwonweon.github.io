---
title: "Packaging Automation: Siemens Healthineers"
date: 2026-07-24
math: false
summary: "A servo-driven robotic folding station that automates shipping-box assembly on a manufacturing packaging line, reclaiming 41 hours/month of manual labor."
tags: ["Automation", "Manufacturing", "Mechanical Design", "Embedded Systems", "MicroPython", "Mechatronics"]
weight: 1
cover:
    image: "/videos/automation/cover.mp4"
    alt: "Packaging Automation Folding Station"
    relative: true
    hiddenInSingle: true
---

## Project Overview

This project was developed during my manufacturing internship at **Siemens Healthineers**, where a packaging line was folding shipping boxes entirely by hand: roughly 10 seconds per box across 15,000 units a month, tying up about **41 hours/month** in repetitive labor and repetitive-motion strain. The goal was to design and prototype a robotic station that could fold the box automatically and free that time for higher-value work.

<div class="video-bleed">
    <video class="folding-video" width="100%" autoplay loop muted playsinline controls>
        <source src="/videos/automation/folding.mp4" type="video/mp4">
        Your browser does not support the video tag.
    </video>
</div>

The solution follows a simple **Feed → Fold → Drop** pipeline: boxes are loaded from a stack, the station erects the box and folds the bottom flaps into a sealed shape, and the finished box is moved off to a bin.

[![View Code on GitHub](https://img.shields.io/badge/View%20Code-GitHub-181717?style=for-the-badge&logo=github)](https://github.com/kyuwonweon/package-automation/tree/main)

---

## Mechanical Design

<div style="display: flex; gap: 1rem;">
    <img src="/images/folding_prototype_cad_final.png" alt="CAD Design" width="50%">
    <img src="/images/folding prototype_final.png" alt="Real 3D Printed Prototype" width="50%">
</div>

1. **Frame** — 2020 T-slot aluminum rails on a 3D-printed base with alignment features
2. **Servo motor** — drives the folding actuations
3. **Rack & pinion** — converts servo rotation into the linear stroke needed for bottom folding
4. **Vacuum pump + suction cup** — holds the box securely in position during feeding and folding
5. **Crank-slider** — allows linear motion for pneumatic pump to feed box from stack to fold

Main actuation mechanisms were paired to the motion each fold step demands: a **linkage** drives bottom-folding, a **rack & pinion** for linear actuation.

<div style="display: flex; gap: 1rem;">
    <video style="width: 50%; min-width: 0; max-width: 100%;" autoplay loop muted playsinline controls>
        <source src="/videos/automation/squaring.mp4" type="video/mp4">
        Your browser does not support the video tag.
    </video>
    <video style="width: 50%; min-width: 0; max-width: 100%;" autoplay loop muted playsinline controls>
        <source src="/videos/automation/tuck_in.mp4" type="video/mp4">
        Your browser does not support the video tag.
    </video>
</div>

---

## Control System
<img src="/images/automationschematic.png" alt="Packaging Automation Folding Station CAD" width="100%">

The station runs on **MicroPython** on a Raspberry Pi Pico 2. Servos are driven over I2C through a PCA9685 16-channel PWM driver, with the pump and solenoid valve switched through an optocoupled relay so the Pico's GPIO never carries load current directly. Power is split across two isolated rails — 6V for the servos and driver, 12V for the pump and valve — sharing a common ground, with actuation sequenced rather than simultaneous to keep peak draw within each rail's rating.

A small command interface (`Run [stage]`, `Set <servo> <value>`, `Pump on/off`) drives an 12-step execution sequence per box:

<ol style="columns: 2; column-gap: 2rem; font-size: 0.95rem; line-height: 1.4; margin: 0.5rem 0;">
    <li>Feed the box</li>
    <li>Form the square</li>
    <li>Push the bottom fold down</li>
    <li>Fold side tabs in</li>
    <li>Half bottom fold (60°)</li>
    <li>Fold tuck tab in</li>
    <li>Pull side-tab fixtures out</li>
    <li>Second bottom fold to seat the tuck tab (80°)</li>
    <li>Pull the tuck flap out</li>
    <li>Final full bottom fold</li>
    <li>Move the box placement guide</li>
    <li>Drop the box</li>
</ol>

---

## Future Work for the remainder of Internship
* **Industrialization** — move the design from a rapid prototype to a mechanism reliable enough for continuous use on an industrial manufacturing floor
