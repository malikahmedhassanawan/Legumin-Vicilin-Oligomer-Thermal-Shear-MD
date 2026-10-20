# Legumin-Vicilin-Oligomer-Thermal-Shear-MD

GROMACS simulations of the pea legumin hexamer and the vicilin trimer at 300 K, at 373 K, and at 373 K with the box sheared. Each condition was run twice for 1 ns. The repository holds the mdp files, the analysis and comparison scripts, and summary tables and figures.

## Background

Extruding plant protein into a meat-like texture exposes it to heat and shear at the same time. Here the questions are narrower: do the full oligomers change when heated from 300 K to 373 K, and does a gentle imposed shear at 373 K add anything on top of the heat? The runs are short on purpose. The aim was to see whether a shear signal shows up at all before spending time on longer runs.

## What the runs show

Everything below comes from 1 ns runs with two replicates per condition.

Heating legumin to 373 K has a clear effect. Backbone RMSD rises by 17 to 22 %, helix content falls by 5 to 6 %, hydrogen bonds inside the hexamer fall by 2 to 4 %, and hydrogen bonds between chains fall by 5 to 6 %. Rg and SASA go up a little (0.7 to 0.9 % and 1.5 to 2.5 %). The two replicates of each condition agree closely, so these changes sit well outside the replicate spread.

Vicilin responds less clearly. Helix falls by about 5 to 6 % and coil/turn rises by 2 to 3 %, and both thermal replicates agree on that. RMSD also rises, but the two thermal replicates are 11 % and 28 % above the 300 K mean, so the size of the increase is uncertain. Rg, hydrogen bonds in the trimer and strand content don't change.

