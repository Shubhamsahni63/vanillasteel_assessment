#  Scenario A - Supplier Data Cleaning

## Content(s)
- Task A.1: Clean & Join

## Output(s)
- `inventory_dataset.csv`

## Summary
- Key Assumptions Made
  - Data Quality Assumptions:
    - Zero values in mechanical properties (rp02, rm, ag, ai) represent missing data and is replaced with NA
    - All text fields is stripped of extra whitespace and has been standardized
    - German terms in supplier_data1 is translated to English for consistency
  - Schema Integration Assumptions:
    - The gross_weight_kg column in df1 is equivalent to weight_kg in df2
    - The datasets is concatenated (not joined) since they represent different inventory items

- Observations:
  - Quantity values should be treated as numeric instead of being decimal or the column Quantity itself is incorrect column name.

- Concatenation vs. Join Rationale
The output file is to concatenate the datasets rather than join them because:
  - The two datasets have different schemas and represent different types of inventory items
  - There is no common unique identifier (key) between the datasets that would allow for a meaningful join
  - The datasets come from different sources with different attribute sets
  - A concatenation approach preserves all records from both sources while maintaining data integrity

--------------------------------

#  Scenario B - RFQ Similarity Pipeline

This repository contains a reproducible pipeline for Scenario B — RFQ Similarity.

## Content(s)
- Task B.1: Reference join & missing values
- Task B.2: Feature engineering (interval IoU, categoricals, mechanical similarity)
- Task B.3: Aggregate similarity and Top-3 retrieval
- Task B.4: Pipeline & documentation

## Inputs
- `rfq.csv`: RFQs with grades, dimensions, and attributes
- `reference_properties.tsv`: Grade reference (mechanical ranges, metadata)

## Outputs
- `rfq_enriched.csv`: RFQs enriched with normalized grades, parsed ranges, reference joins, imputed mechanicals, and flags
- `task_b1_join_report.json`: Join/imputation diagnostics
- `top3.csv`: Top-3 most similar RFQs per line (excluding self and exact duplicates), with per-feature contributions

## How to run

Option B — Notebook:
- Open `Tasks.ipynb`
- Run all cells to produce the same outputs

## Method summary
- Grade normalization aligns RFQ grades with reference; backoff to base-grade families when needed
- Numeric ranges parsed into min/max/mid; unit consistency assumed (mm, MPa)
- Reference join (exact → family backoff) and imputation of missing mechanicals where safe
- Features:
  - Dimensions: 1D Interval IoU per relevant dimension (form-aware)
  - Categoricals: exact match (1/0) for `form`, `coating`, `finish`, `surface_type`
  - Mechanicals: exponential similarity on `yield_strength_mid`, `tensile_strength_mid` (scales 50/75 MPa)
- Similarity score: weighted average of dimensions (0.5), categoricals (0.2), mechanicals (0.3); weights renormalized over present parts, with hard filter on `form`

## Notes
- NaNs are preserved where data is missing; flags included for downstream filtering
- Exact duplicates (same signature across selected fields) are excluded in Top-3
- Weights/scales are easily adjustable in `run.py`
