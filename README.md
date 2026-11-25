# MSA methylation array workflow

This folder contains a Jupyter workflow for analyzing selected Illumina Infinium Methylation Screening Array (MSA) IDATs from GEO **GSE264438**, platform **GPL34394**. `msa_champ` is the project name; the workflow does not use ChAMP's 450K/EPIC IDAT importer for MSA. It imports the array with `minfi`, the MSA A1 manifest and its matching hg38 annotation, and uses `limma` for an optional differential probe analysis.

## What the notebook does

[`notebooks/msa_champ_analysis.ipynb`](notebooks/msa_champ_analysis.ipynb) runs these steps for the GSM accessions in the sample sheet:

1. Reads each GEO sample record and downloads only its red and green compressed IDAT files.
2. Checks that each selected sample has one matching red/green pair.
3. Imports the IDATs with `minfi` and the MSA A1 manifest/annotation packages.
4. Runs noob background and dye-bias correction, sample QC, and detection-p-value filtering.
5. Saves beta and M-value matrices, QC tables, plots, and provenance.
6. Optionally runs a limma differential probe analysis (DMP) when a valid study contrast is configured.

GEO's processed matrix was generated with SeSAMe v1.22. This notebook uses minfi noob, so its results are a separate processing branch and should not be expected to match GEO's values exactly.

## Quick start

### Requirements

- R supported by the installed Bioconductor release.
- Jupyter Notebook or JupyterLab.
- The R Jupyter kernel, IRkernel.
- Internet access for first-time package installation and for fetching GSM IDATs that are not already present locally.

If Jupyter is not installed yet, install Notebook with `python -m pip install notebook` (or install JupyterLab instead).

From an R console, install the kernel and analysis packages:

```r
install.packages(c("BiocManager", "IRkernel", "R.utils", "locfit"))
BiocManager::install(
  c(
    "minfi",
    "limma",
    "GEOquery",
    "IlluminaHumanMethylationMSAmanifest",
    "IlluminaHumanMethylationMSAanno.ilm10a1.hg38"
  ),
  ask = FALSE,
  update = FALSE
)
IRkernel::installspec(
  user = TRUE,
  name = "ir",
  displayname = "R (MSA workflow)"
)
```

The notebook's first code cell also installs missing analysis packages. The IRkernel setup is required before Jupyter can run an R notebook.

### Run the example

1. Open `notebooks/msa_champ_analysis.ipynb` in Jupyter and select the **R (MSA workflow)** kernel.
2. Run all cells from top to bottom. If `metadata/sample_sheet.csv` does not exist, the notebook uses [`metadata/sample_sheet_example.csv`](metadata/sample_sheet_example.csv).
3. The checked-in example selects four GEO Illumina Control DNA samples, GSM8217992–GSM8217995. It is intended to demonstrate IDAT import, preprocessing, QC, and PCA; DMP analysis is disabled for these controls.

To analyze a study cohort, copy [`metadata/sample_sheet_template.csv`](metadata/sample_sheet_template.csv) to `metadata/sample_sheet.csv`, fill in the sample rows, then rerun the notebook. The local `sample_sheet.csv` is ignored by Git.

## Sample sheet and DMP configuration

Provide one row per GSM accession. The template includes:

| Column | Purpose |
| --- | --- |
| `geo_accession` | GEO sample accession, for example `GSM8217992`. |
| `sample_id` | Unique analysis label; used to name matrix columns. |
| `condition` | Biological group used for the optional DMP model. |
| `include_in_dmp` | `TRUE` only for samples in the intended biological contrast. |
| `pair_id` | Optional matched-pair or repeated-measures identifier. |
| `batch` | Optional batch covariate. |
| `tissue` | Tissue/cell type description for QC and interpretation. |
| `description` | Free-text sample notes. |

Set the reference/comparison groups and optional pairing/covariate columns in the notebook configuration cell. The notebook skips DMP unless it finds at least two included samples in each configured group. That is a minimum software guard, not a recommendation for study power. Do not pool unrelated tissue types or cell lines into one case/control contrast.

## Outputs

The notebook writes files under this project folder:

- `data/raw/<GSM>/`: downloaded compressed IDATs and their uncompressed files.
- `data/intermediate/`: noob beta and M-value matrices as RDS files.
- `results/sample_qc.csv`, `results/detection_qc.csv`, and `results/sample_detection_summary.csv`.
- `results/analysis_sample_sheet_used.csv`, `results/analysis_provenance.txt`, and `results/sessionInfo.txt`.
- `results/dmp_limma.csv` only when a valid DMP model is configured and run.

The IDATs, local sample sheet, intermediate matrices, results, logs, and Jupyter checkpoints are excluded from Git by this folder's [`.gitignore`](.gitignore). The executed example plots and printed summaries are embedded in the notebook.

## Platform and interpretation notes

- Use the MSA A1 manifest for the GSE264438 design represented by this example. Illumina now lists a newer B1 revision; verify the manifest revision against the samples before mixing data processed under different designs.
- The workflow uses the Bioconductor `IlluminaHumanMethylationMSAmanifest` package and matching `IlluminaHumanMethylationMSAanno.ilm10a1.hg38` annotation package, based on Illumina's A1 design.
- GEO reports that SNP-probe readings in the deposited IDATs were de-identified. Do not use them for genetic identity, ancestry, or sex inference.
- Review sample QC and exclusions before interpreting results. The default detection-p cutoff and failed-probe fraction are configurable in the notebook and should be justified for the study.

## Repository files

- [`notebooks/msa_champ_analysis.ipynb`](notebooks/msa_champ_analysis.ipynb) — analysis and executed four-sample QC example.
- [`metadata/sample_sheet_template.csv`](metadata/sample_sheet_template.csv) — blank cohort input template.
- [`metadata/sample_sheet_example.csv`](metadata/sample_sheet_example.csv) — four-control QC example.
- [`DATA_SOURCES.md`](DATA_SOURCES.md) — GEO and manifest provenance links.
- [`.gitignore`](.gitignore) — excludes raw/local data and generated outputs.

## Data sources

See [`DATA_SOURCES.md`](DATA_SOURCES.md) for GEO records, manifest revision, annotation package, and processing provenance.
