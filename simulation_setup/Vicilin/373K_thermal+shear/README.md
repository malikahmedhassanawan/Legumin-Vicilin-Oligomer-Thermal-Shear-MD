# Vicilin trimer: 373 K thermal and shear runs

Two independent replicates, identical except for the random seed used to generate starting velocities in the NVT step:

| Folder | Replicate | NVT gen-seed |
|---|---|---|
| Run_1 | r1 | 111 |
| Run_2 | r2 | 222 |

## Shared equilibration
The thermal and shear runs of the same replicate use the same NVT and NPT equilibration (same .mdp and .tpr files). Only the production step differs, so the equilibrated system is identical at the start of both.

## Files in each Run folder
| File | Used for |
|---|---|
| nvt_373_rX.mdp / .tpr | NVT equilibration (thermal and shear) |
| npt_373_rX.mdp / .tpr | NPT equilibration (thermal and shear) |
| md_373_rX.mdp / .tpr | Thermal production run |
| md_373_shear_rX.mdp / .tpr | Shear production run (adds the deform setting) |

(rX = r1 in Run_1, r2 in Run_2.)

Energy minimisation and ion files, topology and starting structures are in [Common files](https://github.com/malikahmedhassanawan/Legumin-Vicilin-Oligomer-Thermal-Shear-MD/tree/main/simulation_setup/Vicilin/Common%20files).
