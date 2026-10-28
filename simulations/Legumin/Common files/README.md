# Legumin hexamer: common setup files

Files shared by all legumin hexamer runs (300 K thermal, 373 K thermal, 373 K shear; replicates r1 and r2). Run-specific files (NVT, NPT and production .mdp/.tpr) are in each run's folder.

## System
- Protein: pea legumin hexamer, 6 chains (A–F), AlphaFold3 model
- Force field: <force field>
- Water model: <water model>
- Box: <box type, distance>
- Ions: <ion type>, <concentration> (neutralised)
- GROMACS version: <version>

## Files
| File | Description |
|---|---|
| topol.top | System topology |
| topol_Protein_chain_A–F.itp | Per-chain topologies (included by topol.top) |
| posre_Protein_chain_A–F.itp | Position restraints per chain |
| solv.gro | Solvated system (before ion addition) |
| ions.mdp | Parameters for genion step |
| em.mdp | Energy minimisation parameters |
| em.tpr | Energy minimisation input (full system with ions) |
| em.gro | Minimised structure, starting point for all NVT runs |

## Setup order
pdb2gmx → editconf → solvate → genion → energy minimisation (em)

## Notes
- Keep topol.top and all .itp files in the same folder.
- r1 and r2 replicates differ only in the NVT gen-seed (r1 = 111, r2 = 222).
- Run-specific files are in ../300K_thermal/, ../373K_thermal/ and ../373K_shear/.
