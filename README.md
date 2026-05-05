

## Overview
This repository provides a specialized computational framework for the system identification of structures subjected to Flow-Induced Vibrations (FIV). The codebase is designed to process empirical or numerical time-history data—specifically aerodynamic lift forces and structural displacements—to extract the governing phenomenological parameters of Vortex-Induced Vibrations (VIV) and transverse galloping. 

By inversely solving the governing differential equations, this suite enables researchers and engineers to quantify non-linear fluid-structure interaction (FSI) coefficients without relying on manual heuristic fitting.

---

## Repository Contents

The framework comprises three primary Jupyter Notebooks, each implementing a distinct mathematical approach tailored to specific dynamic regimes:

| Routine | Primary Application | Methodological Approach |
| :--- | :--- | :--- |
| **`CircularCylinder.ipynb`** | Analysis of pure Vortex-Induced Vibrations (VIV) typical of circular cross-sections. | Employs ridge regression to isolate parameters for the standard Facchinetti wake oscillator model. |
| **`GallopingIdentificationWithoutNN.ipynb`** | Analysis of combined VIV and transverse galloping, standard for asymmetric or D-section geometries. | Utilizes bounded optimization (L-BFGS-B) to decompose total lift into a wake oscillator variable and a quasi-steady polynomial. |
| **`GallopingIdentificationWithNN.ipynb`** | Advanced parameter extraction for highly non-linear galloping phenomena. | Implements a Physics-Informed Neural Network (PINN) / Neural-SINDy framework utilizing a SIREN architecture to enforce structural governing equations during training. |

---

## Theoretical Background

The identification algorithms rely on standard reduced-order models for fluid-structure interaction.

* **Vortex-Induced Vibration (VIV):** The framework models the vortex shedding wake as a non-linear Van der Pol oscillator coupled with the structural dynamics. The algorithms identify the empirical parameters $\epsilon$ (controlling non-linear saturation) and $A$ (scaling the structural acceleration coupling effect).
* **Transverse Galloping:** Modeled via a quasi-steady aerodynamic assumption using a polynomial expansion of the structural velocity. The scripts calculate the linear ($a_1$ or $\beta$) and cubic ($a_3$) galloping lift coefficients. A derived linear coefficient of $a_1 > 0$ satisfies the Den Hartog criterion, analytically confirming a susceptibility to galloping instability.

---

## Methodology and Computational Pipeline

Each routine executes a strictly defined signal processing and mathematical optimization pipeline:

1.  **Data Ingestion & Truncation:** The system reads raw delimited text files containing aerodynamic forces and structural kinematics, automatically applying a masking function (e.g., $t > 50.0$) to isolate the steady-state dynamic response from initial numerical or experimental transients.
2.  **Frequency Domain Analysis:** A Fast Fourier Transform (FFT) is applied to the detrended displacement data to extract the dominant oscillation frequency ($f_n$) and the corresponding angular frequency ($\omega_n$).
3.  **Signal Conditioning:** To preserve low-frequency non-linear harmonics while mitigating high-frequency noise, the data is processed through a lowpass Butterworth filter.
4.  **Kinematic Derivation:** Structural velocity ($\dot{y}$) and acceleration ($\ddot{y}$) are derived from the displacement signal using Savitzky-Golay filtering or auto-differentiation (in the PyTorch implementation).
5.  **Parameter Optimization:** The defined cost function—minimizing the residual of the analytical differential equations against the processed dataset—is solved via either linear algebra, L-BFGS-B optimization, or gradient descent.

---

## Configuration and Usage Instructions

To apply this framework to external datasets, the user must define specific experimental or numerical parameters within the code blocks.

### 1. Data Source Modification
By default, the routines ingest two specific files:
* `drag_lift_body_001.dat`: Contains fluid force time-histories.
* `viv_body1_probe.dat`: Contains structural kinematic time-histories.

**Requirement:** Update the file paths in the `load_data()` functions to point to the relevant local datasets. 

### 2. Array Formatting Constraints
Ensure the input matrices align with the expected array shapes. 
* The fluid data ingestion skips the first row (`skiprows=1`) and parses columns identified as: `time`, `cxp`, `cxs`, `cx`, `cyp`, `cys`, `cy`.
* The structural data ingestion skips the first two rows (`skiprows=2`) and parses columns identified as: `ntime`, `time`, `DX`, `DY`, `VY`, `AY`.
Adjust the `skiprows` parameter and `names` arrays to match the specific header architecture of your data.

### 3. Structural Parameter Initialization
The solvers require accurate baseline structural properties to condition the governing equations. The default scripts are parameterized for a standard low-mass-ratio test case:
* **Mass Ratio ($m$):** Initialized at 5.0.
* **Damping Ratio ($\zeta$ or $\xi$):** Initialized at 0.01 for the deterministic optimization scripts and 0.0 for the neural network implementation.

**Requirement:** Overwrite `m_total` (or `M_RATIO`) and `zeta` (or `XI`) with the precise parameters of the physical domain under investigation.

---

## Dependencies

Execution of these notebooks requires a standard scientific Python environment (Python 3.7+). 

* `numpy`
* `pandas`
* `scipy`
* `torch` (Required exclusively for `GallopingIdentificationWithNN.ipynb`)

**Installation Environment:**
```bash
pip install numpy pandas scipy torch
```
