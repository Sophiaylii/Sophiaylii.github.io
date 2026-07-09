---
layout: post
title: Structural Analysis of a Composite Wing Panel
description: 
  Full-cycle structural analysis and manufacturing of a carbon fiber composite wing panel for a high-performance RC aircraft spanning aerodynamic load definition, analytical hand calculations, FEA validation, composite margin analysis, and physical fabrication.
skills:
  - XFLR5
  - MATLAB
  - Python
  - Fusion 360
  - Abaqus FEA
  - HyperX Composite Analysis (Collier Aerospace)
  - Composite Laminate Theory
  - Wet Layup Manufacturing
  - Hot Wire Foam Cutting
  - Structural Analysis
main-image: /cover-aero-wing.jpg
---

## Overview

This project documents the full design, analysis, and manufacturing cycle of a carbon fiber composite wing panel for a high-performance electric RC aircraft designed to complete two missions: a payload mission carrying four stores, and a high-speed sprint mission. The wing panel is the primary load-bearing structure, and this project traces every step from aerodynamic load definition to composite margin analysis to physical fabrication, executed independently over approximately 2.5 weeks.

The analysis follows a multi-fidelity validation approach: hand calculations, 2D FEA slice, 3D full-panel FEA, composite margin assessment in HyperX. Every level validates the previous, and the final root reaction force agreed with theory to within 0.06%.

---

## Design Parameters

The wing was designed around an all-up mass of 2.5 kg, an aspect ratio of 7, and a cruise CL of 0.6, typical of RC aircrafts. Wing geometry was derived from the lift equation and sized 15% above minimum for handling margin, with the wing area being 0.123m^2, chord length as 0.132m, and the wingspan as 0.927m. 

The NACA 2412 airfoil was selected for its well-documented lift characteristics, moderate camber suitable for payload-carrying, and validated aerodynamic data availability in XFLR5.

### Panel construction:
- 2-ply carbon fiber skin (0/90, 210g 2x2 twill 3k TR30S fiber)
- XPS foam core (35 kg/m^3)
- Pultruded CF spar (8mm OD / 6mm ID) at 25% chord
- Balsa root and tip ribs, plywood inboard and outboard ribs

### Governing load case: 
Mission 2 sprint: 25 m/s, 2.4g maneuver load factor (constrained by CLmax, corrected from initial 3g assumption), producing a root bending moment of 13.18 Nm.

Full calculations and justifications are provided in the engineering report, currently in progress. 

---

## Aerodynamic Load Definition: XFLR5

Spanwise lift distribution L'(y) was extracted from XFLR5 for both missions across a range of angles of attack. The 3D panel analysis confirmed Mission 2 governs due to the combined effect of higher speed and aggressive maneuvering. 

{% include image-gallery.html images="xflr1.png, xflr2.png" height="200" %}
<span style="font-size: 10px">Left: 2D foil analysis, NACA 2412, Mission 1 vs. Mission 2 comparison. Right: 3D wing panel analysis, Mission 2 at 25 m/s.</span>

A critical units error was identified and corrected during this phase: the raw XFLR5 export was approximately 1000x too large (N/m interpreted as N/mm). After correction, an 11-point spanwise L'(y) table was generated and used as input to all downstream analyses.

{% include image-gallery.html images="matlab1.png, matlab2.png" height="360" %}
<span style="font-size: 10px">Bending moment vs. spanwise position, Mission 1 vs. Mission 2 without payload (left) and Mission 1 with stores vs. Mission 2 (right). Mission 2 governs at 13.18 Nm root bending moment.</span>

---

## Analytical Hand Calculations: MATLAB

Bending stiffness (EI) was computed analytically in MATLAB using EA-weighted parallel-axis contributions from each structural component. The skin and spar were modeled as thin-walled sections, with EI computed by integrating modulus-weighted second moments of area about the neutral axis.

### Result: CF skin carries ~93% of total panel bending stiffness. Spar carries ~7%.

This result establishes the hand-calculation baseline that all subsequent FEA models validate against.

{% include image-gallery.html images="ei1.png, ei2.png" height="400" %}
<span style="font-size: 10px">MATLAB output confirming skin fraction, 91.33% from 2D FEA validation run (left), and MATLAB theoretical hand calculation (right).</span>

---

## Finite Element Analysis: Abaqus

### 2D Cross-Section Slice Model

A 2D thin-slice model of the airfoil cross-section was built in Abaqus with the 13.18 Nm root bending moment applied directly. Section forces were extracted and integrated separately for skin and spar elements.

{% include image-gallery.html images="2dfea.png" height="360" %}
<span style="font-size: 10px">2D Abaqus SF1 (spanwise force resultant) contour plot, tension on upper surface, compression on lower.</span>

Result: Skin EI fraction = 91.3% in FEA vs. 93% in MATLAB, agreement within 2%. MATLAB hand calculations validated.

---

### 3D Full-Panel Shell Model

The full 3D wing panel was modeled in Abaqus using shell elements, with the spanwise lift distribution applied as an analytical field pressure load. Foam, ribs, and spar were converted to shells for HyperX compatibility while preserving mass and stiffness properties.

