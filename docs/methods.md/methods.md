# Methods

This page holds the setup details behind the runs summarised in the main README: how the systems were built, how each stage was run, and how the trajectories were analysed. If anything here disagrees with an `.mdp` file in `simulation_setup/`, trust the file.

The runs follow up Legumin-Vicilin-Dimer-Thermal-MD, which used single chains and dimers at 300 K and 400 K.

## Software

- GROMACS 2026.3 (conda-forge, CUDA build), run on a free Colab T4 GPU
- Force field: AMBER99SB-ILDN
- Water model: SPC/E
- Structure prediction: AlphaFold3
- Rendering and plots: PyMOL and Matplotlib

All the code lives in Colab notebooks, which are in `notebooks/`. There is no separate `scripts/` folder.

## Starting structures

Both proteins were modelled with AlphaFold3: the legumin hexamer (6 chains of 496 residues) and the vicilin trimer (3 chains of 415 residues). The original models are in `structures/` as CIF and PDB files.

`alphafold_predictions/` holds PyMOL GIFs of the same models. One set is coloured by prediction confidence. In the other, every chain has its own colour (6 colours for legumin, 3 for vicilin).

## System construction

Each system went through the same steps: `pdb2gmx` (with `-merge no`, so every chain gets its own topology file), `editconf`, `solvate`, `genion`, then energy minimisation. The `em.mdp` and `ions.mdp` files are identical for both proteins and every run.

| | Legumin (11S) | Vicilin (7S) |
|---|---|---|
| Oligomer | hexamer, chains A–F | trimer, chains A–C |
| Residues per chain | 496 | 415 |
| Water molecules | 56,682 | 50,655 |
| Ions | 231 Na⁺, 195 Cl⁻ | 179 Na⁺, 158 Cl⁻ |
| Total atoms | 217,446 | 172,243 |
| Box | rhombic dodecahedron (xy-square) | rhombic dodecahedron (xy-square) |
| Box size | 14.504 × 14.504 × 10.256 nm at the end of minimisation; image distance about 14.6 nm after NPT | image distance about 13.55 nm after NPT |

Vicilin has 19,941 protein atoms, 6,647 per chain. Each replicate's NPT run ends at a slightly different volume, so the replicate boxes differ a little (about 0.1 % in vicilin).

The topology, per-chain `.itp` files, position-restraint files and the `.gro`/`.tpr` files from setup are in each protein's `Common files` folder. Keep `topol.top` and the `.itp` files together, because the topology finds them by relative path.

## Simulation design

There are six production runs per protein and twelve in total, each 1 ns.

| Condition | Replicates | Production ensemble | Thermostat groups |
|---|---|---|---|
| 300 K thermal | r1, r2 | NVT | Protein, Non-Protein |
| 373 K thermal | r1, r2 | NVT | Protein, Non-Protein |
| 373 K shear | r1, r2 | NVT plus box deformation | System (one group) |

Both replicates start from the same energy-minimised structure and differ only in the random seed. Replicate 1 uses `gen-seed = 111` in NVT and `ld-seed = 111` in production. Replicate 2 uses 222 for both. Within a replicate, the thermal and shear production runs start from the same end-of-NPT coordinates and velocities, and the 373 K thermal and shear runs share the same NVT and NPT files.

## Stage settings

These were checked against the vicilin files at 373 K. The legumin runs use the same template.

| Stage | Length | Settings |
|---|---|---|
| NVT | 100 ps (50,000 steps) | Position restraints (`-DPOSRES`), V-rescale with tau-t 0.1 ps, velocities generated at the target temperature, no pressure coupling |
| NPT | 500 ps (250,000 steps) | Position restraints, V-rescale, C-rescale barostat (isotropic, tau-p 2.0 ps, compressibility 4.5e-5 bar⁻¹, 1 bar, `refcoord-scaling = com`), velocities carried over |
| Production, thermal | 1 ns (500,000 steps) | No restraints, V-rescale on Protein and Non-Protein, no pressure coupling, velocities carried over |
| Production, shear | 1 ns (500,000 steps) | As thermal, but V-rescale on the whole system and `deform` set |

Every stage uses the `md` integrator with a 2 fs step, H-bond constraints (LINCS), Verlet cutoffs, PME electrostatics, a 1.0 nm cutoff for both Coulomb and van der Waals, and `DispCorr = EnerPres`. Energies are written every 500 steps. In production, compressed coordinates are also written every 500 steps (1 ps), for the protein only (`compressed-x-grps = Protein`).