Shear made no detectable difference in either protein. Every shear value falls inside, or overlaps, the range spanned by the two thermal replicates. Several features that looked like shear effects in replicate 1 were gone in replicate 2 (see [Results](#results)).

Residue-level difference plots are included, but with two replicates they are noisy. Anything that stands out in them is a guess to test with longer runs.

## Systems

| | Legumin (11S) | Vicilin (7S) |
|---|---|---|
| Oligomer | hexamer, 6 chains | trimer, 3 chains |
| Residues per chain | 496 | 415 |
| Total atoms, with solvent | 217,446 | 172,243 |
| Box | rhombic dodecahedron (xy-square), image distance about 14.6 nm | rhombic dodecahedron (xy-square), image distance about 13.55 nm |

The vicilin system has 19,941 protein atoms (6,647 per chain), 50,655 waters, 179 Na and 158 Cl. Box sizes are from the end of NPT. Each replicate's NPT run ends at a slightly different volume, so replicates have slightly different boxes (about 0.1 % apart in vicilin).

## Simulation design

There are six production runs per protein, twelve in all, each 1 ns.

| Condition | Replicates | Production ensemble | Thermostat groups |
|---|---|---|---|
| 300 K thermal | r1, r2 | NVT | Protein, Non-Protein |
| 373 K thermal | r1, r2 | NVT | Protein, Non-Protein |
| 373 K shear | r1, r2 | NVT plus box deformation | System (one group) |

Replicates r1 and r2 start from the same energy-minimised structure and differ only in the random seed. r1 uses `gen-seed = 111` in NVT and `ld-seed = 111` in production. r2 uses 222 for both. Within one replicate, the thermal and shear production runs start from the same end-of-NPT coordinates and velocities.

The stage settings below were checked against the vicilin files at 373 K.

| Stage | Length | Settings |
|---|---|---|
| NVT | 100 ps (50,000 steps) | position restraints (`-DPOSRES`), V-rescale with tau-t 0.1 ps, velocities generated at the target temperature, no pressure coupling |
| NPT | 500 ps (250,000 steps) | position restraints, V-rescale, C-rescale barostat (isotropic, tau-p 2.0 ps, compressibility 4.5e-5 bar⁻¹, 1 bar, `refcoord-scaling = com`), velocities carried over |
| Production, thermal | 1 ns (500,000 steps) | no restraints, V-rescale on Protein and Non-Protein, no pressure coupling, velocities carried over |
| Production, shear | 1 ns (500,000 steps) | as thermal, but V-rescale on the whole system and `deform` set |

All stages use the `md` integrator with a 2 fs step, H-bond constraints (LINCS), Verlet cutoffs, PME electrostatics, a 1.0 nm cutoff for both Coulomb and van der Waals, and `DispCorr = EnerPres`. Energies are written every 500 steps. In production, compressed coordinates are also written every 500 steps (1 ps), for the protein only (`compressed-x-grps = Protein`). The legumin runs use the same template. The full `.mdp` files for both proteins are in simulation setup.

## Shear

Shear is applied with the GROMACS box deformation option:

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


## Analysis

Each run is analysed by one script, and all trajectories hold protein atoms only. The script first checks that the trajectory has 1001 frames from 0 to 1000 ps with no gaps or repeats, and stops if not. It then makes the molecules whole, brings the chains together across the periodic boundary (handled for the triclinic cell), fits every frame on the backbone and keeps every fifth one (201 frames, 5 ps apart). Chains are found from the residue numbering. The script stops if the chain count or sizes are unexpected, or if any chain loses all neighbours within 0.5 nm in any frame.

| Metric | Method |
|---|---|
| Backbone RMSD | `gmx rms` against the first frame, per chain and for the whole oligomer |
| Rg | `gmx gyrate`, per chain and whole |
| SASA | `gmx sasa`, with each chain measured in the context of the whole oligomer, so chain values add up to the whole |
| H-bonds in the oligomer | `gmx hbond` over the whole protein, counting intra- and inter-chain bonds together |
| Inter-chain H-bonds | `gmx hbond` for each chain pair (3 for vicilin, 15 for legumin), summed |
| RMSF | C-alpha, `gmx rmsf` |
| Per-residue RMSD | C-alpha RMSD after fitting each chain on its own backbone |
| Per-residue H-bonds | custom: donor to acceptor ≤ 0.35 nm, hydrogen to acceptor ≤ 0.25 nm, angle ≤ 30 degrees |
| Secondary structure | `gmx dssp`, per chain, per residue and as composition |


## Results 

Each cell is the percentage change of a run's last-half mean relative to the mean of the two 300 K replicates. Exact values are in `results/*/csv/last_half_means_by_run.csv`.

### Legumin hexamer

| Run | RMSD | Rg | SASA | H-bonds | Inter-chain H-bonds | Helix | Strand | Coil/turn |
|---|---|---|---|---|---|---|---|---|
| 300 K r1 | −0.6 | −0.1 | −0.9 | −0.4 | −0.5 | −1.2 | −1.1 | +0.7 |
| 300 K r2 | +0.6 | +0.1 | +0.9 | +0.4 | +0.5 | +1.2 | +1.1 | −0.7 |
| 373 K r1 | +18.2 | +0.7 | +1.5 | −3.3 | −5.4 | −5.7 | +0.3 | +1.0 |
| 373 K r2 | +20.8 | +0.8 | +2.3 | −2.4 | −6.5 | −6.0 | +0.8 | +0.8 |
| 373 K shear r1 | +16.8 | +0.9 | +2.5 | −3.0 | −1.7 | −5.4 | −0.3 | +1.2 |
| 373 K shear r2 | +21.8 | +0.7 | +2.4 | −4.1 | −5.8 | −5.5 | −0.8 | +1.4 |

### Vicilin trimer

| Run | RMSD | Rg | SASA | H-bonds | Inter-chain H-bonds | Helix | Strand | Coil/turn |
|---|---|---|---|---|---|---|---|---|
| 300 K r1 | −5.6 | −0.1 | −0.8 | +0.9 | +6.3 | +0.2 | +0.4 | −0.4 |
| 300 K r2 | +5.6 | +0.1 | +0.8 | −0.9 | −6.3 | −0.2 | −0.4 | +0.4 |
| 373 K r1 | +27.7 | +0.1 | −0.3 | −0.2 | −0.1 | −5.5 | −0.1 | +2.1 |
| 373 K r2 | +11.3 | −0.4 | −2.9 | −0.0 | +4.7 | −5.9 | −1.0 | +3.0 |
| 373 K shear r1 | +20.0 | −0.0 | −2.6 | +0.0 | +0.0 | −0.8 | +0.5 | −0.1 |
| 373 K shear r2 | +34.0 | +0.1 | −1.7 | −1.4 | +6.9 | −8.9 | −1.9 | +4.8 |

### Reading the legumin table

At 300 K each replicate sits within about 1.2 % of the 300 K mean on every metric, which is what makes the 373 K changes in RMSD, helix and hydrogen bonds stand out. Shear doesn't separate from thermal. Inter-chain hydrogen bonds are the clearest case: shear r1 (−1.7 %) looked like a shear effect, but shear r2 (−5.8 %) matches the thermal runs (−5.4 and −6.5 %). Helix is the other one worth a look: the shear runs lose 5.4 and 5.5 %, the thermal runs 5.7 and 6.0 %.

### Reading the vicilin table

Vicilin is noisier. At 300 K the two replicates already differ by about 11 points in RMSD and 13 points in inter-chain hydrogen bonds. The helix loss and coil gain at 373 K hold up in both thermal replicates (−5.5 and −5.9 % helix, +2.1 and +3.0 % coil). RMSD, SASA and inter-chain hydrogen bonds differ between the two thermal replicates as much as they differ between thermal and shear. The shear replicates disagree with each other on helix (−0.8 and −8.9 %) and RMSD (+20.0 and +34.0 %), so shear leaves no consistent mark.

A SASA drop under shear looked likely in vicilin when only thermal r1 was available. Thermal r2 turned out to be as low as the shear runs, and the two thermal replicates differ by about 13 nm², more than thermal and shear differ.

### Across both proteins

Heating to 373 K lowers helix content by a similar relative amount in both (about 5 to 6 %), and the drop shows up in all four thermal replicates. Several RMSD curves are still rising at 1 ns, so last-half RMSD partly measures how far each run has drifted, not a settled value.

### Per-residue differences

The residue plots (373 K minus 300 K, and shear minus 373 K thermal) are in `results/*/figures/06_residue_deltas.png`. Heating raises C-alpha RMSF across most of the sequence in both proteins, most at the termini and at a few loops. In legumin, both shear runs seem to lower RMSF over roughly residues 290 to 360. That region is the one shear feature that looks worth testing with longer runs. The rest of the per-residue shear differences (SASA, helix, strand, hydrogen bonds) show no consistent pattern.

## Limitations

- There are two replicates per condition. Differences can only be called separated or not, effects smaller than the replicate gap can't be resolved, and two pairs can look different by chance.
- Runs are 1 ns, and several RMSD curves are still rising. Slow processes such as unfolding or rearrangement of the oligomer are out of reach.
- The shear is gentle: a strain rate of 0.25 ns⁻¹ gives a total strain of only 0.25 in 1 ns. Stronger or longer shear is untested.
- The thermostat grouping differs between thermal and shear runs (see [Shear](#shear)).
- 

## Repository Structure

- **structures/** — AlphaFold3 output structures (.cif) and processed dimer PDBs
- **alphafold_predictions/** — AlphaFold confidence (pLDDT) and chain-colored visualizations
- **simulation_setup/** - GROMACS .mdp parameter files for both proteins (complete: em, ions, nvt, npt, md for each conditions) 
- **simulations/** — Raw analysis output (.xvg) per protein, per condition
- - **analysis/** — `legumin/` and `vicilin/` hold per-protein plots (RMSD, Rg, SASA, H-bonds, RMSF) across all conditions; `comparison of different conditions of legumin and vicilin
comparisons; `summary_table.csv`/`.md` condenses all conditions into one numeric table
- **notebooks/** — Full analysis pipeline (Colab-executed)
- **docs/methods.md** — Detailed methodology, software versions, parameters
- 
 ## Methods Summary

| Stage | Tool | Details |
|---|---|---|
| Structure prediction | AlphaFold3 | Homodimer models for both proteins |
| MD engine | GROMACS 2026.3 (conda-forge, CUDA) | GPU-accelerated (Colab T4) |
| Force field | AMBER99SB-ILDN | |
| Water model | SPC/E | |
| Conditions | 300K, 400K × 200ps, 500ps | 4 conditions per protein |
| Analysis | RMSD, Rg, SASA, H-bonds, RMSF (chain-split) | via `gmx rms`, `gmx gyrate`, `gmx sasa`, `gmx hbond`, `gmx rmsf` |
| Visualization | PyMOL, Matplotlib | Structure rendering and trajectory plots |

Full parameter details, exact command-line invocations, and version numbers are documented in [`docs/methods.md`](docs/methods.md).

## Reproducing This Work

All simulations were run in Google Colab using free-tier GPU resources, making this pipeline accessible without dedicated HPC infrastructure. The full notebook pipeline is provided in [`notebooks/`](notebooks/), covering:

1. Structure prediction and preprocessing
2. GROMACS simulation setup and production runs
3. Trajectory analysis and comparative visualization

## Author

**Malik Ahmed Hassan Awan**
BS (Hons) Food Science and Technology, Bahauddin Zakariya University, Multan, Pakistan
Contact: varietyhavenllc@gmail.com

---

*This work is part of an ongoing research effort connecting computational protein science to food security challenges in South Punjab, Pakistan, using locally available legume crops as a model system for sustainable protein engineering.*
