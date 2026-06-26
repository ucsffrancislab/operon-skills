---
name: TCGA-dataset
description: TCGA (The Cancer Genome Atlas) is an NCI-supported repository containing multi-omics data (RNA-seq, DNA methylation, mutation, copy number, clinical survival outcomes) across over 20,000 primary cancer and matched normal samples for 33 cancer types. Reach for this skill whenever a task touches the TCGA glioma cohort (GBM + LGG) on the francislab cluster — covariates, IDH/1p19q/TERT/MGMT/WHO subgroupings, survival outcomes, raw Affy6 genotype, imputed genotype, PGS scores, or AGS-IPS-TCGA harmonized analyses. Use when the user says "TCGA", "TCGA glioma", "TCGA GBM", "TCGA LGG", "GBMLGG", "WTCCC", "Affy6", or references TCGA case_submitter_ids (e.g. TCGA-02-0001). Provides canonical file paths and schema notes so analyses don't drift across sessions.
---


Make this more general to all of our TCGA data but then include specific


# TCGA glioma (GBM + LGG)

The TCGA glioma cohort assembled on the francislab cluster covers GBM (Glioblastoma multiforme) and LGG (Lower Grade Glioma) cases. Genotyping is from the TCGA Affymetrix 6.0 arrays merged with WTCCC controls.

All paths below are on the C4 cluster (`ssh:c4-log1-dev4`). Files under `/francislab/data1/refs/TCGA/` are read-only — do downstream work in your own working directory.

## Canonical files

| Purpose | Path |
|---|---|
| **Covariates (canonical)** | `/francislab/data1/refs/TCGA/TCGA.Glioma.metadata.tsv` |
| **Tissue source site codes** | `/francislab/data1/refs/TCGA/TCGA.Tissue_Source_Site_Codes.tsv` |
| **Sample type codes** | `/francislab/data1/refs/TCGA/TCGA.Sample_Type_Codes.tsv` |
| **GBM site codes** | `/francislab/data1/refs/TCGA/TCGA.GBM_codes.txt` |
| **LGG site codes** | `/francislab/data1/refs/TCGA/TCGA.LGG_codes.txt` |

The canonical covariate file is 1115 cases (`case_submitter_id` key), covering GBM and LGG together with project labels.

## Raw genotype (hg19)

| Cohort | PLINK prefix |
|---|---|
| TCGA glioma + WTCCC controls (Affy6) | `/francislab/data1/raw/20210223-TCGA-GBMLGG-WTCCC-Affy6/TCGA_WTCCC_for_QC.{bed,bim,fam}` |

This is a **merged** TCGA + WTCCC PLINK fileset for QC; subjects from both studies share the same `.fam`. Filter on FID/IID prefix or pull a separate sample manifest when you need TCGA-only.

## Imputed genotype

TCGA was imputed to the same UMich 1000 Genomes hg19 panel as AGS and IPS. The exact directory for this build is in the AGS/IPS-CIDR-ONCO-I370-TCGA working tree:

- `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/` (browse for the latest TCGA-imputed subdirectory)

Conventions (consistent across all three cohorts):
- Chromosome names: `1`, `2`, … (**no** `chr` prefix).
- INFO field is Minimac4 `R2`; use `R2 ≥ 0.8` as the standard well-imputed cutoff.

## Imputed PGS

| Provider | Path |
|---|---|
| UMich Imputation Server | `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/20250724-pgs/pgs-tcga-hg19/scores.txt.gz` |
| Locally | `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/20260410-compute_PGS_hg19/pgs-calc-scores/tcga/scores.txt.gz` |

## Imputed HLA

| Path |
|---|
| `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/20250725-hla/hla-tcga-hg19/chr6.dose.vcf.gz` |


## Schema notes (canonical covariates)

| Column | Meaning |
|---|---|
| `case_submitter_id` | TCGA barcode (e.g. `TCGA-02-0001`) — primary key |
| `project_id` | `TCGA-GBM` or `TCGA-LGG` |
| `primary_diagnosis` | Free-text diagnosis (e.g. "Glioblastoma", "Oligodendroglioma, anaplastic") |
| `race`, `ethnicity`, `gender` | Free-text strings (e.g. `white`, `not hispanic or latino`, `female`) |
| `RE_names` | Repeat-element annotation (study-specific) |
| `IDH` | `MUT` / `WT` / `NA` (string — not 0/1 as in AGS) |
| `x1p19q` | `codel` / `non-codel` / `NA` |
| `TERT` | `MUT` / `WT` / `NA` |
| `IDH_1p19q_status` | Combined string label (e.g. `IDH-WT:1p19q-non-codel`) |
| `WHO_groups` | Combined WHO grouping string |
| `Triple_group` | IDH + 1p19q + TERT triple class string |
| `Tissue_sample_location` | Source institution |
| `MGMT` | `Methylated` / `Unmethylated` / `NA` |
| `Age` | Years |
| `Survival_months` | Months (decimal) — **note units differ from AGS/IPS** |
| `Vital_status` | 1=deceased, 0=alive (verify against your downstream needs) |

## Cross-cohort harmonization (TCGA ↔ AGS/IPS)

TCGA's schema is **string-coded** rather than numeric, and several columns need conversion before merging:

| Concept | TCGA | AGS/IPS |
|---|---|---|
| IDH | `IDH` string (`MUT`/`WT`/`NA`) | `idhmut` (1/0/9) |
| 1p/19q | `x1p19q` string (`codel`/`non-codel`/`NA`) | `pq` (1/0/8/9) |
| Sex/gender | `gender` (`male`/`female`) | `sex` (M/F or 1/2) |
| Survival time | `Survival_months` (months) | `survdays` (days) |
| Vital status | `Vital_status` (0/1) | AGS `deceased` (A/D); IPS `deceased` (0/1) |
| WHO classification | `WHO_groups` string | AGS `who2016` / IPS `who2021` (numeric) |
| MGMT | `MGMT` string | IPS `mgmt`/`mgmtindex`; AGS canonical has none |

Standardize **before** joining. A typical harmonization step: convert TCGA `IDH` to `idhmut ∈ {0,1,NA}`, convert `Survival_months → survdays` (`*30.4375`), recode `gender → sex`.

## TCGA-glioma subsetting

To restrict to GBM or LGG cases by tissue source site, use the code lists:
- `TCGA.GBM_codes.txt` — site codes for glioblastoma multiforme
- `TCGA.LGG_codes.txt` — site codes for brain lower-grade glioma

These were generated from `TCGA.Tissue_Source_Site_Codes.tsv` (see the README in the TCGA refs github checkout). For most analyses, `project_id` in the metadata file (`TCGA-GBM` vs `TCGA-LGG`) is the simpler filter.

## Known gotchas

- TCGA molecular-marker columns (`IDH`, `x1p19q`, `TERT`, `MGMT`) are **strings**, not the numeric encodings used in AGS/IPS. Recode before any cross-cohort merge.
- `Survival_months` is in **months**, not days — multiply by 30.4375 to match AGS/IPS `survdays`.
- The raw PLINK fileset is TCGA **+ WTCCC controls** merged. Always confirm which subjects are TCGA before running TCGA-specific analyses.
- `Vital_status` encoding (0 vs 1) and string conventions in `race`/`ethnicity`/`gender` differ from AGS/IPS. Standardize to your project's convention up front.

