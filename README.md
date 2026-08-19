# PRISM

## Projected Residue Interaction-Space Mapper

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.20111879.svg)](https://doi.org/10.5281/zenodo.20111879)

![GitHub last commit](https://img.shields.io/github/last-commit/thorfriisphd-rgb/ibam-grammar-engine)
![GitHub repo size](https://img.shields.io/github/repo-size/thorfriisphd-rgb/ibam-grammar-engine)

**Structure-based evolutionary projection of protein interaction grammar**

PRISM (Projected Residue Interaction-Space Mapper; formerly the **IBAM Grammar Engine**) is a reproducible analysis pipeline developed to identify conserved interaction chemistry across deeply divergent protein homologues.

PRISM was developed for analysis of C12orf29/IBAM (*In Between Actin and Myosin*) interactions with the myosin tail (MyhT) across a 26-taxon reference panel spanning approximately one billion years of evolutionary divergence.

---

## Overview

Conventional multiple sequence alignment of C12orf29 orthologues yields approximately 30–40% pairwise identity. This is sufficient to establish homology, but limits confident inference about conservation of individual functional residues.

PRISM approaches the problem from a different direction.

Rather than asking whether the same amino acid occupies the same position in a primary-sequence alignment, PRISM asks whether structurally corresponding residues repeatedly participate in the same three-dimensional interaction — and whether those residues retain similar physicochemical character despite sequence divergence.

The pipeline integrates three analytical stages:

1. **Dynamic contact decoding** from molecular dynamics trajectories
2. **Evolutionary projection** of interaction residues into a common positional framework
3. **Chemical grammar analysis** of the projected interaction sites

The resulting projected FASTA alignment therefore has a different interpretation from a conventional sequence alignment. Its columns represent structurally recurrent interaction sites rather than simply linear sequence positions.

Conservation detected by PRISM reflects constraint on the **three-dimensional interaction surface**, rather than conservation of primary sequence alone.

---

## Taxonomic Panel

The reference analysis comprises 26 taxa spanning major eukaryotic lineages together with a bacterial outgroup.

| Clade                      | Representative taxa                                                                  |
| -------------------------- | ------------------------------------------------------------------------------------ |
| Bacteria (outgroup)        | *Hahella chejuensis*                                                                 |
| Heterolobosea              | *Naegleria gruberi*, *Willaertia magna*                                              |
| Euglenozoa                 | *Euglena longa*, *Eutreptiella gymnastica*                                           |
| Cnidaria                   | *Clytia hemispherica*, *Podocoryna carnea*                                           |
| Annelida                   | *Eisenia fetida*, *Lamellibrachia satsuma*                                           |
| Mollusca                   | *Haliotis asinina*, *Magallana angulata*, *Octopus vulgaris*                         |
| Brachiopoda                | *Lingula anatina*                                                                    |
| Phoronida                  | *Phoronis australis*                                                                 |
| Myriapoda                  | *Henia illyrica*, *Lithobius forficatus*                                             |
| Onychophora                | *Euperipatoides rowelli*                                                             |
| Chordata — Tunicata        | *Ciona intestinalis*, *Salpa thompsoni*                                              |
| Chordata — Cephalochordata | *Branchiostoma floridae*                                                             |
| Chordata — Vertebrata      | *Lampetra planeri*, *Myxine glutinosa*, *Mus musculus*, *Ovis aries*, *Homo sapiens* |
| Hemichordata               | *Saccoglossus kowalevskii*                                                           |

The pipeline is not restricted to this reference panel and can accommodate additional taxa where the required structural, trajectory and sequence inputs are available.

---

## Pipeline Architecture

```text
AF3-predicted protein complexes
            │
            ▼
Molecular dynamics trajectories
            │
            ▼
┌──────────────────────────────────────────────┐
│  1. DYNAMIC CONTACT DECODING                 │
│                                              │
│  PRCO contact extraction                     │
│  └── persistent interface residues           │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│  2. EVOLUTIONARY PROJECTION                  │
│                                              │
│  MAFFT sequence alignment                    │
│          +                                   │
│  structurally defined contact watchlists     │
│          │                                   │
│          ▼                                   │
│  projected FASTA                             │
│          │                                   │
│          ▼                                   │
│  dual-gate filtering                         │
│  ├── phylogenetic occupancy                  │
│  └── chemical dominance                      │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│  3. CHEMICAL GRAMMAR ANALYSIS                │
│                                              │
│  H / P / B / A class frequencies             │
│  ├── per-column composition                  │
│  ├── occupancy and dominance                 │
│  ├── entropy                                 │
│  ├── projected sequence logos                │
│  └── stacked chemical barcodes               │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
          tables + figures + provenance
```

The complete production workflow is orchestrated by `run_pipeline.sh`.

Two principal working scripts perform the underlying analysis:

* `PDWLS.sh` — **PRCO Decode WebLogo Script**
* `BARWLS.sh` — **Barcode WebLogo Script**

These scripts are retained as working components of the validated production pipeline.

---

## Dual-Gate Filtering

The raw projected alignment contains positions with differing levels of phylogenetic representation and chemical coherence. PRISM therefore applies two independent filters to identify the conserved interaction core.

### Occupancy gate

A projected position must be represented across a specified proportion of the taxonomic panel.

This asks whether the interaction site is recurrent across a sufficiently broad evolutionary sample.

### Chemical dominance gate

Among residues occupying that projected position, a specified proportion must belong to the same broad physicochemical class:

* **H** — hydrophobic
* **P** — polar
* **B** — basic
* **A** — acidic

This asks whether the chemical character of the interaction site has been retained even where amino-acid identity has changed.

A position must satisfy **both gates** to enter the filtered interaction grammar.

The reference analysis evaluates multiple gating regimes:

```text
50/85
60/90
70/90
```

where the first value specifies the occupancy threshold and the second specifies the chemical-dominance threshold.

Comparison across gating regimes provides a direct test of threshold robustness. Increasing stringency progressively contracts the projected cassette while allowing the stability of its conserved core to be assessed.

---

## Quick Start

### Prerequisites

PRISM requires:

* **GROMACS** ≥ 2025.x (`gmx` available in `PATH`)
* **Python** ≥ 3.8
* **NumPy**
* **pandas**
* **MDAnalysis**
* **matplotlib**
* **MAFFT**
* **WebLogo** 3.7+
* **pdf2svg** for WebLogo SVG generation

On Debian/Ubuntu:

```bash
sudo apt install mafft pdf2svg
```

### Clone the Repository

```bash
git clone https://github.com/thorfriisphd-rgb/ibam-grammar-engine.git
cd ibam-grammar-engine
```

The pipeline scripts are self-contained within the repository and require no separate PRISM package installation.

---

## Reference Dataset

The molecular-dynamics reference dataset is archived on Zenodo:

**IBAM Grammar Engine reference dataset — Version 2**
Published 10 May 2026
DOI: **10.5281/zenodo.20111879**

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.20111879.svg)](https://doi.org/10.5281/zenodo.20111879)

Download and extract the archive, then provide its root directory to PRISM using `--data-root`.

---

## Running the Pipeline

### Full reference run

```bash
./run_pipeline.sh \
  --data-root /path/to/zenodo_deposit \
  --samples /path/to/zenodo_deposit/samples.tsv \
  --gates "50/85,60/90,70/90" \
  --label C12_26taxon \
  --barcode
```

### Single gate, no barcode

```bash
./run_pipeline.sh \
  --data-root /path/to/zenodo_deposit \
  --samples /path/to/zenodo_deposit/samples.tsv \
  --gates "60/90" \
  --label C12_60-90_only
```

### Dry run

```bash
./run_pipeline.sh \
  --data-root /path/to/zenodo_deposit \
  --samples /path/to/zenodo_deposit/samples.tsv \
  --dry-run
```

### Key Flags

| Flag          | Description                                              |
| ------------- | -------------------------------------------------------- |
| `--data-root` | Root directory containing the molecular-dynamics dataset |
| `--samples`   | Path to the `samples.tsv` taxon registry                 |
| `--gates`     | Comma-separated occupancy/chemistry gate combinations    |
| `--label`     | Run identifier used in output directories and manifests  |
| `--barcode`   | Enable chemical barcode analysis                         |
| `--dry-run`   | Validate configuration without executing                 |

---

## `samples.tsv` Format

Taxa are defined in a tab-separated registry containing one row per taxon.

Example:

```text
Branchiostoma   /path/to/data/Taxon_MDS_data/Branchiostoma   1   309   310   381
```

Columns:

```text
SAMPLE  WORKDIR  C12_START  C12_END  MYH_START  MYH_END
```

Each `WORKDIR` must contain:

```text
md.gro
md.tpr
md.xtc
```

Lines beginning with `#` are ignored.

The canonical registry uses portable paths relative to the supplied data root.

---

## Output Structure

Each run creates a timestamped results directory:

```text
results/<label>_<timestamp>/
├── prco_decode/
│   └── per-taxon contact tables
│
├── alignment/
│   └── MAFFT alignments
│
├── projection/
│   └── projected FASTA files
│
├── logo/
│   └── projected sequence logos
│
├── barcode/
│   └── barcode_*/
│       ├── BAR_*_chemcomp.tsv
│       ├── BAR_*_percol.tsv
│       ├── BAR_*_summary.tsv
│       ├── BAR_*_stacked.png
│       ├── mini_results_table.tsv
│       └── MANIFEST.txt
│
└── logs/
    └── per-taxon processing logs
```

Run-level provenance is recorded in:

```text
manifest/
├── checksums_*.sha256
├── manifest_*.json
└── versions_*.txt
```

These records provide traceability from the input trajectories through to the final projected interaction grammar.

---

## Scripts

| Script                                         | Purpose                                                       |
| ---------------------------------------------- | ------------------------------------------------------------- |
| `run_pipeline.sh`                              | Master production orchestrator                                |
| `scripts/PDWLS.sh`                             | PRCO decoding, evolutionary projection and WebLogo generation |
| `scripts/BARWLS.sh`                            | Chemical barcode analysis and rendering                       |
| `scripts/prco_decode_cli.py`                   | Time-integrated contact decoding from MD trajectories         |
| `scripts/project_multiple_taxa.py`             | Contact-watchlist mapping and projected FASTA generation      |
| `scripts/chemical_barcode_analyzer_v2.py`      | Physicochemical class-frequency analysis                      |
| `scripts/complementary_pattern_analyzer_v2.py` | Optional complementary-pattern sanity check                   |
| `scripts/plot_barcode_stacked.py`              | Stacked chemical-barcode rendering                            |

The scripts in `scripts/` are retained working components of the production analysis. The top-level orchestrator handles directory construction, parameter passing, logging and provenance without altering their analytical logic.

---

## Molecular-Dynamics Parameters

The reference IBAM analysis used standardised molecular-dynamics conditions:

| Parameter      | Value            |
| -------------- | ---------------- |
| Force field    | CHARMM36-jul2022 |
| Water model    | TIP3P            |
| Electrostatics | Cutoff, 1.0 nm   |
| Production MD  | 10 ns standard   |
| Temperature    | 310 K            |
| Contact cutoff | 4.5 Å            |

Longer 50- and 100-ns simulations were additionally generated for selected analyses.

The methodological basis for the molecular-dynamics protocol, including the comparison of cutoff, Reaction Field and PME electrostatics using the GCN4 control system, is described in the accompanying manuscript. Those controls informed the simulation protocol and are not part of the PRISM analytical pipeline.

---

## Environment Variables

The production scripts expose several lower-level controls:

| Variable            | Default | Effect                                    |
| ------------------- | ------- | ----------------------------------------- |
| `MAKE_PROTEIN_ONLY` | `1`     | Generate `protein_only.gro` using GROMACS |
| `MAKE_PROT_XTC`     | `0`     | Also generate `md_protein.xtc`            |
| `OCC_THR`           | `0.05`  | Minimum occupancy for watchlist inclusion |
| `P1_THR`            | none    | Optional partner-1 occupancy threshold    |
| `P2_THR`            | none    | Optional partner-2 occupancy threshold    |

These controls normally do not need to be changed when reproducing the reference analysis.

**Release check:** the precise relationship between `OCC_THR=0.05` and the contact-occupancy criterion should be confirmed directly from the frozen production scripts before this README is committed. No threshold should be documented from historical notes where the implementation can provide the authoritative answer.

---

## Reproducibility

PRISM was designed for end-to-end reproducibility.

Each production run records:

* SHA256 checksums
* analysis parameters
* software versions
* timestamps
* per-taxon processing logs
* projected FASTA outputs
* chemical-composition tables
* run manifests

To reproduce the reference analysis:

1. Clone this repository.
2. Download Version 2 of the reference dataset from Zenodo.
3. Extract the dataset.
4. Point `--data-root` to the extracted `zenodo_deposit`.
5. Supply its canonical `samples.tsv`.
6. Run the full reference command shown above.
7. Compare the resulting outputs and checksums with the archived reference analysis.

---

## Troubleshooting

### WebLogo fails because `pdf2svg` is missing

WebLogo requires the external `pdf2svg` utility for SVG generation.

Debian/Ubuntu:

```bash
sudo apt install pdf2svg
```

macOS with Homebrew:

```bash
brew install pdf2svg
```

### `WORKDIR not found`

Run the pipeline from the repository root and provide the extracted Zenodo root explicitly:

```bash
--data-root /path/to/zenodo_deposit
```

The canonical `samples.tsv` uses portable paths rather than machine-specific absolute paths.

---

## Data Availability

* **Code:** this repository, released under the MIT License
* **Reference dataset:** Version 2, published 10 May 2026
* **Zenodo DOI:** **10.5281/zenodo.20111879**
* **Contents:** AF3-predicted structures and GROMACS molecular-dynamics data supporting the reference IBAM analysis

The data are maintained separately from the GitHub repository so that the code repository remains lightweight while the complete analysis remains reproducible.

---

## Citation

If you use PRISM or the accompanying reference dataset, please cite:

> Friis TE. (2026). *C12orf29 encodes IBAM (In Between Actin and Myosin), a sarcomeric protein with a conserved actomyosin binding grammar.* Manuscript in preparation.

The archived reference dataset is available at DOI **10.5281/zenodo.20111879**.

---

## License

MIT License. See `LICENSE` for details.

---

## Author

**Thor Einar Friis, PhD**

[![ORCID](https://img.shields.io/badge/ORCID-0000--0002--4132--4912-A6CE39?logo=orcid\&logoColor=white)](https://orcid.org/0000-0002-4132-4912)

Independent researcher, Bodø, Norway.
PhD in Molecular Biology, Queensland University of Technology (QUT).

PRISM forms part of a reproducible computational framework developed for the investigation of IBAM/C12orf29 and the MyhT–IBAM major-groove interface.
