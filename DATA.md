# MOSAIC Data Access

## Why the data are not included in this repository

The MOSAIC analysis is built on the **Medicare Hospital Cost Report Information System (HCRIS)** panel, covering **83,512 hospital facility-years (FY2011–2024)**. In its processed form this dataset exceeds GitHub's file-size limits and is therefore not included here.

## How to obtain the data

All source data are publicly available at no cost from the **Centers for Medicare & Medicaid Services (CMS)**:

> **CMS Cost Reports**
> [https://www.cms.gov/data-research/statistics-trends-and-reports/cost-reports](https://www.cms.gov/data-research/statistics-trends-and-reports/cost-reports)

Download the **Hospital (Form 2552)** cost report files for fiscal years **2011 through 2024**.

## Preprocessing

Once the raw CMS files are downloaded, the MOSAIC preprocessing pipeline will:

1. Parse and merge the alpha (`HCRIS_ALPHA`) and numeric (`HCRIS_NMRC`) worksheets.
2. Construct the 108 signals across the three MOSAIC channels — **behavioral**, **analytical**, and **model-based**.
3. Assign fiscal-year regime labels: *Transition* (FY2011), *Baseline* (FY2012–2018), *Shock* (FY2019–2021), and *Recovery* (FY2022+).
4. Output the analysis-ready panel in **Parquet** format.

Detailed instructions are provided in [`docs/data_preparation.md`](docs/data_preparation.md) *(forthcoming)*.

## Questions

Please open a [GitHub Issue](../../issues) if you encounter problems accessing or preprocessing the data.
