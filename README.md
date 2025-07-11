# Reduced Order Modeling for a Chemical Dispenser

[![FEniCS](https://img.shields.io/badge/FEniCS-2019.1-blue.svg)](https://fenicsproject.org/)
[![Python 3.8+](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![NumPy](https://img.shields.io/badge/NumPy-013243.svg?style=flat&logo=numpy&logoColor=white)](https://numpy.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-%23ffffff.svg?style=flat&logo=Matplotlib&logoColor=black)](https://matplotlib.org/)

This project, developed for the "Model Order Reduction techniques" course, focuses on designing and optimizing a chemical dispenser. The system is governed by time-dependent convection-diffusion equations, where the fluid flow is driven by a Stokes problem.

We implement a **POD-Galerkin Reduced Order Model (ROM)** to create a high-fidelity, real-time surrogate of the original **Full Order Model (FOM)**. The primary goal is to leverage this ROM to solve a parameter-dependent optimization problem efficiently, which would be computationally prohibitive using the FOM alone.

---

## 🚀 Showcase & Key Results

### Optimization of Chemical Flow
The main achievement of this project is the successful use of the ROM to solve an optimization problem. The goal was to find the optimal inflow angle $\theta$ that minimizes the amount of chemical delivered to the *bottom* exit gate, ensuring most of it goes to the top one. The ROM's prediction aligns remarkably well with the FOM, but is computed in a fraction of the time.

![ROM vs FOM Optimization](https://i.imgur.com/GgGv3qR.png)
*<p align="center">Figure: The ROM accurately predicts the optimal angle θ that minimizes the bottom outflow J, closely matching the expensive FOM-based optimization.</p>*

### Performance and Speed-Up
The implemented ROM provides a massive computational speed-up while maintaining high accuracy, making it suitable for real-time analysis and design loops.

| Model | Avg. Relative Error (Test Set) | Time for 1 Simulation | Speed-up Factor |
| :--- | :---: | :---: | :---: |
| **FOM** | - | ≈ 14.9 seconds | 1x |
| **ROM (RB Solver)** | 0.88% | ≈ 103 milliseconds | **~144x** |

### High-Fidelity Solution Reconstruction
The ROM can accurately reconstruct the high-dimensional concentration field at various time steps.

![FOM vs ROM Comparison](https://i.imgur.com/8QO9fQJ.png)
*<p align="center">Figure: Visual comparison of the FOM solution (top), the ROM approximation (middle), and the difference (bottom) at different time instances.</p>*

---

## 🔧 How It Works

### 1. The Full Order Model (FOM)
The system models the concentration $u$ of a chemical in a 2D dispenser. The process is described by two coupled physics problems:
1.  **Stokes' Equations:** First, the velocity field of the fluid, **b**, is computed based on three inflow parameters: $\boldsymbol{\mu} = [c_1, c_2, c_3]$.
2.  **Advection-Diffusion Equation:** The velocity field **b** is then used to model the transport of the chemical concentration $u$ over time.

The core equation for the chemical concentration is:
$$
\frac{\partial u}{\partial t} - \frac{1}{2} \Delta u + \mathbf{b}(\boldsymbol{\mu}) \cdot \nabla u = 0
$$

### 2. The Reduced Order Model (ROM): POD-Galerkin
To accelerate the simulations, we built a ROM using the POD-Galerkin methodology.

#### A. Problem Reformulation & Linear Superposition
The problem is first transformed into a homogeneous form ($u_{hom} = u - 1$). Crucially, since the Stokes' equations are linear, the velocity field **b** can be expressed as a linear combination of pre-computed basis solutions:
$$
\mathbf{b}(\boldsymbol{\mu}) = c_1\mathbf{b_1} + c_2\mathbf{b_2} + c_3\mathbf{b_3}
$$
This *offline/online decomposition* is key to the efficiency of the ROM, as the expensive step of solving for the flow field is done only once during the offline phase.

#### B. Proper Orthogonal Decomposition (POD)
We generate a dataset of 20 FOM simulations. From the resulting snapshots, we use **Singular Value Decomposition (SVD)** to extract an optimal, low-dimensional basis (POD modes) that can represent the solution space. The rapid decay of the singular values indicates that a small number of modes (**113 modes**) are sufficient to capture over 99.999% of the system's energy.

![Singular Value Decay](https://i.imgur.com/yGjF1jV.png)
*<p align="center">Figure: The fast decay of singular values confirms the system's suitability for model reduction.</p>*

#### C. Galerkin Projection
The governing PDE is projected onto the reduced basis space spanned by the POD modes. This transforms the high-dimensional system of partial differential equations into a much smaller system of ordinary differential equations (ODEs) which can be solved almost instantly.

### 3. Model Validation
The ROM's accuracy was rigorously tested against the FOM on a separate test set. The model shows excellent performance, with an average relative error of less than 1%.

![Error Quantiles](https://i.imgur.com/lJ4iK5E.png)
*<p align="center">Figure: The distribution of relative errors over time for the 10 test simulations. The median error remains low, and the quantiles show a tight error spread.</p>*

---

## 💻 How to Run

### Dependencies
This project relies on the **FEniCS** library for FEM simulations. Key Python dependencies include:

numpy
matplotlib
scipy

The notebook also uses the custom `dlroms` library provided by the course instructors. A `dispenser.py` file containing helper functions is also required.

### Instructions
1.  **Clone the repository:**
    ```bash
    git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
    cd YOUR_REPO
    ```

2.  **Install FEniCS:**
    Follow the official installation guide for your platform: [FEniCS Documentation](https://fenicsproject.org/download/).

3.  **Install Python packages:**
    ```bash
    pip install numpy matplotlib scipy
    pip install --no-deps git+https://github.com/NicolaRFranco/dlroms.git
    ```

4.  **Run the Jupyter Notebook:**
    Launch the `Chemical_Dispenser_ROM.ipynb` notebook.
    ```bash
    jupyter notebook Chemical_Dispenser_ROM.ipynb
    ```
    The notebook is self-contained. It will generate (or load) the dataset, build the ROM, and reproduce all the figures and results shown here.

---

## 🏁 Conclusion

This project successfully demonstrates the effectiveness of the POD-Galerkin method for a real-world engineering problem. We developed a ROM that is:
-   **Fast:** Achieving a speed-up of over **144x**.
-   **Accurate:** With an average relative error below 1%.
-   **Useful:** Capable of solving a complex optimization problem that would be impractical with the FOM.

This work highlights the potential of model order reduction to accelerate design cycles and enable advanced analysis in computational engineering.

## 🙏 Acknowledgments

This project was completed for the "Model Order Reduction techniques" course (A.Y. 2024-2025) at **Politecnico di Milano**, taught by Prof. Nicola R. Franco and Prof. Andrea Manzoni.
