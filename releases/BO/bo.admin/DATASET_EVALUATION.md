# BO_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `bo.admin`
Version: `v1.0.0`
Country: `BO`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `bo.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Bolivia dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `bo.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `BO`              |
| Country Name            | `Bolivia`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.44`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_bo_boundaries.py`       |
| Generated Boundary       | `tmp/bo_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth BO admin-0 country boundary.` |
| Boundary BBox            | `[-69.66649226999988, -22.89725758799996, -57.465660766999946, -9.679821471999986]` |
| OSM Source               | `geofabrik:south-america/bolivia`      |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Bolivia engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_department`   | 9             | Departments |
| 6     | `admin_province`     | 112           | Provinces |
| 8     | `admin_municipality` | 335           | Municipal/local administrative units |
| 9     | `admin_canton`       | 207           | Canton/detail coverage |
| 10    | `admin_neighborhood` | 21            | Neighborhood/detail coverage |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 3     | 0           | Excluded because no scoped features were present |
| 5     | 0           | Excluded because no scoped features were present |
| 7     | 1           | Excluded as a single sparse overlay |

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
* Dataset path: `BO/bo.admin/v1.0.0`

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
| Throughput                | `11344.440` QPS |
| Total Runtime             | `0.881 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 8,933 / 78 / 989 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 8,933 / 78 / 989 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 8,933 / 78 / 989 / 0 |
| no_nearby    | 99.36%    | 99.29%           | 64     | 64            | 8,864 / 73 / 1,063 / 0 |
| osm_only     | 99.36%    | 99.29%           | 64     | 64            | 8,864 / 73 / 1,063 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 74              |
| Total vs OSM-only | 74              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `bo.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape        | Count |
| ------------ | ----: |
| `[4,6,8]`    | 6,546 |
| `[4,6,8,9]` | 2,351 |
| `[]`         | 1,020 |
| `[4]`        | 71    |
| `[4,6,8,10]` | 5    |
| `[4,8]`      | 4     |
| `[4,6]`      | 3     |

Empty shapes correspond to:

* `empty_shape`: 989
* `offshore`: 31

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,937 |
| nearby        | 43    |
| admin_tree_id | 12    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,925 |
| none                   | 1,020 |
| nearby                 | 43    |
| admin_tree_id\|polygon | 12    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 8,980 |
| empty_shape      | 989   |
| offshore         | 31    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `9`
* Total level-4 hits across all samples: `8,980`
* Total level-4 hits across inside samples: `8,979`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Santa Cruz | 2,973 | 29.73% | 2,973 | 33.03% |
| Beni | 1,742 | 17.42% | 1,742 | 19.36% |
| La Paz | 1,085 | 10.85% | 1,085 | 12.06% |
| Potosí | 955 | 9.55% | 955 | 10.61% |
| Pando | 543 | 5.43% | 543 | 6.03% |
| Cochabamba | 505 | 5.05% | 505 | 5.61% |
| Chuquisaca | 452 | 4.52% | 452 | 5.02% |
| Oruro | 403 | 4.03% | 403 | 4.48% |
| Tarija | 322 | 3.22% | 321 | 3.57% |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/bo_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Bolivia dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,6,8]`, reflecting department, province, and municipality coverage.
3. Level 9 adds canton/detail coverage where available; level 10 is sparse but valid neighborhood/detail coverage.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Level 7 is intentionally excluded from the initial runtime contract because it appears as a single sparse overlay.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `cdb292d832b3c826ef04f7a033e45990cdb2b322`
- Cadis version:
  `0.8.44`
- Boundary builder: `scripts/build_bo_boundaries.py`
- Build boundary: `tmp/bo_country.json`
- Evaluation boundary: `tmp/bo_country.json`
- Staged dataset: `BO/bo.admin/v1.0.0`
- Source OSM SHA256:
  `82c3a440cc05405c8a4d845ba8825d5cd9a1fc760f05b7a11adcbc604e496ac0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `bo.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Bolivia levels 4, 6, 8, 9, and 10

The dataset is suitable for human quality review.
