# Methods

## Rationale and System Overview

This study focuses on understanding how gatekeeper mutations in the **KIT kinase** influence the binding of two structurally distinct kinase inhibitors: **avapritinib (AVA)** and **midostaurin (MID)**. The clinically relevant **D816V mutation** was used as a reference state ("wild-type" in this context), and **three additional gatekeeper mutations** at **residue T670**—A (T670A), V (T670V), and I (T670I)—were introduced to form double mutants.

Both ligands were docked into homology models of KIT kinase carrying these mutations, and **molecular dynamics (MD) simulations** were used to explore the conformational ensemble. Snapshots from these simulations served as inputs for **non-equilibrium thermodynamic integration (TI)** free energy calculations using **soft-core potentials in a modified GROMACS SC build**. This workflow enabled estimation of **ΔΔG** (mutation-induced changes in binding free energy) and **ΔΔΔG** (ligand selectivity changes due to mutations), compared with **experimental IC₅₀-derived ΔG values** from **Ba/F3 cell viability assays**.

---

## 1. Model Creation

### 1.1 Homology Modeling

- Starting structure: **KIT kinase (PDB ID: 1PKG)**.
- Missing loops were modeled using **PDGFR (PDB ID: 6JOL)**.
- The **D816V** mutation was introduced using **ChimeraX**.
- ATP was removed to generate the **D816V-apo** configuration.

### 1.2 Docking

- **Ligands**: Avapritinib and Midostaurin.
- Docked into the D816V-apo structure using **Schrödinger Glide XP**.
- Resulting complexes: **D816V-ligand bound** forms.

### 1.3 Creation of Double Mutants

- Gatekeeper mutations: T670 → A, V, I.
- Final set: **D816V/T670A, D816V/T670V, D816V/T670I** (each with AVA and MID bound).
- Mutants generated in **ChimeraX**.

---

## 2. Molecular Dynamics (MD) Simulations

### 2.1 System Setup

- All systems solvated in **TIP3P water** with Na⁺ and Cl⁻.
- Simulated in **GROMACS** with 3 × 500 ns replicas per system.

### 2.2 Energy Minimization

- Method: **Steepest descent**, 50,000 steps max.
- Convergence: Max force < 1000 kJ/mol/nm.
- Cutoffs: 1.0 nm for van der Waals and Coulomb.
- Long-range electrostatics: **PME method**.
- **LINCS** used for constraining H-bonds.

### 2.3 Equilibration

- 10 ns (5,000,000 steps), 2 fs time step.
- Ensemble: **NVT**, Temp = 300 K.
- Center of mass motion removed every 500,000 steps.
- Position restraints on protein.

### 2.4 Production Run

- Duration: 500 ns (250,000,000 steps).
- Ensemble: **NPT**, Temp = 300 K, Pressure = 1 bar.
- Snapshots saved every 1,000,000 steps.

### 2.5 Post-Processing

- Extracted snapshots from **100 ns onward**, skipping every 4th frame.
- Snapshots (GRO files) used in TI simulations.
- Periodic boundary conditions corrected using `trjconv`.

---

## 3. Free Energy Simulations

### 3.1 Hybrid Topology Generation

- **PMX** used to build hybrid topologies.
- Transitions: WT ↔ Mutant for each gatekeeper mutation.
- Separate forward and reverse transitions prepared.

### 3.2 Minimization and Equilibration

- **Minimization**: Steepest descent, same settings as above.
- **Equilibration**: 500 ps with position restraints.
- Ensemble: **NPT**, Temp = 310 K, Pressure = 1 bar.
- Soft-core settings applied during equilibration.

### 3.3 Non-Equilibrium Fast Thermodynamic Integration (TI)

- λ transitions using delta-lambda = **0.00002**.
- **Soft-core potentials** enabled:
  - `sc-alpha = 0.3`, `sc-sigma = 0.3`
- Temperature control: **V-rescale thermostat**.
- Pressure control: **Berendsen barostat**.
- `nstdhdl = 10` to log ∂H/∂λ frequently.

### 3.4 Free Energy Analysis

- **Work values** collected across snapshots.
- Free energy estimates (ΔG) computed using:
  - **Jarzynski Equality**
  - **Crooks Fluctuation Theorem**
  - **Bennett Acceptance Ratio (BAR)** via PMX

---

## 4. Experimental Assays

### 4.1 Ba/F3 Cell Assay (IC₅₀)

- Ba/F3 cells expressing:
  - D816V
  - D816V + T670A / T670V / T670I
- IC₅₀ values measured for AVA and MID.

| System          | IC₅₀ AVA (nM) | IC₅₀ MID (nM) |
|-----------------|--------------|---------------|
| D816V           | 3.8          | 36            |
| D816V/T670A     | 20           | 12            |
| D816V/T670V     | 360          | 7             |
| D816V/T670I     | 270          | 180           |

---

## 5. Experimental & Computational ΔG Calculations

### 5.1 ΔG from IC₅₀

- Equation:  
  \[
  ΔG = RT \cdot \ln(IC₅₀)
  \]
  where R = 0.001987 kcal/mol·K, T = 298.15 K

### 5.2 ΔΔG and ΔΔΔG Definitions

- ΔΔG: Mutation effect on binding affinity
  \[
  ΔΔG = ΔG_{mutant} - ΔG_{WT}
  \]
- ΔΔΔG: Ligand preference shift due to mutation
  \[
  ΔΔΔG = ΔG_{MID} - ΔG_{AVA}
  \]

### 5.3 Summary Table

| Mutation | ΔΔG-AVA (exp) | ΔΔG-AVA (comp) | ΔΔG-MID (exp) | ΔΔG-MID (comp) | ΔΔΔG (exp) | ΔΔΔG (comp) |
|----------|---------------|----------------|---------------|----------------|------------|-------------|
| T670A    | -0.98         | -3.63          | 0.65          | -1.28          | -1.63      | -2.35       |
| T670V    | -2.70         | -1.88          | 0.97          | 4.39           | -3.67      | -6.27       |
| T670I    | -2.53         | -0.12          | -0.95         | 3.43           | -1.57      | -3.55       |

---

## Notes on Computational Choices

- **Soft-core potentials** prevent singularities when atoms are alchemically introduced or deleted.
- **Non-equilibrium TI** allows use of fast transitions to estimate equilibrium ΔG values.
- **PMX + GROMACS SC** offers flexibility for hybrid topology simulations and integration with Bennett estimator and fluctuation theorems.

---

## References

(Include links to software, tools, and any supporting publications.)

