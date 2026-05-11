# MK_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `mk.admin`
Version: `v1.0.0`
Country: `MK`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `mk.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Macedonia dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `mk.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `MK`              |
| Country Name            | `Macedonia`       |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.41`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_mk_boundaries.py`       |
| Generated Boundary       | `tmp/mk_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth MK admin-0 country boundary.` |
| Boundary BBox            | `[20.44415734900008, 40.849394023000045, 23.00958215300011, 42.37033477900009]` |
| OSM Source               | `geofabrik:europe/macedonia`           |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Macedonia engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 7     | `admin_municipality` | 80            | Municipalities |
| 8     | `admin_settlement`   | 1,739         | Settlements |
| 9     | `admin_neighborhood` | 17            | Sparse Skopje/local neighborhood detail |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 6     | 1           | Excluded as the single City of Skopje envelope |
| 10    | 0           | Excluded because no scoped features were found |

The engine uses canonical names from `name:mk`, `name:sq`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `en`, `mk`, `sq`, `sr`, and `tr`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `MK/mk.admin/v1.0.0`

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
| Throughput                | `13071.880` QPS |
| Total Runtime             | `0.765 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,052 / 0 / 948 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,052 / 0 / 948 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,052 / 0 / 948 / 0 |
| no_nearby    | 98.79%    | 98.66%           | 121    | 121           | 8,880 / 0 / 1,120 / 0 |
| osm_only     | 98.79%    | 98.66%           | 121    | 121           | 8,880 / 0 / 1,120 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 172             |
| Total vs OSM-only | 172             |

Nearby fallback resolves boundary-adjacent and municipal-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `mk.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[7,8]`   | 8,883 |
| `[]`      | 980   |
| `[7]`     | 96    |
| `[7,8,9]` | 41    |

Empty shapes correspond to:

* `empty_shape`: 948
* `offshore`: 32

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,880 |
| nearby        | 140   |
| admin_tree_id | 13    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,867 |
| none                   | 980   |
| nearby                 | 140   |
| admin_tree_id\|polygon | 13    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,020 |
| empty_shape      | 948   |
| offshore         | 32    |

---

# 10. Level-4 Coverage

No level-4 city hit rates are reported for this dataset because the Macedonia runtime contract does not expose level 4.

The primary runtime parent layer is level 7 municipality coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/mk_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Macedonia dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[7,8]`, reflecting municipality and settlement coverage.
3. Level 9 adds sparse neighborhood/local detail where available.
4. Nearby fallback is bounded at 5 km and accounts for a small rescue effect around administrative edges.
5. The single level-6 City of Skopje envelope is intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `9eecb2ad28d6d4d24e0ad0f24e692cfd397e7a22`
- Cadis version:
  `0.8.41`
- Boundary builder: `scripts/build_mk_boundaries.py`
- Build boundary: `tmp/mk_country.json`
- Evaluation boundary: `tmp/mk_country.json`
- Staged dataset: `MK/mk.admin/v1.0.0`
- Source OSM SHA256:
  `2a8338dd64f4b5102c7cd1c69d411d273edefd12b4726a41276ac8342b1cce72`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `mk.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Macedonia levels 7, 8, and 9

The dataset is suitable for human quality review.
