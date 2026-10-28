# Vicilin trimer: 373 K thermal runs

Two independent replicates, identical except for the random seed used to generate starting velocities in the NVT step:

| Folder | Replicate | NVT gen-seed |
|---|---|---|
| Run_1 | r1 | 111 |
| Run_2 | r2 | 222 |

Each folder contains the NVT, NPT and production .mdp/.tpr files for that replicate (`nvt_373_rX`, `npt_373_rX`, `md_373_rX`; rX = r1 in Run_1, r2 in Run_2). The same NVT and NPT equilibration is used for the 373 K shear runs, which are in `373K_thermal+shear`. Energy minimisation and ion files, topology and starting structures are in [Common files](https://github.com/malikahmedhassanawan/Legumin-Vicilin-Oligomer-Thermal-Shear-MD/tree/main/simulation_setup/Vicilin/Common%20files).
