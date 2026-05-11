# NI_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ni.admin`
Version: `v1.0.0`
Country: `NI`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ni.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Nicaragua dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `ni.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `NI`              |
| Country Name            | `Nicaragua`       |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.61`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_ni_boundaries.py`       |
| Generated Boundary       | `tmp/ni_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4 and 6 whose representative point is covered by the Natural Earth NI boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox            | `[-87.89381697684628, 10.70966411069074, -82.47872485780692, 15.087456173777174]` |
| OSM Source               | `geofabrik:central-america/nicaragua`  |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Nicaragua administrative coverage represented by materialized OSM department and municipality relations. The country envelope is excluded from the initial runtime contract.

---

# 4. Administrative Model

The initial Nicaragua engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_department`   | 17            | Department/region-level coverage |
| 6     | `admin_municipality` | 153           | Municipality-level coverage |
| 8     | `admin_district`     | 6             | District-level coverage where available |
| 9     | `admin_neighborhood` | 4             | Neighborhood coverage where available |
| 10    | `admin_detail`       | 5,493         | Detail/local coverage where available |
| 11    | `admin_micro_detail` | 8             | Micro-detail coverage where available |
| 12    | `admin_sub_detail`   | 269           | Sub-detail coverage where available |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |

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
* Dataset path: `NI/ni.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over scoped administrative coverage, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value          |
| ------------------------- | -------------- |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |
| Failed Samples            | `0`            |
| Throughput                | `7194.130` QPS |
| Total Runtime             | `1.390 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,021 / 0 / 979 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,021 / 0 / 979 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,021 / 0 / 979 / 0 |
| no_nearby    | 100.00%   | 100.00%          | 0      | 0             | 9,000 / 0 / 1,000 / 0 |
| osm_only     | 100.00%   | 100.00%          | 0      | 0             | 9,000 / 0 / 1,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 21              |
| Total vs OSM-only | 21              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `ni.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape        | Count |
| ------------ | ----: |
| `[4,6]`      | 8,932 |
| `[]`         | 997   |
| `[4,6,10]`  | 49    |
| `[4,6,8]`   | 8     |
| `[4,6,9]`   | 6     |
| `[4,6,8,10]` | 5    |
| `[4]`        | 3     |

Empty shapes correspond to:

* `empty_shape`: 979
* `offshore`: 18

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 9,000 |
| admin_tree_id | 22    |
| nearby        | 3     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,978 |
| none                   | 997   |
| admin_tree_id\|polygon | 22    |
| nearby                 | 3     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,003 |
| empty_shape      | 979   |
| offshore         | 18    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `17`
* Total level-4 hits across all samples: `9,003`
* Total level-4 hits across inside samples: `9,000`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Costa Caribe Norte | 2,604 | 26.04% | 2,602 | 28.91% |
| Costa Caribe Sur | 2,281 | 22.81% | 2,281 | 25.34% |
| Jinotega | 527 | 5.27% | 527 | 5.86% |
| Río San Juan | 489 | 4.89% | 488 | 5.42% |
| Chontales | 476 | 4.76% | 476 | 5.29% |
| Chinandega | 426 | 4.26% | 426 | 4.73% |
| Matagalpa | 372 | 3.72% | 372 | 4.13% |
| León | 362 | 3.62% | 362 | 4.02% |
| Rivas | 334 | 3.34% | 334 | 3.71% |
| Managua | 260 | 2.60% | 260 | 2.89% |
| Nueva Segovia | 222 | 2.22% | 222 | 2.47% |
| Boaco | 207 | 2.07% | 207 | 2.30% |
| Estelí | 119 | 1.19% | 119 | 1.32% |
| Granada | 110 | 1.10% | 110 | 1.22% |
| Carazo | 93 | 0.93% | 93 | 1.03% |
| Madriz | 91 | 0.91% | 91 | 1.01% |
| Masaya | 30 | 0.30% | 30 | 0.33% |

Hit distribution reflects uniform sampling over the scoped administrative coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ni_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Nicaragua dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,6]`, reflecting strong department/region and municipality coverage.
3. Levels 8, 9, 10, 11, and 12 provide valid lower-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `903e091e9584f04972291600bb7a9bda66fd8391`
- Cadis version:
  `0.8.61`
- Boundary builder: `scripts/build_ni_boundaries.py`
- Build boundary: `tmp/ni_country.json`
- Evaluation boundary: `tmp/ni_country.json`
- Staged dataset: `NI/ni.admin/v1.0.0`
- Source OSM SHA256:
  `25085ae9ebd58961e7803db7047b006adc34282b9250c4a92a71cc9abf4a3d9b`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ni.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Nicaragua levels 4, 6, 8, 9, 10, 11, and 12

The dataset is suitable for human quality review.
