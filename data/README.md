# Dataset placement

This repository expects the **unbiased Criteo Uplift v2.1** CSV at:

```text
data/criteo-uplift-v2.1.csv
```

The CSV is approximately 3 GiB uncompressed and is intentionally excluded from Git.

## Download

1. Visit the [official Criteo Uplift Prediction Dataset page](https://ailab.criteo.com/criteo-uplift-prediction-dataset/).
2. Download the unbiased v2.1 release (`criteo-research-uplift-v2.1.csv.gz`).
3. Decompress it and rename the CSV to `criteo-uplift-v2.1.csv`.
4. Place the file in this directory.

Expected schema:

```text
f0, f1, ..., f11, treatment, conversion, visit, exposure
```

The release contains 13,979,592 rows. The notebooks also validate the schema, binary fields, missingness, and experiment allocation before modeling.

The dataset has its own terms and citation requirements. It is not distributed under this repository's code terms. If you use the data, cite:

> Diemert, E., Betlei, A., Renaudin, C., & Amini, M.-R. (2018). A Large Scale Benchmark for Uplift Modeling. AdKDD & TargetAd, KDD 2018.
