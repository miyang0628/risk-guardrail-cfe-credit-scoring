# Datasets

## UCI German Credit (`german.data-numeric`)

Included in this repository. Source: UCI Machine Learning Repository,
"Statlog (German Credit Data)". The `*.data-numeric` file is the
numeric-encoded version (24 features + target). Target is recoded as
`1 = good`, `0 = bad` inside the notebook.

## FICO HELOC (`heloc_dataset_v1.csv`)

Included in this repository for reproducibility. The dataset originates from
the FICO Community "Explainable Machine Learning Challenge" and remains subject
to **FICO's data-usage license**; users must comply with those terms, and
redistribution may require obtaining the dataset directly from FICO. If the
file must be removed for licensing reasons, download it from the official FICO
Community challenge and place it at `data/heloc_dataset_v1.csv`.

The notebook expects the original columns, with the target column
`RiskPerformance` (`Good`/`Bad`). Special sentinel values `-7`, `-8`, `-9`
are mapped to `0` during preprocessing, matching the reported setup.

> The committed result tables in `results/tables/` were produced from this
> dataset, so the figures can be regenerated with `RUN_EXPERIMENT = False`
> even without re-running the full pipeline.
