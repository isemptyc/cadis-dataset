# JM_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `jm.admin`
Version: `v1.0.0`
Country: `JM`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `jm.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Jamaica dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `jm.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `JM`              |
| Country Name            | `Jamaica`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.60`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_jm_boundaries.py`       |
| Generated Boundary       | `tmp/jm_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 5 and 6 whose representative point is covered by the Natural Earth JM boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox            | `[-78.36630957433923, 17.707605720477666, -76.1849980927078, 18.523009379111777]` |
| OSM Source               | `geofabrik:central-america/jamaica`    |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Jamaica administrative coverage represented by materialized OSM county and parish relations. The country envelope is excluded from the initial runtime contract.

---

# 4. Administrative Model

The initial Jamaica engine exposes these OSM administrative levels:

| Level | Runtime Label    | Dataset Count | Notes |
| ----: | ---------------- | ------------: | ----- |
| 5     | `admin_county`   | 3             | County-level coverage |
| 6     | `admin_parish`   | 14            | Parish-level coverage |
| 8     | `admin_locality` | 4             | Locality coverage where available |
| 9     | `admin_detail`   | 9             | Detail coverage where available |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |

The engine uses canonical names from `name`, `official_name`, and `name:en`, with bounded multilingual aliases for `en`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `JM/jm.admin/v1.0.0`

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
| Throughput                | `6767.440` QPS |
| Total Runtime             | `1.478 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,064 / 0 / 936 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,064 / 0 / 936 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,064 / 0 / 936 / 0 |
| no_nearby    | 100.00%   | 100.00%          | 0      | 0             | 9,001 / 0 / 999 / 0 |
| osm_only     | 100.00%   | 100.00%          | 0      | 0             | 9,001 / 0 / 999 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 63              |
| Total vs OSM-only | 63              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `jm.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[5,6]`   | 8,765 |
| `[]`      | 993   |
| `[5,6,9]` | 209   |
| `[5,6,8]` | 28    |
| `[5]`     | 5     |

Empty shapes correspond to:

* `empty_shape`: 936
* `offshore`: 57

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 9,001 |
| nearby        | 6     |
| admin_tree_id | 3     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,998 |
| none                   | 993   |
| nearby                 | 6     |
| admin_tree_id\|polygon | 3     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,007 |
| empty_shape      | 936   |
| offshore         | 57    |

---

# 10. Level-5 Coverage

* Unique level-5 units hit: `3`
* Dominant runtime shape: `[5,6]`
* Total polygon-sourced samples: `9,001`

Jamaica does not expose level 4 in this dataset. County and parish coverage is therefore summarized through shape and source distributions rather than level-4 hit-rate tables.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/jm_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Jamaica dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[5,6]`, reflecting strong county and parish coverage.
3. Levels 8 and 9 provide valid lower-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `605fce49ad2cf89e71fc01a9a9b775ab9ae7a311`
- Cadis version:
  `0.8.60`
- Boundary builder: `scripts/build_jm_boundaries.py`
- Build boundary: `tmp/jm_country.json`
- Evaluation boundary: `tmp/jm_country.json`
- Staged dataset: `JM/jm.admin/v1.0.0`
- Source OSM SHA256:
  `365c6649b02f1d16ab98c6571a856b80b3d59ca3e054a4de9947cb83d4dfafed`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `jm.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Jamaica levels 5, 6, 8, and 9

The dataset is suitable for human quality review.
