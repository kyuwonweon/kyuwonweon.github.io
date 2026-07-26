---
title: "Medtronic NextGen Cervical Plate"
date: 2022-05-12
math: true
summary: "Topology-optimized anterior cervical plate"
tags: ["Ansys", "FEA", "Static Structural", "SolidWorks", "Topology Optimization", "Product Design"]
weight: 10
cover:
    image: "/images/cervicalplate.png" 
    alt: "CAD for anterior cervical plate"
    relative: true
    hiddenInSingle: true
---

## Project Overview
This project, developed in collaboration with **Medtronic**, addresses a critical issue in anterior cervical discectomy and fusion (ACDF) surgeries: post-operative dysphagia caused by bulky implant profiles.

**The Challenge:** Existing cervical plates provide necessary stabilization but often impinge on the pharynx wall due to excessive thickness and mass.

**The Solution:** We developed a "NextGen" low-profile plate system (1-level and 2-level) that maintains FDA-specified load-bearing capabilities while significantly reducing implant volume.

{{< figure src="/images/1evel.png" title="Figure 1: (a) Front (b) Isometric (c) Side View of the 1 level Plate" >}}
{{< figure src="/images/2level.png" title="Figure 2: (a) Front (b) Isometric (c) Side View of the 2 level Plate" >}}

**The Result:** A topology-optimized implant that achieves a **20% reduction in mass** and lower profile compared to the predicate device, validated through ASTM-F1717 computational testing.
{{< figure src="/images/comparison.png" title="Figure 3: Comparison of the Final Prototype (right, top) and Medtronic’s Predicate Plate (left, bottom)" >}}

## Mathematical Foundation
To reduce bulk without compromising safety, we utilized **Topology Optimization**.

**1. Optimization Objective**
The goal was to minimize the mass ($M$) and compliance ($C$) of the plate to find the most efficient material layout:
$$\min_{x} \quad (1 - w) \frac{C(x)}{C_0} + w \frac{M(x)}{M_0}$$
* $x$: Material density distribution variable.
* $w$: Weighting factor.

**2. Constraints (Safety Factors)**
The optimization is constrained by the Von-Mises yield criterion to ensure the device never fails under physiological loads:
$$\sigma_{vm} \leq \frac{\sigma_{yield}}{SF}$$
* **Constraint:** Minimize mass by 90% while maintaining global stress limits.

**3. Load Cases (ASTM-F1717)**
We modeled the FDA-standard testing setup mathematically as a boundary value problem under three distinct loading conditions:
* **Compression:** Simulating head weight.
* **Tension:** Simulating neck extension.
* **Torsion:** Simulating neck rotation.

## System Architecture (Design Process)
The development followed a rigorous V-Model engineering process:

* **Requirements Definition:** Mapped qualitative surgeon feedback (visibility, locking mechanism) to quantitative engineering specifications (profile thickness $< 2.0$mm, safety factor $> 1.5$).
* **Topology Optimization:** Used computational solvers to remove "dead mass" from non-load-bearing regions.
{{< figure src="/images/topology.png" title="Figure 3: Topology Optimization" >}}
* **FEA Validation:**
    * **Static Structural Analysis:** Simulated the screw-plate interface and vertebral loads.
    * **Result:** Final prototype showed peak stresses within safe limits (e.g., -28% displacement in compression vs predicate).
* **Prototyping:** 3D printed (SLA) and machined prototypes for fit-check validation with Medtronic surgical instrumentation.

## Key Challenges & Resolutions

**1. Geometric Constraints vs. Visibility**
* **Challenge:** Surgeons require a clear "viewing window" to monitor bone graft fusion, but reducing material in the center weakens the plate.
* **Resolution:** We optimized the material layout specifically around the screw holes and locking mechanisms, creating a "truss-like" structure that preserved the central window while handling torsional loads.

**2. Patent Landscape**
* **Challenge:** The orthopedic device market is saturated with patents, making unique locking mechanisms difficult to design.
* **Resolution:** We adapted a standard "twist-lock" mechanism compatible with existing Medtronic drivers, ensuring the innovation focused on the *plate topology* rather than the commodity locking screw.

## Future Work
* **Fatigue Testing:** Physical cyclic loading (10 million cycles) to validate long-term durability.
* **Cadaver Labs:** Surgical implantation trials to quantify the reduction in tissue retraction required.


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