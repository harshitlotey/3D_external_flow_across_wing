# External Flow Analysis of a Straight Rectangular Wing (Clark Y)

## 1. Introduction
This repository contains an OpenFOAM case study analyzing the external aerodynamic flow over a straight rectangular wing. The project utilizes the **Clark Y airfoil** profile to determine aerodynamic coefficients and validate results against theoretical predictions using **XFOIL** and **Prandtl’s Lifting-Line Theory**.

The simulation relies on steady-state RANS (Reynolds-Averaged Navier-Stokes) equations using the $k-\omega SST$ turbulence model.

## 2. Problem Description

### Geometry
The "Body of Influence" is a straight rectangular wing generated via FreeCAD.
* **Airfoil:** Clark Y
* **Span ($l_s$):** $1.5 m$
* **Chord ($c$):** $0.25 m$
* **Aspect Ratio ($AR$):** 6
* **Angle of Attack ($\alpha$):** $4^{\circ}$


### Fluid Properties (Air)
* **Density ($\rho$):** $1.225 kg/m^3$
* **Kinematic Viscosity ($\nu$):** $1e-5 m^2/s$
* **Inlet Velocity ($U$):** Magnitude of $\approx 25 m/s$ at $4^{\circ}$ angle.

## 3. Case Setup & Pre-processing

### Domain & Meshing
The computational domain is a box surrounding the wing, modeled with a symmetry boundary condition on the middle parting plane to reduce computational cost.

* **Software:** `snappyHexMesh` utility.
* **Base Element Size:** $0.1 m$.
* **Refinement:**
    * Curvature captured at Level 5 ($15^{\circ}$ feature angle).
    * Wake region refinement: Level 2.
* **Boundary Layers:** 5 inflation layers with an expansion ratio of 1.2.
* **Total Cells:** $\approx 2.59$ million cells.


### Numerical Model
* **Solver Type:** Steady-state incompressible (implied `simpleFoam`).
* **Turbulence Model:** $k-\omega SST$ with assumed $5\%$ turbulence intensity.
* **Boundary Conditions:**
    * **Inlet:** Constant velocity.
    * **Outlet:** Constant pressure.
    * **Wing Surface:** No-slip wall with wall functions for $k$, $\omega$, and $\nu_t$.
    * **Symmetry:** Symmetry plane.

## 4. Results & Verification

The primary goal was to calculate Lift ($C_l$) and Drag ($C_d$) coefficients and verify them against XFOIL data corrected for finite wing effects using Prandtl's Lifting-Line Theory.

### Force Coefficients
| Coefficient | RANS Simulation (This Project) | Theoretical Prediction (XFOIL + Lifting Line) |
| :--- | :--- | :--- |
| **Lift ($C_l$)** | **0.556** | **0.627** |
| **Drag ($C_d$)** | **0.0409** | **0.03** |

### Pressure Coefficient ($C_p$)
The $C_p$ vs. $x/c$ plot demonstrates behavior consistent with XFOIL benchmarks, though the peak suction magnitude differs slightly due to mesh resolution limitations.


### Flow Visualization
The simulation successfully captured wingtip vortices, illustrating the induced drag and downwash effects characteristic of finite wings.

## 5. Limitations
* **Mesh Resolution:** The mesh was not refined enough to fully capture intense pressure gradients, leading to slight inaccuracies in $C_p$ minima.
* **Domain Size:** The domain boundaries may still influence the flow due to computational resource constraints.

## 6. How to Run
*Prerequisites: OpenFOAM 12 (Foundation) or later (or equivalent fork), ParaView for post-processing.*

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/harshitlotey/3D_external_flow_across_wing.git
    cd 3D_external_flow_across_wing
    ```

2.  **Generate Mesh:**
    ```bash
    blockMesh
    surfaceFeatureExtract
    snappyHexMesh -overwrite
    ```

3.  **Run Simulation:**
    *Decompose if running in parallel*
    ```bash
    decomposePar
    mpirun -np <number_of_cores> simpleFoam -parallel
    reconstructPar
    ```
    *(Note: If running serial, simply execute `simpleFoam`)*

4.  **Post-processing:**
    ```bash
    paraFoam
    ```

## 7. Author
**Harshit Dhiman**
*Research and Development Engineer*
*Date: July, 2025*

## 8. References
1.  UIUC Airfoil Coordinates Database.
2.  Prandtl’s Classical Lifting-Line Theory.
3.  XFOIL Subsonic Airfoil Analysis.
