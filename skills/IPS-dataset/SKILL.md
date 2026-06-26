---
name: IPS-dataset
description: The UCSF Immune Profile Study (IPS) is an ongoing longitudinal observational trial focused on analyzing the blood and immune systems of patients with newly diagnosed or recurrent central nervous system tumors, primarily gliomas. The primary goal is to establish blood-based immune biomarkers to predict patient survival and assess treatment response. Reach for this skill whenever a task touches the IPS (Immune Profiles Study) cohort — covariates, IDH/1p19q/MGMT/survival outcomes, raw or imputed genotype, PGS scores, CIDR/Francis_2520 genotyping, or AGS-IPS-TCGA harmonized analyses. Use when the user says "IPS", "Immune Profiles Study", "CIDR", "the IPS cases", "Francis 2520", "GDA TOP", or references subject_ids in the G### format. Provides canonical file paths on the francislab cluster and schema notes so analyses don't drift across sessions.
---


# IPS — Immune Profiles Study (UCSF)

UCSF glioma cohort with paired immune-profiling assays. Genotyping was performed at CIDR (Center for Inherited Disease Research) on the Illumina GDA TOP array. Many subjects have multiple timepoints (pre-/post-surgery; longitudinal).

All paths below are on the C4 cluster (`ssh:c4-log1-dev4`). Files under `/francislab/data1/refs/IPS/` are read-only — do downstream work in your own working directory.

## Canonical files

| Purpose | Path |
|---|---|
| **Covariates (canonical)** | `/francislab/data1/refs/IPS/CIDR_IPS_phenotype_2026-03-03_updated_vstatus.csv` |
| **Data dictionary** | `/francislab/data1/refs/AGS/Subject_Phenotype_Meth_Batch1_DD.csv` |
| **Full clinical (810 samples)** | `/francislab/data1/refs/IPS/14.Jan.2026_Patients_810samples_IPSdata_clinical.csv` |
| **Array locations (797 samples)** | `/francislab/data1/refs/IPS/20.April.2026_797samples_IPS_arraylocations_5-18-26hmh_v2.csv` |
| **PGS + ancestry PCs** | `/francislab/data1/refs/IPS/cidr_pgs.tsv` |
| **All IPS IDs** | `/francislab/data1/refs/IPS/All_IPS_ids.txt` |
| **Treatment covariates documentation** | `/francislab/data1/refs/IPS/Documentation_IPS_treatment_covariates_2025-10-10.csv` |
| **RAD + TMZ (treatment subset)** | `/francislab/data1/refs/IPS/IPS_CIDR_446_RAD_TMZ_2025-10-10.csv` |

The canonical covariate file is 446 subjects (`subject_id` key) and is **schema-harmonized with AGS** — the DD at `/francislab/data1/refs/AGS/Subject_Phenotype_Meth_Batch1_DD.csv` covers both (`study_source` 1=AGS, 2=IPS). The 810-sample clinical file is sample-level (one row per blood draw) rather than subject-level.

## Raw genotype (hg38)

| Platform | PLINK prefix |
|---|---|
| Illumina GDA TOP (CIDR) | `/francislab/data1/raw/20250813-CIDR/CIDR.{bed,bim,fam}` |

The `.bed/.bim/.fam` files are symlinks to `francis_2520/Post_Datacleaning/Post_DataCleaning/PLINK_Files/Francis_GDA_TOP_subject_level_long.*` in the same directory. **Build is hg38** (raw delivery from CIDR).


## Raw genotype (hg19)

After liftover to hg19, case extraction and merge with the MDS AML controls.

| Platform | PLINK prefix |
|---|---|
| Illumina GDA TOP (CIDR) | `/francislab/data1/working/20250813-CIDR/20260320d-merge_with_mdsaml/cidr.{bed,bim,fam}` |


## Imputed genotype

UMich 1000 Genomes hg19 panel — IPS was **lifted to hg19 before imputation** so it sits in the same coordinate space as AGS and TCGA imputed outputs:

