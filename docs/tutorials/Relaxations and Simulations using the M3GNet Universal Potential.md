---
layout: default
title: Relaxations and Simulations using the M3GNet Universal Potential.md
nav_exclude: true
---

# Introduction

This notebook demonstrates the use of the pre-trained M3GNet model to perform structural relaxations, molecular dynamics simulations and single-point calculations.

Author: Tsz Wai Ko (Kenko)
Email: t1ko@ucsd.edu


```python
from __future__ import annotations

import warnings

from ase.md.velocitydistribution import MaxwellBoltzmannDistribution
from pymatgen.core import Lattice, Structure
from pymatgen.io.ase import AseAtomsAdaptor

import matgl
from matgl.ext.ase import M3GNetCalculator, MolecularDynamics, Relaxer

# To suppress warnings for clearer output
warnings.simplefilter("ignore")
```

# Loading the pre-trained M3GNet PES model

We will first load the M3GNet PES model, which is trained on the MP-2021.2.8 dataset. This can be done with a single line of code.


```python
pot = matgl.load_model("M3GNet-MP-2021.2.8-PES")
```

# Structure Relaxation

To perform structure relaxation, we use the Relaxer class. Here, we demonstrate the relaxation of a simple CsCl structure.


```python
relaxer = Relaxer(potential=pot)
struct = Structure.from_spacegroup("Pm-3m", Lattice.cubic(4.5), ["Cs", "Cl"], [[0, 0, 0], [0.5, 0.5, 0.5]])
relax_results = relaxer.relax(struct, fmax=0.01)
# extract results
final_structure = relax_results["final_structure"]
final_energy = relax_results["trajectory"].energies[-1]
# print out the final relaxed structure and energy

print(final_structure)
print(f"The final energy is {float(final_energy):.3f} eV.")
```

    Full Formula (Cs1 Cl1)
    Reduced Formula: CsCl
    abc   :   4.211410   4.211410   4.211410
    angles:  90.000000  90.000000  90.000000
    pbc   :       True       True       True
    Sites (2)
      #  SP      a     b     c
    ---  ----  ---  ----  ----
      0  Cs    0    -0    -0
      1  Cl    0.5   0.5   0.5
    The final energy is -6.543 eV.


# Molecular Dynamics

MD simulations are performed with the ASE interface.


```python
ase_adaptor = AseAtomsAdaptor()
# Create ase atom object
atoms = ase_adaptor.get_atoms(final_structure)
# Initialize the velocity according to Maxwell Boltzamnn distribution
MaxwellBoltzmannDistribution(atoms, temperature_K=300)
# Create the MD class
driver = MolecularDynamics(atoms, potential=pot, temperature=300, logfile="md_trial.log")
# Run
driver.run(100)
print(f"The potential energy of CsCl at 300 K after 100 steps is {float(atoms.get_potential_energy()):.3f} eV.")
```

    The potential energy of CsCl at 300 K after 100 steps is -6.430 eV.


# Single point energy calculation

Perform a single-point calculation for final structure using M3GNetCalculator.


```python
# define the M3GNet calculator
calc = M3GNetCalculator(pot)
# set up the calculator for atoms object
atoms.set_calculator(calc)
print(f"The calculated potential energy is {float(atoms.get_potential_energy()):.3f} eV.")
```

    The calculated potential energy is -6.430 eV.
