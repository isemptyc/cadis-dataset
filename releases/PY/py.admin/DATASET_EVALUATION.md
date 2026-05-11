# PY_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `py.admin`
Version: `v1.0.0`
Country: `PY`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `py.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Paraguay dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `py.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `PY`              |
| Country Name            | `Paraguay`        |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.47`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_py_boundaries.py`       |
| Generated Boundary       | `tmp/py_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth PY admin-0 country boundary.` |
| Boundary BBox            | `[-62.65035721899994, -27.58684214299994, -54.24528885899994, -19.28672861699995]` |
| OSM Source               | `geofabrik:south-america/paraguay`     |
| OSM Snapshot Timestamp   | `2026-05-09T20:21:02Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Paraguay engine exposes these OSM administrative levels:

| Level | Runtime Label         | Dataset Count | Notes |
| ----: | --------------------- | ------------: | ----- |
| 4     | `admin_department`    | 18            | Departments and capital-level unit |
| 6     | `admin_district`      | 10            | Sparse district coverage |
| 7     | `admin_municipality`  | 12            | Sparse municipality coverage |
| 8     | `admin_locality`      | 116           | Locality coverage |
| 9     | `admin_neighborhood`  | 213           | Neighborhood/detail coverage |
| 10    | `admin_detail`        | 333           | Additional detail coverage |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 3     | 2           | Excluded as broad regional overlays |
| 5     | 0           | Excluded because no scoped features were present |

The engine uses canonical names from `name:es`, `name`, `official_name`, and `name:en`, with bounded multilingual aliases for `es` and `en`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `PY/py.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over land area, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value           |
| ------------------------- | --------------- |
| Overall Pass Rate         | `100.00%`       |
| Inside Coverage Pass Rate | `100.00%`       |
| Policy Pass Rate          | `100.00%`       |
| Failed Samples            | `0`             |
| Throughput                | `15280.170` QPS |
| Total Runtime             | `0.654 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,007 / 1 / 992 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,007 / 1 / 992 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,007 / 1 / 992 / 0 |
| no_nearby    | 99.66%    | 99.62%           | 34     | 34            | 8,966 / 1 / 1,033 / 0 |
| osm_only     | 99.66%    | 99.62%           | 34     | 34            | 8,966 / 1 / 1,033 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 41              |
| Total vs OSM-only | 41              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `py.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape       | Count |
| ----------- | ----: |
| `[4,8,10]` | 3,342 |
| `[4,8]`    | 3,153 |
| `[4]`      | 2,078 |
| `[]`       | 1,004 |
| `[4,7]`    | 279   |
| `[4,6]`    | 73    |
| `[4,10]`   | 49    |
| `[4,8,9]` | 10    |

Empty shapes correspond to:

* `empty_shape`: 992
* `offshore`: 12

## 9.2 Node Source Distribution

| Source  | Count |
| ------- | ----: |
| polygon | 8,967 |
| nearby  | 29    |

## 9.3 Source Mix Distribution

| Mix     | Count |
| ------- | ----: |
| polygon | 8,967 |
| none    | 1,004 |
| nearby  | 29    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 8,996 |
| empty_shape      | 992   |
| offshore         | 12    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `17`
* Total level-4 hits across all samples: `8,995`
* Total level-4 hits across inside samples: `8,994`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Boquerón | 2,015 | 20.15% | 2,015 | 22.39% |
| Alto Paraguay | 1,718 | 17.18% | 1,718 | 19.09% |
| Presidente Hayes | 1,629 | 16.29% | 1,628 | 18.09% |
| San Pedro | 476 | 4.76% | 476 | 5.29% |
| Concepción | 373 | 3.73% | 373 | 4.14% |
| Canindeyú | 372 | 3.72% | 372 | 4.13% |
| Itapúa | 341 | 3.41% | 341 | 3.79% |
| Alto Paraná | 317 | 3.17% | 317 | 3.52% |
| Ñeembucú | 284 | 2.84% | 284 | 3.16% |
| Caaguazú | 268 | 2.68% | 268 | 2.98% |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/py_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Paraguay dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shapes are `[4,8,10]`, `[4,8]`, and `[4]`, reflecting broad department coverage with uneven lower-level detail.
3. Levels 6 and 7 are intentionally included because they provide valid but sparse intermediate administrative coverage.
4. Level 3 is intentionally excluded from the runtime contract because it represents two broad regional overlays rather than the primary operational hierarchy.
5. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `b308f8cc3c0bf8f1f35de3e33a6e5aab9284efcc`
- Cadis version:
  `0.8.47`
- Boundary builder: `scripts/build_py_boundaries.py`
- Build boundary: `tmp/py_country.json`
- Evaluation boundary: `tmp/py_country.json`
- Staged dataset: `PY/py.admin/v1.0.0`
- Source OSM SHA256:
  `e27336c619efd6b5850eeafc68373c75e207ab438c93efa203db9e6f7a6581f7`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `py.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Paraguay levels 4, 6, 7, 8, 9, and 10

The dataset is suitable for human quality review.
