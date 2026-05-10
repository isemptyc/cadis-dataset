# AR_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ar.admin`
Version: `v1.0.0`
Country: `AR`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ar.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the selected Argentina dataset scope
* Quantifies structural completeness under randomized lookup stress testing
* Documents deterministic policy effects
* Validates boundary isolation against the declared evaluation boundary
* Provides reproducible integrity metrics for release review

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value       |
| ----------------------- | ----------- |
| Dataset ID              | `ar.admin`  |
| Dataset Version         | `v1.0.0`    |
| Country                 | `AR`        |
| Country Name            | `Argentina` |
| Policy Version          | `1.0`       |
| Cadis Minimum Version   | `0.8.20`    |
| Hierarchy Required      | `True`      |
| Repair Required         | `False`     |
| Runtime Policy Detected | `True`      |

## 2.1 Scope

| Field                    | Value |
| ------------------------ | ----- |
| Dataset scope            | `full AR Natural Earth admin-0 boundary` |
| Boundary builder         | `build_ar_boundaries.py` |
| Build boundary JSON      | `ar_country.json` |
| Evaluation boundary JSON | `ar_country.json` |
| Selection rule           | Natural Earth admin-0 country row selected by `ISO2=AR` and country name `Argentina` |
| Scoped boundary bbox     | `[-73.57273962499994, -55.05201588299991, -53.661551879999934, -21.786937763999973]` |

The build and evaluation used the same scoped boundary. No intentional territorial subset is applied for the `v1.0.0` Argentina dataset scope.

---

# 3. Test Methodology

## 3.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside points: `9,000`
* Outside points: `1,000`
* Lookup mode: `runtime`
* Dataset dir: `AR/ar.admin/v1.0.0`

The test intentionally injects outside-boundary samples to validate rejection behavior, offshore classification, and policy-layer containment.

---

# 4. Performance Metrics

| Metric                    | Value           |
| ------------------------- | --------------- |
| Throughput                | `10147.560` QPS |
| Total Runtime             | `0.985 sec`     |
| Overall Pass Rate         | `100.00%`       |
| Inside Coverage Pass Rate | `100.00%`       |
| Policy Pass Rate          | `100.00%`       |

No policy, coverage, or inside-boundary failures were observed.

---

# 5. Scenario Comparison

| Scenario       | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (`ok`/`partial`/`failed`) |
| -------------- | --------- | ---------------- | ------ | ------------- | -------------------------------- |
| `full_policy`  | 100.00%   | 100.00%          | 0      | 0             | 81 / 8,926 / 993                |
| `no_hierarchy` | 100.00%   | 100.00%          | 0      | 0             | 81 / 8,926 / 993                |
| `no_repair`    | 100.00%   | 100.00%          | 0      | 0             | 81 / 8,926 / 993                |
| `no_nearby`    | 99.70%    | 99.67%           | 30     | 30            | 73 / 8,900 / 1,027              |
| `osm_only`     | 99.70%    | 99.67%           | 30     | 30            | 73 / 8,900 / 1,027              |

---

# 6. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 34              |
| Total vs OSM-only | 34              |

## Interpretation

* OSM-only success rate is already high at `99.70%`.
* Hierarchy supplementation was not needed for this sampled run.
* No repair layer is required or activated.
* Nearby fallback resolves a small number of boundary-adjacent inside samples.
* Observed failures without nearby are localized coverage-edge misses, not structural hierarchy defects.

---

# 7. Structural Distribution

## 7.1 Build-Time Administrative Coverage

| Level | Count | Parent Coverage |
| ----- | ----: | --------------- |
| `4`   | 24    | root level      |
| `5`   | 527   | 527 / 527       |
| `8`   | 1,998 | 1,997 / 1,998   |

The engine intentionally exposes OSM admin levels 4, 5, and 8: provinces/autonomous city, departments/partidos/comunas where present, and localities.

## 7.2 Runtime Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[4,5]`   | 8,918 |
| `[]`      | 1,000 |
| `[4,5,8]` | 74    |
| `[4]`     | 8     |

Empty shapes correspond to expected outside-boundary and offshore outcomes:

* `empty_shape`: 993
* `offshore`: 7

## 7.3 Node Source Distribution

