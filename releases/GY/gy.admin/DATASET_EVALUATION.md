# GY_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `gy.admin`
Version: `v1.0.0`
Country: `GY`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `gy.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Guyana dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `gy.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `GY`              |
| Country Name            | `Guyana`          |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.46`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_gy_boundaries.py`       |
| Generated Boundary       | `tmp/gy_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth GY admin-0 country boundary.` |
| Boundary BBox            | `[-61.39671280899992, 1.1858202110000775, -56.481819010999914, 8.558010158000059]` |
| OSM Source               | `geofabrik:south-america/guyana`       |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Guyana engine exposes these OSM administrative levels:

| Level | Runtime Label          | Dataset Count | Notes |
| ----: | ---------------------- | ------------: | ----- |
| 4     | `admin_region`         | 10            | Regions |
| 6     | `admin_district`       | 114           | District/lower regional coverage |
| 8     | `admin_local_district` | 145           | Local district coverage where available |
| 10    | `admin_neighborhood`   | 77            | Neighborhood/detail coverage |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 3     | 0           | Excluded because no scoped features were present |
| 5     | 0           | Excluded because no scoped features were present |
| 7     | 0           | Excluded because no scoped features were present |
| 9     | 0           | Excluded because no scoped features were present |

The engine uses canonical names from `name:en`, `name`, and `official_name`, with bounded multilingual aliases for `en`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `GY/gy.admin/v1.0.0`

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
| Throughput                | `19477.760` QPS |
| Total Runtime             | `0.513 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 60 / 8,952 / 988 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 60 / 8,952 / 988 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 60 / 8,952 / 988 / 0 |
| no_nearby    | 99.44%    | 99.38%           | 56     | 56            | 47 / 8,897 / 1,056 / 0 |
| osm_only     | 99.44%    | 99.38%           | 56     | 56            | 47 / 8,897 / 1,056 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 68              |
| Total vs OSM-only | 68              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `gy.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape       | Count |
| ----------- | ----: |
| `[4,6]`     | 8,927 |
| `[]`        | 1,001 |
| `[4,6,8]`  | 47    |
| `[4]`       | 22    |
| `[4,6,10]` | 3     |

Empty shapes correspond to:

* `empty_shape`: 988
* `offshore`: 13

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,944 |
| nearby        | 55    |
| admin_tree_id | 16    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,928 |
| none                   | 1,001 |
| nearby                 | 55    |
| admin_tree_id\|polygon | 16    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 8,999 |
| empty_shape      | 988   |
| offshore         | 13    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `10`
* Total level-4 hits across all samples: `8,999`
* Total level-4 hits across inside samples: `8,998`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Upper Takutu-Upper Essequibo | 2,321 | 23.21% | 2,321 | 25.79% |
| Cuyuni-Mazaruni | 2,085 | 20.85% | 2,085 | 23.17% |
| East Berbice-Corentyne | 1,628 | 16.28% | 1,627 | 18.08% |
| Potaro-Siparuni | 881 | 8.81% | 881 | 9.79% |
| Barima-Waini | 785 | 7.85% | 785 | 8.72% |
| Upper Demerara-Berbice | 651 | 6.51% | 651 | 7.23% |
| Pomeroon-Supenaam | 256 | 2.56% | 256 | 2.84% |
| Mahaica-Berbice | 155 | 1.55% | 155 | 1.72% |
| Essequibo Islands | 151 | 1.51% | 151 | 1.68% |
| Demerara-Mahaica | 86 | 0.86% | 86 | 0.96% |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/gy_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Guyana dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,6]`, reflecting broad region and district coverage.
3. Level 8 and level 10 are present but sparse in sampled lookup results.
4. Most successful inside lookups are marked `partial` because the dominant source shape does not include level 8.
5. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `510abf0e93935677264854c0ab8f30766ed1ecbe`
- Cadis version:
  `0.8.46`
- Boundary builder: `scripts/build_gy_boundaries.py`
- Build boundary: `tmp/gy_country.json`
- Evaluation boundary: `tmp/gy_country.json`
- Staged dataset: `GY/gy.admin/v1.0.0`
- Source OSM SHA256:
  `246fab981c14cb538333453fd5f597c2e06228fb8b16a0743490368627796067`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `gy.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Guyana levels 4, 6, 8, and 10

The dataset is suitable for human quality review.