{% include image-gallery.html images="cad.png, undeformed.png" height="360" %}
<span style="font-size: 10px">Left: Fusion 360 CAD assembly showing foam sections, ribs, spar, and skin. Right: Undeformed Abaqus shell mesh, full 424mm structural panel.</span>

{% include image-gallery.html images="sf1.png, rf3.png" height="360" %}
<span style="font-size: 10px">Left: SF1 deformed contour, bending load path clearly visible through skin. Right: RF3 (vertical reaction force) distribution at root boundary condition.</span>

A units error in the pressure load definition (N/mm entered instead of N/mm^2) was independently caught and corrected during model verification. Reaction forces had been 15-20x too high, which triggered the investigation.

| Validation Check | Predicted | Reference | Agreement |
|---|---|---|---|
| Skin EI fraction (2D FEA vs. MATLAB) | 91.3% | 93% | ~2% |
| Root reaction RF3 (3D FEA vs. theory) | -15.88 N | -15.89 N | 0.06% |

---

## Composite Margin Analysis: HyperX (Collier Aerospace)

FEA-extracted shell load resultants were imported into HyperX for composite margin assessment. Structures and zones were created for all panel components, with material properties assigned from the FEM and verified against source data. Failure modes analyzed include panel buckling (biaxial, shear, interaction), composite ply strength (Max Strain, Max Stress, Tsai-Wu, Tsai-Hill, LaRC03), and isotropic strength for foam and wood components.

{% include image-gallery.html images="tsaiwu.png" height="420" %}
<span style="font-size: 10px">HyperX margin of safety plot, Tsai-Wu criterion across full panel. Minimum MS = 76.5 at tip zone. All zones positive.</span>

**All failure modes returned positive margins of safety across the full panel:**

| Failure Mode | Min MS | Governing Zone |
|---|---|---|
| Panel Buckling - Biaxial + TSF | 27.4 | Zone 10 (tip) |
| Panel Buckling - Shear + TSF | Very High | - |
| Panel Buckling - Interaction + TSF | **26.7** | Zone 10 (tip) |
| Composite, Tsai-Wu | 76.5 | Zone 10 (tip) |
| Composite, Max Strain 1 | 104 | Zone 10 (tip) |
| Composite, Tsai-Hill | 79.0 | Zone 10 (tip) |

**Governing failure mode: Panel buckling under biaxial-shear interaction, MS = 26.7.**
**Governing strength failure: Tsai-Wu criterion, MS = 76.5.**

The panel is structurally adequate under Mission 2 design loads with significant positive margin across all failure criteria. The CF skin carries ~93% of bending stiffness, consistent across all three levels of analysis.

---

## Manufacturing

{% include image-gallery.html images="manufacture.jpg" height="420" %}

*In progress — physical test results to be added upon completion of 3-point bend test.*

The panel was fabricated using wet layup with 210g 2x2 twill 3k TR30S carbon fiber and US Composites 635 thin epoxy , selected for compatibility with the XPS foam core's 74C service temperature limit which ruled out pre-preg processing.

Foam core was cut to NACA 2412 profile using a hot wire foam cutter designed and built specifically for this project. The cutter uses a tensioned nichrome wire with a variable voltage supply, guided by NACA 2412 templates at root and tip stations to produce accurate profile cuts across the full panel span.

Ribs were hand-cut from balsa (root and tip) and aircraft-grade birch plywood (inboard and outboard store stations), sanded to final profile, and fitted with spar tube pass-through holes aligned to 25% chord.

Skin layup was performed as a wet layup with two plies of CF fabric in 0/90 orientation over the foam core, with the pultruded CF spar tube seated in the core prior to bagging. The panel was cured at room temperature

---

## Key Independent Catches

Four significant errors were independently identified and corrected during the project. Each caught through sanity checks rather than discovered after the fact:

- **Load factor correction** - Initial 3g assumption was reduced to 2.4g after recognizing CLmax constrained the achievable maneuver load factor
- **XFLR5 units error** - Raw export was ~1000x too large (N/m interpreted as N/mm); corrected before applying to structural model
- **Abaqus pressure units error** - Pressure load entered as N/mm instead of N/mm^2 (MPa), causing reaction forces 15-20x too high; caught via RF3 sanity check against theory
- **Spar sizing error** - Radius/diameter confusion in spar cross-section definition; caught during geometry verification

---

## Tools & Skills

| Tool | Role |
|---|---|
| XFLR5 | 2D/3D aerodynamic analysis, spanwise lift distribution |
| MATLAB | Bending stiffness hand calculations, load integration, bending moment diagrams |
| Fusion 360 | Full 3D CAD assembly |
| Abaqus 2024 | 2D slice validation, 3D full-panel FEA, shell element modeling |
| HyperX 2026.1.16 | Composite margin analysis, failure mode assessment |
| Wet Layup | CF skin fabrication |
| Hot Wire Cutter | XPS foam core cutting to NACA 2412 profile |

---

*Full engineering report available below (in progress). Bend test results to follow.*