- `/francislab/data1/working/20250813-CIDR/20260320f-impute_genotypes/imputed-umich-cidr/`

Conventions:
- Chromosome names: `1`, `2`, … (**no** `chr` prefix).
- INFO field is Minimac4 `R2`; use `R2 ≥ 0.8` as the standard well-imputed cutoff.

## Genotype ↔ covariate linker (subject_id ↔ VCF_ID)

- `/francislab/data1/working/20250813-CIDR/20260320f-impute_genotypes/subjectid_VCFid.csv` (columns: `subject_id`, `VCFid`)


## Imputed PGS

| Provider | Path |
|---|---|
| UMich Imputation Server | `/francislab/data1/working/20250813-CIDR/20260320g-impute_pgs/pgs-cidr-hg19/scores.txt.gz` |
| Locally | `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/20260410-compute_PGS_hg19/pgs-calc-scores/cidr/scores.txt.gz` |

join on `VCFid`


## Imputed HLA

| Path |
|---|
| `/francislab/data1/working/20250813-CIDR/20260320h-impute_hla/hla-cidr-hg19/chr6.dose.vcf.gz` |

join on `VCFid`


## Schema notes (canonical covariates)

- **ID key**: `subject_id` (string, e.g. `G317`)
- **`id`**: secondary numeric ID (matches `sampleid` family in the 810-sample clinical)
- **`sex`**: numeric (1/2) — **not** M/F as in AGS
- **`ethnicity`**: numeric coding (verify against DD before use)
- **`idhmut`**: 0=WT, 1=MUT, 9=unknown, blank=missing (same coding as AGS)
- **`pq`**: same coding as AGS
- **`deceased`**: 0/1 (numeric) — **not** A/D as in AGS
- **`survdays`**: days from first diagnosis to death/last contact
- **`who2021`**: WHO 2021 classification (AGS uses `who2016`)
- **`dex`**: dexamethasone at draw — same coding as AGS
- **`surgtype`**: same coding as AGS
- **`mgmt`** + **`mgmtindex`**: IPS carries MGMT methylation calls; AGS canonical does not
- **`timept`**: 1=pre-surgery, 2=post-surgery; longitudinal subjects have multiple rows in the sample-level files

## Cross-cohort harmonization (IPS ↔ AGS)

See the `ags-data` skill for the full mapping table. Recoding to combine cohorts:

- AGS `sex` (M/F) → numeric to match IPS, or vice versa
- AGS `deceased` (A/D) → numeric 0/1 to match IPS
- AGS `who2016` and IPS `who2021` are different classifications — do not merge directly
- AGS `dxgroup_ags` and IPS `dxgroup` use different coding tables

For TCGA, schema is independent — see the `tcga-data` skill.

## AGS ↔ IPS sample overlap

IPS subjects who were previously enrolled in AGS can appear in both genotype cohorts. Round-2 decision (locked 2026-05-26) for declaring duplicates:
- PLINK2 wide net at `--king-cutoff 0.354` for 1st-degree+ candidates
- Strict duplicate call: `IBS0/IBS_total < 0.005` AND `IBD2 > 0.95`

True duplicates are tagged `cross_cohort_replicate=TRUE` with matched AGS_ID. 1st/2nd-degree relatives stay in their cohort and are recorded in a separate kinship file.

## Known gotchas

- The **older** IPS phenotype file `CIDR_IPS_phenotype_2025-08-10_with_IPS_ID.csv` is permission-restricted (mode `400`) and is **superseded** by the 2026-03-03 file — do not use unless you have a specific reason.
- The 810-sample clinical file is **sample-level** (some subjects have multiple rows for multiple timepoints). The 446-subject canonical file is subject-level. Pick the right grain for your analysis.
- `CIDR_case_covariates.csv` (cases-only subset, 33 KB) exists but is not the canonical file.
- IPS sex coding (1/2) differs from AGS (M/F). Always recode before joining.

