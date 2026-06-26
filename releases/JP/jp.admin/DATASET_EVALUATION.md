# JP_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `jp.admin`
Version: `v1.0.3`
Country: `JP`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `jp.admin v1.0.3` dataset under Cadis Runtime.

This report:

* Describes the condition of the source OSM data
* Quantifies structural incompleteness
* Documents deterministic policy effects
* Validates boundary isolation under stress testing
* Provides reproducible integrity metrics

OSM data is not incorrect.
Observed anomalies reflect structural incompleteness, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value       |
| ----------------------- | ----------- |
| Dataset ID              | `jp.admin`  |
| Dataset Version         | `v1.0.3`    |
| Country                 | `JP`        |
| Policy Version          | `1.0`       |
| Hierarchy Required      | `True`      |
| Repair Required         | `False`     |
| Runtime Policy Detected | `True`      |

---

# 3. Test Methodology

## 3.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside ratio: `0.9`
* Expected outside ratio: `0.1`

The test intentionally injects ~10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over land area (not population-weighted).

---

## 3.2 Observed Distribution

| Category                               | Count |
| -------------------------------------- | ----- |
| Sample labels: expected inside points  | 9,000 |
| Sample labels: expected outside points | 1,000 |
| Structural non-empty outcomes          | 9,004 |
| Empty-shape outcomes (`[]`)            | 996   |
| Offshore outcomes (subset of `[]`)     | 4     |

Outside-labeled samples are intentional and do not indicate dataset coverage gaps.
`expected inside/outside` comes from Natural Earth sampling labels; structural outcomes come from Cadis runtime evaluation.

---

# 4. Performance Metrics

| Metric                    | Value          |
| ------------------------- | -------------- |
| Throughput                | `2242.120` QPS |
| Total Runtime             | `4.460 sec`    |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |

---

# 5. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed |
| ------------ | --------- | ---------------- | ------ |
| full_policy  | 100.00%   | 100.00%          | 0      |
| no_hierarchy | 100.00%   | 100.00%          | 0      |
| no_repair    | 100.00%   | 100.00%          | 0      |
| no_nearby    | 100.00%   | 100.00%          | 0      |
| osm_only     | 100.00%   | 100.00%          | 0      |

---

# 6. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 4               |
| Total vs OSM-only | 4               |

## Interpretation

* OSM-only success rate: `100.00%`
* Geometry coverage for Japan is high; direct polygon hits cover all 9,000 inside points.
* Nearby fallback rescued 4 offshore samples — coastal points that fall just outside land polygons due to Natural Earth boundary precision at narrow coastlines and island edges.
* No hierarchy supplementation was required; parent linkage is complete in the JP OSM dataset.
* No geometric repair operations were triggered.

The 4 offshore-rescued points confirm the v1.0.3 fix: all 47 prefectures now contribute to country scope geometry, enabling offshore resolution across the full territory including Okinawa and remote island chains.

---

# 7. Structural Distribution

## 7.1 Shape Distribution

| Shape     | Count |
| --------- | ----- |
| [3, 4, 7] | 9,004 |
| []        | 996   |

Empty shapes correspond to:

* `empty_shape`: 992
* `offshore`: 4

---

## 7.2 Node Source Distribution

| Source        | Count |
| ------------- | ----- |
| polygon       | 9,003 |
| admin_tree_id | 6,626 |
| nearby        | 1     |

### Source Mix

| Mix                       | Count |
| ------------------------- | ----- |
| admin_tree_id \| polygon  | 6,625 |
| polygon                   | 2,378 |
| admin_tree_id \| nearby   | 1     |
| none                      | 996   |

---

# 8. Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----- |
| shape_status_map | 9,004 |
| empty_shape      | 992   |
| offshore         | 4     |

---

# 9. Level-4 Coverage

* Unique level-4 prefectures hit: `47` (all 47 prefectures)
* Total level-4 hits (all samples): `9,004`
* Total level-4 hits (inside samples): `9,000`

Full prefecture coverage is confirmed. All 47 都道府県 were hit in a 10,000-sample run.

## 9.1 Top-10 Level-4 Hit Rates (Prefecture)

| Prefecture | Hits  | Hit Rate (All) | Hits (Inside) | Hit Rate (Inside) |
| ---------- | ----- | -------------- | ------------- | ----------------- |
| 北海道     | 2,378 | 23.78%         | 2,378         | 26.42%            |
| 岩手県     | 333   | 3.33%          | 333           | 3.70%             |
| 鹿児島県   | 297   | 2.97%          | 296           | 3.29%             |
| 福島県     | 255   | 2.55%          | 255           | 2.83%             |
| 長野県     | 251   | 2.51%          | 251           | 2.79%             |
| 熊本県     | 250   | 2.50%          | 250           | 2.78%             |
| 大分県     | 250   | 2.50%          | 250           | 2.78%             |
| 秋田県     | 240   | 2.40%          | 240           | 2.67%             |
| 宮崎県     | 237   | 2.37%          | 236           | 2.62%             |
| 新潟県     | 230   | 2.30%          | 230           | 2.56%             |

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence of hierarchy or nearby layers creating cross-border escalation.

This confirms strict boundary containment within the JP dataset.

---

# 11. Structural Observations

1. Geometry integrity is high; no repair layer activation.
2. Parent linkage is complete; hierarchy layer was not needed for any sample.
3. Japan's OSM data has only one admin_level=3 polygon (北海道地方 / Hokkaido region). The remaining 7 regional groupings (地方) are not represented as OSM polygons. This is expected: Japanese regional groupings are informal and not administratively defined at the OSM admin_level=3 boundary.
4. `country_scope_flag` is set on all 47 level-4 prefectures so that offshore resolution applies to the full territory, including Okinawa and the Ryukyu island chain.
5. The 4 offshore samples confirm that the scope fix correctly extends coverage to points near Japan's many coastal and island boundaries.
6. Dataset achieves full inside-land coverage under all tested policy scenarios.
7. Boundary rejection is strict and leak-free.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- Cadis version: `v0.9.0`

The dataset package was generated from a clean working tree.
No local modifications were present at release time.

---

# 13. Conclusion

The `jp.admin v1.0.3` dataset demonstrates:

* High geometric integrity across all 47 prefectures
* Complete parent linkage; no hierarchy supplementation needed
* Effective offshore resolution across the full Japanese territory
* Minimal coastal fallback usage (4 samples)
* Full inside-boundary coverage under all policy scenarios
* Strict cross-border isolation

Key change from v1.0.1: `country_scope_flag` is now set on all level-4 prefecture features. This ensures the offshore resolver correctly covers remote prefectures (e.g. 沖縄県, 鹿児島県 island chains) that fall outside Natural Earth land boundaries but are within Japan's 20 km offshore policy threshold.

OSM data is not incorrect.
It is occasionally incomplete at the regional grouping level.

Cadis does not modify geography.
It enforces structural determinism and boundary integrity.
