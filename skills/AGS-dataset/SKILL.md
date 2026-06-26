---
name: AGS-dataset
description: The UCSF Adult Glioma Study is a landmark, case-control research initiative aimed at identifying the genetic, developmental, environmental, and immunologic risk factors for gliomas. The study also maps molecular-level tumor markers to help classify these brain cancers into more homogeneous, targeted treatment groups. Reach for this skill whenever a task touches the AGS (Adult Glioma Study) cohort — covariates, IDH/1p19q/survival outcomes, raw or imputed genotype, PGS scores, HLA calls, ancestry PCs, or AGS-IPS-TCGA harmonized analyses. Use when the user says "AGS", "Adult Glioma Study", "AGS-Mayo Oncoarray", "AGS Illumina", "AGS I370", "the glioma cases", or refers to AGSIDs (e.g. AGS40056). Provides canonical file paths on the francislab cluster and schema notes so analyses don't drift across sessions.
---

# AGS — Adult Glioma Study (UCSF)

UCSF case-control glioma study. Adult glioma patients enrolled at UCSF plus controls. Covariates: demographics, tumor molecular markers (IDH, 1p/19q, TERT, WHO 2016), treatment exposure (dexamethasone, chemo, radiation, surgery), and overall survival.

All paths below are on the C4 cluster (`ssh:c4-log1-dev4`). Files under `/francislab/data1/refs/AGS/` are read-only — do downstream work in your own working directory.

## Canonical files

| Purpose | Path |
|---|---|
| **Covariates (canonical)** | `/francislab/data1/refs/AGS/Subject_Phenotype_DS_only_AGSID.csv` |
| **Data dictionary** | `/francislab/data1/refs/AGS/Subject_Phenotype_Meth_Batch1_DD.csv` |
| **Extended covariates + HLA** | `/francislab/data1/refs/AGS/Cleaned_Covariates_with_HLA.tsv` |
| **PGS + ancestry PCs** | `/francislab/data1/refs/AGS/merged_pgs_ancestry_PCs.tsv` |
| **All AGS IDs** | `/francislab/data1/refs/AGS/All_AGS_ids.txt` |

The canonical covariate file is 473 subjects (`AGSID` key), curated and **schema-harmonized with IPS** (the DD covers both: `study_source` 1=AGS, 2=IPS). The extended HLA file is 3508 subjects — the full covariate dump including non-DS subjects — use it when you need HLA calls, serology, or the larger sample set.

**Legacy / superseded** (do not use as default):
- `AGS_Covariates_Updated_2022-11-03.csv` — older snapshot, no HLA.
- `20.Feb.2026_…AGScases…` + `…AGScontrols…` — Feb-2026 split-file pair.

## Raw genotype (hg19)

Two array platforms; subject sets are **disjoint**.

| Platform | PLINK prefix |
|---|---|
| Mayo OncoArray | `/francislab/data1/raw/20210226-AGS-Mayo-Oncoarray/AGS_Mayo_Oncoarray_for_QC.{bed,bim,fam}` |
| Illumina I370 | `/francislab/data1/raw/20210302-AGS-illumina/AGS_illumina_for_QC.{bed,bim,fam}` |

Both are hg19. The Illumina `.bim.original` preserves pre-position-update calls.

## Imputed genotype

UMich 1000 Genomes hg19 panel (**NOT TopMed** — verified 2026-05-26):
- ONCO: `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/20251218-survival_gwas/imputed-umich-onco/`
- I370: `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/20251218-survival_gwas/imputed-umich-i370/`

Conventions:
- Chromosome names: `1`, `2`, … (**no** `chr` prefix).
- INFO field is Minimac4 `R2`; use `R2 ≥ 0.8` as the standard well-imputed cutoff.
- VCF headers do not carry build / contig lengths — build was confirmed by spot-checking known SNP positions.

## Genotype ↔ covariate linkers (AGSID ↔ VCF_ID)

- ONCO: `/francislab/data1/raw/20260514-CIDR-AGS-Methylation/AGS_ONCO_Genotyping_Subjects_VCF_ID.csv` (1339 AGS)
- I370: `/francislab/data1/raw/20260514-CIDR-AGS-Methylation/AGS_I370_Genotyping_Subjects_VCF_ID.csv` (1268 AGS)

