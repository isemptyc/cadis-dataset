# SV_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `sv.admin`
Version: `v1.0.0`
Country: `SV`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `sv.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the El Salvador dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `sv.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `SV`              |
| Country Name            | `El Salvador`     |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.55`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_sv_boundaries.py`       |
| Generated Boundary       | `tmp/sv_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4 and 5 whose representative point is covered by the Natural Earth SV boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox            | `[-90.13528067428713, 13.145996909078313, -87.69348244205072, 14.44831625527712]` |
| OSM Source               | `geofabrik:central-america/el-salvador` |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is El Salvador administrative coverage represented by materialized OSM department and region relations. The country envelope is excluded from the initial runtime contract.

---

# 4. Administrative Model

The initial El Salvador engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_department`   | 14            | Department-level coverage |
| 5     | `admin_region`       | 44            | Newer regional/district grouping coverage |
| 6     | `admin_municipality` | 262           | Municipality-level coverage |
| 7     | `admin_canton`       | 125           | Canton/local coverage where available |
| 8     | `admin_district`     | 2             | District coverage where available |
| 9     | `admin_neighborhood` | 14            | Neighborhood coverage where available |
| 10    | `admin_detail`       | 1             | Detail coverage where available |

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
* Dataset path: `SV/sv.admin/v1.0.0`

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
| Throughput                | `9602.510` QPS |
| Total Runtime             | `1.041 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,040 / 1 / 959 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,040 / 1 / 959 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,040 / 1 / 959 / 0 |
| no_nearby    | 99.99%    | 99.99%           | 1      | 1             | 8,999 / 1 / 1,000 / 0 |
| osm_only     | 99.99%    | 99.99%           | 1      | 1             | 8,999 / 1 / 1,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 41              |
| Total vs OSM-only | 41              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `sv.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape           | Count |
| --------------- | ----: |
| `[4,5,6]`       | 8,538 |
| `[]`            | 993   |
| `[4,5,6,7]`     | 352   |
| `[4,5]`         | 96    |
| `[4,5,6,8]`     | 9     |
| `[4]`           | 5     |
| `[4,6]`         | 2     |
| `[4,5,6,7,8]`  | 2     |

Empty shapes correspond to:

* `empty_shape`: 959
* `offshore`: 34

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 9,000 |
| admin_tree_id | 37    |
| nearby        | 7     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,963 |
| none                   | 993   |
| admin_tree_id\|polygon | 37    |
| nearby                 | 7     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,007 |
| empty_shape      | 959   |
| offshore         | 34    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `14`
* Total level-4 hits across all samples: `9,006`
* Total level-4 hits across inside samples: `8,999`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Usulután | 942 | 9.42% | 941 | 10.46% |
| San Miguel | 934 | 9.34% | 934 | 10.38% |
| Chalatenango | 889 | 8.89% | 889 | 9.88% |
| Santa Ana | 887 | 8.87% | 887 | 9.86% |
| La Unión | 836 | 8.36% | 832 | 9.24% |
| La Libertad | 732 | 7.32% | 732 | 8.13% |
| Morazán | 576 | 5.76% | 576 | 6.40% |
| La Paz | 533 | 5.33% | 533 | 5.92% |
| Ahuachapán | 520 | 5.20% | 520 | 5.78% |
| Sonsonate | 519 | 5.19% | 518 | 5.76% |

Hit distribution reflects uniform sampling over the scoped administrative coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/sv_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the El Salvador dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,5,6]`, reflecting strong department, region, and municipality coverage.
3. Levels 7, 8, 9, and 10 provide valid lower-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `260cb695c6f4a83a83ad48480cb1ae72345c153e`
- Cadis version:
  `0.8.55`
- Boundary builder: `scripts/build_sv_boundaries.py`
- Build boundary: `tmp/sv_country.json`
- Evaluation boundary: `tmp/sv_country.json`
- Staged dataset: `SV/sv.admin/v1.0.0`
- Source OSM SHA256:
  `b32cbe98c40422e7eee60d55cec21460f24020e295c7cfd5c71366585289c6d2`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `sv.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for El Salvador levels 4, 5, 6, 7, 8, 9, and 10

The dataset is suitable for human quality review.
