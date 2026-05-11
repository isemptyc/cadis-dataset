# HT_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ht.admin`
Version: `v1.0.0`
Country: `HT`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ht.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Haiti dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `ht.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `HT`              |
| Country Name            | `Haiti`           |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.57`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_ht_boundaries.py`       |
| Generated Boundary       | `tmp/ht_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4 and 5 whose representative point is covered by the Natural Earth HT boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox            | `[-74.47894910925477, 18.02263251395402, -71.63092577259803, 20.087583254759057]` |
| OSM Source               | `geofabrik:central-america/haiti-and-domrep` |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Haiti administrative coverage represented by materialized OSM department and arrondissement relations.

---

# 4. Administrative Model

The initial Haiti engine exposes these OSM administrative levels:

| Level | Runtime Label          | Dataset Count | Notes |
| ----: | ---------------------- | ------------: | ----- |
| 4     | `admin_department`     | 10            | Department-level coverage |
| 5     | `admin_arrondissement` | 42            | Arrondissement-level coverage |
| 8     | `admin_commune`        | 143           | Commune-level coverage |
| 10    | `admin_section`        | 569           | Communal section/local coverage |
| 11    | `admin_quarter`        | 3             | Quarter-level coverage where available |

The engine uses canonical names from `name:fr`, `name`, `official_name`, and `name:en`, with bounded multilingual aliases for `fr`, `ht`, and `en`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `HT/ht.admin/v1.0.0`

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
| Throughput                | `5093.590` QPS |
| Total Runtime             | `1.963 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,055 / 1 / 944 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,055 / 1 / 944 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,055 / 1 / 944 / 0 |
| no_nearby    | 100.00%   | 100.00%          | 0      | 0             | 9,001 / 1 / 998 / 0 |
| osm_only     | 100.00%   | 100.00%          | 0      | 0             | 9,001 / 1 / 998 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 54              |
| Total vs OSM-only | 54              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `ht.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape        | Count |
| ------------ | ----: |
| `[4,5,8,10]` | 8,997 |
| `[]`         | 994   |
| `[4,8,10]`  | 4     |
| `[4]`        | 2     |
| `[5,8,10]`  | 1     |
| `[4,5,10]`  | 1     |
| `[4,10]`    | 1     |

Empty shapes correspond to:

* `empty_shape`: 944
* `offshore`: 50

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 9,002 |
| admin_tree_id | 46    |
| nearby        | 4     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,956 |
| none                   | 994   |
| admin_tree_id\|polygon | 46    |
| nearby                 | 4     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,006 |
| empty_shape      | 944   |
| offshore         | 50    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `10`
* Total level-4 hits across all samples: `9,005`
* Total level-4 hits across inside samples: `8,999`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Département de l'Ouest | 1,694 | 16.94% | 1,692 | 18.80% |
| Département de l'Artibonite | 1,573 | 15.73% | 1,572 | 17.47% |
| Département du Centre | 1,165 | 11.65% | 1,164 | 12.93% |
| Département du Sud | 888 | 8.88% | 888 | 9.87% |
| Département du Nord | 717 | 7.17% | 716 | 7.96% |
| Département du Sud-Est | 705 | 7.05% | 705 | 7.83% |
| Département du Nord-Ouest | 704 | 7.04% | 703 | 7.81% |
| Département de la Grande-Anse | 598 | 5.98% | 598 | 6.64% |
| Département du Nord-Est | 562 | 5.62% | 562 | 6.24% |
| Département des Nippes | 399 | 3.99% | 399 | 4.43% |

Hit distribution reflects uniform sampling over the scoped administrative coverage.

---

# 11. Boundary Isolation Validation

The mixed test includes 1,000 outside samples. Under full policy:

| Metric | Value |
| ------ | ----: |
| Outside Samples | 1,000 |
| Outside Policy Failures | 0 |
| Offshore Samples | 50 |
| Failed Samples | 0 |

The result confirms the scoped Haiti dataset rejects out-of-scope points deterministically while permitting explicit offshore classification near the boundary where allowed by runtime policy.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,5,8,10]`, reflecting strong department, arrondissement, commune, and communal-section coverage.
3. Level 11 provides valid quarter-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `20cf7b41cd0c7e9aefafd29e8688c5e3a8a45db5`
- Cadis version:
  `0.8.57`
- Boundary builder: `scripts/build_ht_boundaries.py`
- Build boundary: `tmp/ht_country.json`
- Evaluation boundary: `tmp/ht_country.json`
- Staged dataset: `HT/ht.admin/v1.0.0`
- Source OSM SHA256:
  `4ebf62b545fa6b17e4870dabcc6b63cb37dffe24a4e82ab101c172ad12192cc7`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ht.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Haiti levels 4, 5, 8, 10, and 11

The dataset is suitable for human quality review.