Sets are non-overlapping (0 in both). Always check which platform an AGSID is on before genotype analyses.

##	Imputed PGS

| Platform | Provider | Path |
|---|---|
| Illumina I370 | UMich Imputation Server | `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/20250724-pgs/pgs-i370-hg19/scores.txt.gz` |
| Mayo OncoArray | UMich Imputation Server | `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/20250724-pgs/pgs-onco-hg19/scores.txt.gz` |
| Illumina I370 | Locally | `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/20260410-compute_PGS_hg19/pgs-calc-scores/i370/scores.txt.gz` |
| Mayo OncoArray | Locally | `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/20260410-compute_PGS_hg19/pgs-calc-scores/onco/scores.txt.gz` |

##	Imputed HLA

| Platform | Path |
|---|---|
| Illumina I370 | `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/20250725-hla/hla-i370-hg19/chr6.dose.vcf.gz` |
| Mayo OncoArray | `/francislab/data1/working/20250800-AGS-CIDR-ONCO-I370-TCGA/20250725-hla/hla-onco-hg19/chr6.dose.vcf.gz` |

## Schema notes (canonical covariates)

- **ID key**: `AGSID` (string, e.g. `AGS40056`)
- **`sex`**: `M` / `F`
- **`idhmut`**: 0=WT, 1=MUT, 9=unknown, blank=missing
- **`pq`**: 0=intact, 1=codeleted, 8=assay failed, 9=not assayed, blank=missing
- **`pqimpute`**: AGS-only imputed 1p/19q (0=intact, 1=codel, blank=missing) — see DD for inference rules
- **`deceased`**: `A`=alive, `D`=deceased, blank=missing/control
- **`survdays`**: days from first diagnosis to death/last contact; blank for controls
- **`who2016`**: 1=IDH+1p19q codel oligo, 2=IDH-mut astro, 3=IDH-mut GBM, 4=IDH-WT GBM, 5=IDH-WT diffuse astro, 9=unknown
- **`ethnicity`**: 1=Hispanic, 2=Non-Hispanic, 9=unknown
- **`race`**: 1=White, 2=Black/AfAm, 3=Asian/PI, 4=Other/mixed, 5=Native American, 9=unknown
- **`dex`**: dexamethasone at blood draw — 1=yes, 0=no, 9=unknown
- **`surgtype`**: 0=biopsy, 1=resection
- **`timept`**: 1=pre-surgery, 2=post-surgery (controls: at interview)

## Cross-cohort harmonization (AGS ↔ IPS)

The canonical AGS file shares schema with the canonical IPS file (`CIDR_IPS_phenotype_2026-03-03_updated_vstatus.csv`). Variable meanings match for `idhmut`, `pq`, `survdays`, `deceased`, `dex`, `surgtype`, `bmi`, `bldyear`, `timept`, `first_surg_year`. Differences to handle when joining:

| Concept | AGS column | IPS column | Note |
|---|---|---|---|
| ID | `AGSID` | `subject_id` | Different string formats; no overlap by construction |
| Sex | `sex` (M/F) | `sex` (numeric 1/2) | Recode before merging |
| WHO grade | `who2016` | `who2021` | Different classifications |
| Diagnosis group | `dxgroup_ags` | `dxgroup` | Different coding tables |
| MGMT | — | `mgmt`, `mgmtindex` | AGS canonical does not carry MGMT |

For TCGA, schema is independent — see the `tcga-data` skill.

## Known gotchas

- `Cleaned_Covariates_with_HLA.tsv` (3508 rows) and `Subject_Phenotype_DS_only_AGSID.csv` (473 rows) are NOT the same cohort. The 473-subject file is the curated downsampled set used in harmonized analyses; the 3508-subject file is the full dump.
- AGS genotype is split across two disjoint array platforms. PGS scores, imputation, and any genotype analysis must be aware of which platform each AGSID is on (use the linker files above).
- The Feb-2026 split-file pair (`…AGScases…csv` + `…AGScontrols…csv`) is a recent processed-clinical snapshot — use only if you explicitly need it; otherwise prefer the canonical DS file.


