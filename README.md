# External Flow Analysis of a Straight Rectangular Wing (Clark Y)

> **[Read the full technical report here (PDF)](./rectangularWing.pdf)**

## 1. Introduction
This project analyzes the external aerodynamic flow over a straight rectangular wing using **OpenFOAM**. The study utilizes the **Clark Y airfoil** profile to determine aerodynamic coefficients ($C_l$, $C_d$, $C_p$) and verifies the results against theoretical predictions using **XFOIL** and **Prandtl’s Lifting-Line Theory**.

The simulation employs a steady-state RANS approach with the $k-\omega SST$ turbulence model, targeting a high-fidelity resolution of the boundary layer.

## 2. Problem Description

### Geometry
The computational domain consists of a "Body of Influence" (straight rectangular wing) generated in FreeCAD.

| Parameter | Value |
| :--- | :--- |
| **Airfoil Profile** | Clark Y |
| **Span ($l_s$)** | $1.5\, m$ |
| **Chord ($c$)** | $0.25\, m$ |
| **Aspect Ratio ($AR$)** | 6 |
| **Angle of Attack ($\alpha$)** | $4^{\circ}$ |

### Fluid Properties (Air)
| Property | Value |
| :--- | :--- |
| **Density ($\rho$)** | $1.225\, kg/m^3$ |
| **Kinematic Viscosity ($\nu$)** | $1\times 10^{-5}\, m^2/s$ |
| **Inlet Velocity ($U$)** | $25\, m/s$ (at $4^{\circ}$ to X-axis) |

## 3. Numerical Setup

### Numerical Model
* **Solver:** `simpleFoam` (Steady-state RANS, incompressible)
* **Turbulence Model:** $k-\omega SST$ (Shear Stress Transport)
* **Turbulence Intensity:** Assumed 5% ($T_i = 0.05$) to represent a fully turbulent freestream condition.

### Mesh Statistics
The mesh was generated using `snappyHexMesh` with high refinement levels to capture flow gradients.

| Statistic | Value |
| :--- | :--- |
| **Type** | Unstructured Hex-dominant |
| **Total Cell Count** | **~14.7 Million** |
| **Boundary Layer** | 12 inflation layers (Expansion ratio: 1.3) |
| **First Layer Thickness** | $1.766 \times 10^{-5}\, m$ |
| **Wall Resolution ($y^+$)** | Average $y^+ \approx 1.48$ (Resolved Viscous Sublayer) |

## 4. Results & Verification

The results were verified against XFOIL simulations ($Re = 3.75 \times 10^5$). Since XFOIL is a 2D solver, **Prandtl’s Lifting-Line Theory** was used to estimate the theoretical 3D coefficients for a finite wing ($AR=6$).

### Force Coefficients

| Coefficient | RANS Simulation (OpenFOAM) | Theoretical (XFOIL + Lifting Line) | Deviation |
| :--- | :--- | :--- | :--- |
| **Lift ($C_l$)** | **0.5024** | **0.58** | -13.5% |
| **Drag ($C_d$)** | **0.0311** | **0.0268** | +16.0% |

### Discussion
* **Lift:** The RANS simulation slightly under-predicts lift compared to the theoretical model.
* **Drag:** The drag coefficient is over-predicted ($0.0311$ vs $0.0268$). This is likely due to the **fully turbulent** assumption in the $k-\omega SST$ model, which generates higher skin friction compared to XFOIL's transition model.
* **Mesh Resolution:** With an average $y^+$ of 1.48, the simulation successfully resolves the linear region of the boundary layer, confirming the validity of the near-wall physics.

## 5. Conclusion

This study analyzed the external aerodynamic flow over a finite rectangular wing with a Clark Y airfoil using a steady-state RANS approach ($k-\omega$ SST turbulence model). The simulation results were verified against theoretical predictions derived from XFOIL data and Prandtl’s Lifting-Line Theory.

The key findings are summarized below:

* **Lift Coefficient ($C_l$):** The simulation predicted a lift coefficient of **0.5024**, which is approximately **13.5% lower** than the theoretical estimation of 0.58. This under-prediction is primarily attributed to the RANS model's assumption of fully turbulent flow over the entire wing surface. This assumption leads to a thicker boundary layer at the trailing edge compared to the XFOIL model, which accounts for laminar flow in the leading-edge region.
* **Drag Coefficient ($C_d$):** The calculated drag coefficient was **0.0311**, representing a **16% over-prediction** compared to the theoretical value of 0.0268. This discrepancy is consistent with the fully turbulent flow assumption, which generates higher skin friction drag ($C_{d,profile}$) than the transitional flow regime assumed in the XFOIL benchmark.
* **Flow Physics & Validation:** The pressure coefficient ($C_p$) distribution closely matches the characteristic shape of the XFOIL data. Additionally, the simulation successfully captured 3D flow features, specifically the formation of wingtip vortices and the resulting downwash. The mesh quality was validated with an average $y^+$ of $\approx 1.48$, confirming that the viscous sublayer was adequately resolved.

In conclusion, the OpenFOAM simulation successfully captures the fundamental aerodynamics of a finite wing. While the fully turbulent $k-\omega$ SST model yields conservative estimates for lift and drag compared to 2D transitional codes, the results are physically consistent and valid for a fully turbulent flight regime.

## 6. How to Run

**Prerequisites:** OpenFOAM (v2412 or equivalent), MPI, ParaView.

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

3.  **Run Simulation (Parallel):**
    ```bash
    decomposePar
    mpirun -np <N_CORES> simpleFoam -parallel
    reconstructPar
    ```

4.  **Post-processing:**
    ```bash
    paraFoam
    ```

## 7. References
1.  UIUC Airfoil Coordinates Database.
2.  Prandtl’s Classical Lifting-Line Theory.
3.  XFOIL Subsonic Airfoil Analysis.
