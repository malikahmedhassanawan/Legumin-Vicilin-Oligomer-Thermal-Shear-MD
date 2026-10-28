# Legumin hexamer: common setup files

Files shared by all legumin hexamer runs (300 K thermal, 373 K thermal, 373 K shear; replicates r1 and r2). Run-specific files (NVT, NPT and production .mdp/.tpr) are in each run's folder.

## System
- Protein: pea legumin hexamer, 6 chains (A–F), AlphaFold3 model (starting structure in PDB and CIF format in the repository's `structures/` folder)
- Force field: AMBER99SB-ILDN
- Water model: SPC/E
- Box: rhombic dodecahedron (xy-square), vectors 14.504 × 14.504 × 10.256 nm
- Solvent: 56,682 water molecules
- Ions: Na+/Cl- (231 Na+, 195 Cl-), neutralised
- GROMACS version: 2026.3

## Files
| File | Description |
|---|---|
| topol.top | System topology |
| topol_Protein_chain_A–F.itp | Per-chain topologies (included by topol.top) |
| posre_Protein_chain_A–F.itp | Position restraints per chain |
| solv.gro | Solvated system |
| ions.mdp | Parameters for genion step |
| ions.tpr | Input for genion step |
| em.mdp | Energy minimisation parameters |
| em.tpr | Energy minimisation input |
| em.gro | Minimised structure (water and ions included), starting point for all NVT runs |

## Setup order
pdb2gmx → editconf → solvate → genion → energy minimisation (em)

## Notes
- Keep topol.top and all .itp files in the same folder.
- r1 and r2 replicates differ only in the NVT gen-seed (r1 = 111, r2 = 222).
- Run-specific files are in the 300K_thermal, 373K_thermal and 373K_shear folders.
