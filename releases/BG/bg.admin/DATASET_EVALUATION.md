# BG_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `bg.admin`
Version: `v1.0.0`
Country: `BG`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `bg.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Bulgaria dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `bg.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `BG`              |
| Country Name            | `Bulgaria`        |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.26`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_bg_boundaries.py`       |
| Generated Boundary       | `tmp/bg_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth BG admin-0 country boundary.` |
| Boundary BBox            | `[22.345023234000053, 41.238104147000044, 28.603526238000086, 44.22843453899999]` |
| OSM Source               | `geofabrik:europe/bulgaria`            |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Bulgaria engine exposes these OSM administrative levels:

| Level | Runtime Label          | Probe Count | Notes |
| ----: | ---------------------- | ----------: | ----- |
| 4     | `admin_province`       | 28          | Provinces |
| 5     | `admin_municipality`   | 265         | Municipalities |
| 6     | `admin_city_district`  | 35          | City districts in major municipalities |
| 8     | `admin_settlement`     | 1,983       | Settlements |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 9     | 137         | Excluded as sparse neighborhood/microdistrict coverage |
| 10    | 7           | Excluded as sparse microdistrict coverage |

The engine uses canonical names from `name:bg`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `bg` and `en`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `BG/bg.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over land area, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value           |
| ------------------------- | --------------- |
| Overall Pass Rate         | `100.00%`       |
| Inside Coverage Pass Rate | `100.00%`       |
| Policy Pass Rate          | `100.00%`       |
| Failed Samples            | `0`             |
| Throughput                | `11598.840` QPS |
| Total Runtime             | `0.862 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,025 / 6 / 969 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,025 / 6 / 969 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,025 / 6 / 969 / 0 |
| no_nearby    | 98.25%    | 98.06%           | 175    | 175           | 8,819 / 6 / 1,175 / 0 |
| osm_only     | 98.25%    | 98.06%           | 175    | 175           | 8,819 / 6 / 1,175 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 206             |
| Total vs OSM-only | 206             |

## Interpretation

* OSM-only success rate: `98.25%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves boundary-adjacent and coastal-edge samples.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `bg.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape       | Count |
| ----------- | ----: |
| `[4,5]`     | 4,775 |
| `[4,5,8]`   | 4,017 |
| `[]`        | 1,051 |
| `[4,5,6,8]` | 103  |
| `[4]`       | 25    |
| `[4,8]`     | 13    |
| `[4,5,6]`   | 10    |
| `[5,8]`     | 3     |
| `[8]`       | 3     |

Empty shapes correspond to:

* `empty_shape`: 969
* `offshore`: 82

## 9.2 Node Source Distribution

| Source          | Count |
| --------------- | ----: |
| polygon         | 8,825 |
| nearby          | 124   |
| admin_tree_id   | 17    |

## 9.3 Source Mix Distribution

| Mix                   | Count |
| --------------------- | ----: |
| polygon               | 8,808 |
| none                  | 1,051 |
| nearby                | 124   |
| admin_tree_id\|polygon | 17    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 8,949 |
| empty_shape      | 969   |
| offshore         | 82    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `28`
* Total level-4 hits across all samples: `8,943`
* Total level-4 hits across inside samples: `8,943`

| Level-4 Unit       | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Бургас             | 605  | 6.05%                   | 605                   | 6.72%                      |
| Софийска           | 556  | 5.56%                   | 556                   | 6.18%                      |
| Благоевград        | 494  | 4.94%                   | 494                   | 5.49%                      |
| Пловдив            | 472  | 4.72%                   | 472                   | 5.24%                      |
| Хасково            | 437  | 4.37%                   | 437                   | 4.86%                      |
| Плевен             | 416  | 4.16%                   | 416                   | 4.62%                      |
| Добрич             | 394  | 3.94%                   | 394                   | 4.38%                      |
| Велико Търново     | 392  | 3.92%                   | 392                   | 4.36%                      |
| Стара Загора       | 386  | 3.86%                   | 386                   | 4.29%                      |
| Ловеч              | 350  | 3.50%                   | 350                   | 3.89%                      |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/bg_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Bulgaria dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shapes are `[4,5]` and `[4,5,8]`, reflecting province/municipality coverage with settlement detail where available.
3. Level 6 city districts appear only in major cities and are represented in a smaller number of sampled outcomes.
4. Nearby fallback is bounded and accounts for the only meaningful rescue effect.
5. Levels 9 and 10 are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `2eceaf28bb0ce16f3a42c9344df8c92ed08209ff`
- Cadis version:
  `0.8.26`
- Boundary builder: `scripts/build_bg_boundaries.py`
- Build boundary: `tmp/bg_country.json`
- Evaluation boundary: `tmp/bg_country.json`
- Staged dataset: `BG/bg.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `bg.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Bulgaria levels 4, 5, 6, and 8

The dataset is suitable for human quality review.
