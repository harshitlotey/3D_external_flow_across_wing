# External Flow Analysis of a Straight Rectangular Wing (Clark Y)

> [!Note]
> **Status:** The current mesh configuration results in an average $y^+$ of 22. While Lift ($C_l$) results are valid, this wall resolution leads to an over-prediction of Drag ($C_d$) when using the $k-\omega SST$ model. See **Section 4** for details.

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

### Numerical Model
* **Solver:** Steady-state RANS (`simpleFoam`).
* **Turbulence:** $k-\omega SST$ model.
* **Wall Treatment:** Wall functions for $k$, $\omega$, and $\nu_t$.

### Mesh Statistics
* **Type:** Hex-dominant mesh (`snappyHexMesh`).
* **Cell Count:** $\approx 2.59$ million.
* **Boundary Layer:** 5 inflation layers (Expansion ratio: 1.2).
* **Wall Resolution:** Average $y^+ \approx 22$ (Buffer layer).

## 4. Results & Verification

The primary goal was to calculate Lift ($C_l$) and Drag ($C_d$) coefficients and verify them against XFOIL data corrected for finite wing effects using Prandtl's Lifting-Line Theory.

### Force Coefficients
| Coefficient | RANS Simulation (This Project) | Theoretical Prediction (XFOIL + Lifting Line) |
| :--- | :--- | :--- |
| **Lift ($C_l$)** | **0.556** | **0.627** |
| **Drag ($C_d$)** | **0.0409** | **0.03** |

### Data Analysis
* **Lift:** The simulation aligns with theoretical expectations for a finite wing, confirming that the bulk pressure distribution is captured correctly.
* **Drag:** The drag coefficient is higher than the theoretical baseline. This discrepancy is attributed to the average $y^+$ value of 22. The $k-\omega SST$ model typically requires $y^+ < 1$ to accurately resolve skin friction drag; operating in the buffer layer ($5 < y^+ < 30$) results in the observed over-prediction.

### Flow Visualization
The simulation successfully captured wingtip vortices, illustrating the induced drag and downwash effects characteristic of finite wings.

## 5. Limitations
* **Wall Resolution:** The current $y^+$ of 22 falls within the buffer layer. Future iterations will target $y^+ < 1$ to improve drag accuracy.
* **Domain Size:** The wake region extends 12 chord lengths downstream; extending this would further isolate the outlet boundary.

## 6. How to Run
*Prerequisites: OpenFOAM 2412 (ESI) or later (or equivalent fork), ParaView for post-processing.*

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