## Shear

Shear uses the GROMACS box deformation option:

```
deform = 0 0 0 0.003644 0 0     ; legumin (nm/ps)
deform = 0 0 0 0.003387 0 0     ; vicilin (nm/ps)
```

The fourth value is the x-component of the second box vector, so the box tilts in the xy plane at a constant rate.

| | Legumin | Vicilin |
|---|---|---|
| Tilt rate | 3.644 nm/ns | 3.387 nm/ns |
| Box length | 14.577 nm | 13.559 nm |
| Strain rate | 0.25 ns⁻¹ (2.5 × 10⁸ s⁻¹) | 0.25 ns⁻¹ (2.5 × 10⁸ s⁻¹) |
| Tilt after 1 ns | 3.644 nm | 3.387 nm |
| Strain after 1 ns | 0.25 | 0.25 |

The tilt after 1 ns is well under half the box length, so GROMACS never flips the box. The last box line of each shear run ends in the expected tilt (3.644 or 3.387).

This is an imposed deformation of the periodic cell, not an applied stress, and no stress response was analysed. Both proteins get the same strain rate, so differences between them don't come from different shear.

The shear runs use one thermostat group and the thermal runs use two. No shear effect turned up, so the mismatch didn't change any conclusion, but a follow-up should use the same grouping for both.

## Restarted runs

The shear r1 runs for both proteins were restarted from a checkpoint and the parts joined afterwards.

Vicilin shear r2 stopped at 259.68 ps, although its xtc and edr files had reached 274 ps. It was resumed from the checkpoint and the first part was cut at 259 ps. The combined files were checked to run from 0 to 1000 ps at 1 ps spacing, with no gaps.

## Analysis

Each run is analysed in a Colab notebook, and all trajectories hold protein atoms only. The notebook first checks that the trajectory has 1001 frames from 0 to 1000 ps, with no gaps or repeats, and stops if it doesn't. It then makes the molecules whole, brings the chains together across the periodic boundary (handled for the triclinic cell), fits every frame on the backbone, and keeps every fifth frame (201 frames, 5 ps apart).

Chains are found from the residue numbering. The notebook stops if the chain count or chain sizes are unexpected, or if any chain loses all its neighbours within 0.5 nm in any frame.

| Metric | Method |
|---|---|
| Backbone RMSD | `gmx rms` against the first frame, per chain and for the whole oligomer |
| Rg | `gmx gyrate`, per chain and whole |
| SASA | `gmx sasa`, with each chain measured in the context of the whole oligomer, so chain values add up to the whole |
| H-bonds in the oligomer | `gmx hbond` over the whole protein, counting intra- and inter-chain bonds together |
| Inter-chain H-bonds | `gmx hbond` for each chain pair (3 for vicilin, 15 for legumin), summed |
| RMSF | C-alpha, `gmx rmsf` |
| Per-residue RMSD | C-alpha RMSD after fitting each chain on its own backbone |
| Per-residue H-bonds | Custom: donor to acceptor ≤ 0.35 nm, hydrogen to acceptor ≤ 0.25 nm, angle ≤ 30 degrees |
| Secondary structure | `gmx dssp`, per chain, per residue and as composition |

"Last-half mean" means the mean over 0.5 to 1.0 ns. Error bars in the bar charts are the standard deviation over time within that half, not the spread between replicates.

For replicate comparisons, a difference between two conditions counts as separated when the two replicates of one condition don't overlap with the two of the other. With two replicates this is a rough check and not a statistical test. Percentage changes in the heatmaps are relative to the mean of the two 300 K replicates.

## Where the files are

| Folder | Contents |
|---|---|
| `simulation_setup/<protein>/Common files/` | Topology, `.itp` files, `.gro` files, `em` and `ions` files, README |
| `simulation_setup/<protein>/<condition>/Run_X/` | NVT, NPT and production `.mdp`/`.tpr` files for that run |
| `simulations/` | `.xvg` data files for each run |
| `analysis/<protein>/` | Per-run figures; `comparison/csv` and `comparison/figures` hold the all-runs summaries |
| `structures/` | Original AlphaFold3 models (CIF and PDB) |
| `alphafold_predictions/` | PyMOL GIFs, by confidence and by chain |
| `notebooks/` | Colab notebooks with the setup, run and analysis code |
