# Vicilin trimer: common setup files

Files shared by all vicilin trimer runs (300 K thermal, 373 K thermal, 373 K shear; replicates r1 and r2). Run-specific files (NVT, NPT and production .mdp/.tpr) are in each run's folder.

## System
- Protein: pea vicilin trimer, 3 chains (A–C, 415 residues each), AlphaFold3 model (starting structure in PDB and CIF format: [structures folder](https://github.com/malikahmedhassanawan/Legumin-Vicilin-Oligomer-Thermal-Shear-MD/tree/main/structures))
- Force field: <from topol.top>
- Water model: <from topol.top>
- Box: <from last line of em.gro>
- Solvent: <number of SOL in topol.top> water molecules
- Ions: Na+/Cl- (<NA count> Na+, <CL count> Cl-), neutralised
- Total atoms: 172,243
- GROMACS version: <from topol.top header>

## Files
| File | Description |
|---|---|
| topol.top | System topology |
| topol_Protein_chain_A–C.itp | Per-chain topologies (included by topol.top) |
| posre_Protein_chain_A–C.itp | Position restraints per chain |
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
- Run-specific files are in the 300K_thermal, 373K_thermal and 373K_thermal+shear folders.