| Source          | Count |
| --------------- | ----: |
| `polygon`       | 8,973 |
| `nearby`        | 27    |
| `admin_tree_id` | 5     |

---

# 8. Policy Reason Distribution

| Reason             | Count |
| ------------------ | ----: |
| `shape_status_map` | 9,000 |
| `empty_shape`      | 993   |
| `offshore`         | 7     |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `23`
* Total level-4 hits: `9,000`
* Total level-4 hits inside boundary: `8,996`

Hit distribution reflects uniform land-area sampling.

## 9.1 Top Level-4 Hit Rates

| Level-4 Unit             | Hits | Hit Rate | Inside Hits | Inside Hit Rate |
| ------------------------ | ---: | -------: | ----------: | --------------: |
| `Santa Cruz`             | 959  | 9.59%    | 958         | 10.64%          |
| `Buenos Aires`           | 938  | 9.38%    | 938         | 10.42%          |
| `Chubut`                 | 787  | 7.87%    | 784         | 8.71%           |
| `Río Negro`              | 699  | 6.99%    | 699         | 7.77%           |
| `Córdoba`                | 528  | 5.28%    | 528         | 5.87%           |
| `La Pampa`               | 500  | 5.00%    | 500         | 5.56%           |
| `Mendoza`                | 487  | 4.87%    | 487         | 5.41%           |
| `Santiago del Estero`    | 423  | 4.23%    | 423         | 4.70%           |
| `Santa Fe`               | 414  | 4.14%    | 414         | 4.60%           |
| `Salta`                  | 413  | 4.13%    | 413         | 4.59%           |
| `Catamarca`              | 311  | 3.11%    | 311         | 3.46%           |
| `Neuquén`                | 305  | 3.05%    | 305         | 3.39%           |
| `La Rioja`               | 302  | 3.02%    | 302         | 3.36%           |
| `San Juan`               | 291  | 2.91%    | 291         | 3.23%           |
| `Chaco`                  | 288  | 2.88%    | 288         | 3.20%           |
| `Corrientes`             | 272  | 2.72%    | 272         | 3.02%           |
| `San Luis`               | 255  | 2.55%    | 255         | 2.83%           |
| `Formosa`                | 234  | 2.34%    | 234         | 2.60%           |
| `Entre Ríos`             | 216  | 2.16%    | 216         | 2.40%           |
| `Jujuy`                  | 164  | 1.64%    | 164         | 1.82%           |
| `Provincia de Misiones`  | 82   | 0.82%    | 82          | 0.91%           |
| `Tierra del Fuego`       | 77   | 0.77%    | 77          | 0.86%           |
| `Tucumán`                | 55   | 0.55%    | 55          | 0.61%           |

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed.
* Empty-shape and offshore outcomes matched expected outside-boundary behavior.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.

This confirms strict boundary containment within the AR dataset for this sampled run.

---

# 11. Structural Observations

1. Geometry integrity is high; no repair layer activation.
2. Level-5 parent coverage is complete in the build artifact.
3. Level-8 parent coverage is nearly complete, with one unparented level-8 feature observed at build time.
4. Most runtime samples resolve to `[4,5]`; `[4,5,8]` locality hits are sparse under uniform land-area sampling.
5. Nearby fallback is small and bounded.
6. Dataset achieves full inside-boundary coverage under policy mode.
7. Boundary rejection behavior is strict and leak-free in the sampled run.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit: `b4272517e6e1f0ebdd2e74a786262a406ea026b6`
* Cadis version: `0.8.20`
* OSM source: `geofabrik:south-america/argentina`
* OSM replication timestamp: `2026-05-09T20:21:02Z`
* Boundary builder: `build_ar_boundaries.py`
* Build boundary: `ar_country.json`
* Evaluation boundary: `ar_country.json`
* Dataset package: `AR/ar.admin/v1.0.0`

The staged dataset was generated from a clean `cadis_dataset_engine` working tree.

---

# 13. Conclusion

The `ar.admin v1.0.0` dataset demonstrates:

* High geometric integrity
* Complete policy-mode inside-boundary coverage in the sampled run
* No required repair layer
* Minimal nearby fallback use
* Strict cross-border isolation
* A conservative, stable runtime model based on levels 4, 5, and 8

