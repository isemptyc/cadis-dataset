# UA_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ua.admin`
Version: `v1.0.0`
Country: `UA`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ua.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Ukraine dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `ua.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `UA`              |
| Country Name            | `Ukraine`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.43`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_ua_boundaries.py`       |
| Generated Boundary       | `tmp/ua_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth UA admin-0 country boundary.` |
| Boundary BBox            | `[22.13283980300011, 45.21356842700004, 40.1595430910001, 52.36894928000008]` |
| OSM Source               | `geofabrik:europe/ukraine`             |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Ukraine engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_oblast`       | 25            | Oblasts and Kyiv |
| 6     | `admin_raion`        | 126           | Raions |
| 7     | `admin_hromada`      | 1,466         | Hromadas |
| 8     | `admin_locality`     | 171           | Locality coverage where modeled |
| 9     | `admin_settlement`   | 5,247         | Settlement/detail coverage |
| 10    | `admin_neighborhood` | 889           | Neighborhood/detail coverage |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 3     | 0           | Excluded because no scoped features were present |
| 5     | 0           | Excluded because no scoped features were present |

The engine uses canonical names from `name:uk`, `name:ru`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `en`, `pl`, `ro`, `ru`, and `uk`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `UA/ua.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over land area, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value          |
| ------------------------- | -------------- |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |
| Failed Samples            | `0`            |
| Throughput                | `6152.040` QPS |
| Total Runtime             | `1.625 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,012 / 2 / 986 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,012 / 2 / 986 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,012 / 2 / 986 / 0 |
| no_nearby    | 99.68%    | 99.64%           | 32     | 32            | 8,970 / 2 / 1,028 / 0 |
| osm_only     | 99.68%    | 99.64%           | 32     | 32            | 8,970 / 2 / 1,028 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 42              |
| Total vs OSM-only | 42              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `ua.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape          | Count |
| -------------- | ----: |
| `[4,6,7]`      | 8,453 |
| `[]`           | 992   |
| `[4,6,7,9]`   | 267   |
| `[4,6,7,10]`  | 114   |
| `[4,6]`        | 80    |
| `[4,6,7,8,10]` | 28   |
| `[4,6,7,8]`   | 27    |
| `[4]`          | 14    |

Empty shapes correspond to:

* `empty_shape`: 986
* `offshore`: 6

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,972 |
| nearby        | 36    |
| admin_tree_id | 6     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,966 |
| none                   | 992   |
| nearby                 | 36    |
| admin_tree_id\|polygon | 6     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,008 |
| empty_shape      | 986   |
| offshore         | 6     |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `25`
* Total level-4 hits across all samples: `9,006`
* Total level-4 hits across inside samples: `8,998`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Одеська область | 535 | 5.35% | 531 | 5.90% |
| Чернігівська область | 526 | 5.26% | 526 | 5.84% |
| Дніпропетровська область | 508 | 5.08% | 508 | 5.64% |
| Житомирська область | 482 | 4.82% | 482 | 5.36% |
| Харківська область | 478 | 4.78% | 478 | 5.31% |
| Вінницька область | 451 | 4.51% | 451 | 5.01% |
| Київська область | 450 | 4.50% | 450 | 5.00% |
| Полтавська область | 429 | 4.29% | 429 | 4.77% |
| Херсонська область | 410 | 4.10% | 409 | 4.54% |
| Запорізька область | 409 | 4.09% | 409 | 4.54% |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ua_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Ukraine dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,6,7]`, reflecting oblast, raion, and hromada coverage.
3. Level 9 and level 10 add dense settlement and neighborhood detail where available.
4. Nearby fallback is bounded at 5 km and accounts for a small rescue effect around administrative edges.
5. Levels 3 and 5 are intentionally excluded from the initial runtime contract because no scoped features were present.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `27b8dddcf69a35d0f04f4c2de76bb833f60b2c1a`
- Cadis version:
  `0.8.43`
- Boundary builder: `scripts/build_ua_boundaries.py`
- Build boundary: `tmp/ua_country.json`
- Evaluation boundary: `tmp/ua_country.json`
- Staged dataset: `UA/ua.admin/v1.0.0`
- Source OSM SHA256:
  `44d1a7db0b30d4ffff2b8ab4d85869d4d9c7c23c656862dfe281881cab20889a`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ua.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Ukraine levels 4, 6, 7, 8, 9, and 10

The dataset is suitable for human quality review.
