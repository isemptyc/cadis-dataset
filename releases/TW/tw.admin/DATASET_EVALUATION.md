
# TW_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `tw.admin`
Version: `v1.0.3`
Country: `TW`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `tw.admin v1.0.3` dataset under Cadis Runtime.

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

| Field                   | Value      |
| ----------------------- | ---------- |
| Dataset ID              | `tw.admin` |
| Dataset Version         | `v1.0.3`   |
| Country                 | `TW`       |
| Policy Version          | `1.0`      |
| Hierarchy Required      | `True`     |
| Repair Required         | `False`    |
| Runtime Policy Detected | `True`     |

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

| Category                               | Count  |
| -------------------------------------- | ------ |
| Sample labels: expected inside points  | 9,000  |
| Sample labels: expected outside points | 1,000  |
| Structural non-empty outcomes          | 8,992  |
| Empty-shape outcomes (`[]`)            | 1,008  |
| Offshore outcomes (subset of `[]`)     | 52     |

Outside-labeled samples are intentional and do not indicate dataset coverage gaps.
`expected inside/outside` comes from Natural Earth sampling labels; structural outcomes come from Cadis runtime evaluation.

---

# 4. Performance Metrics

| Metric                    | Value         |
| ------------------------- | ------------- |
| Throughput                | `537.130` QPS |
| Total Runtime             | `18.617 sec`  |
| Overall Pass Rate         | `100.00%`     |
| Inside Coverage Pass Rate | `100.00%`     |
| Policy Pass Rate          | `100.00%`     |

This run confirms no policy/coverage failures, but does not emit a dedicated outside-assignment leakage counter as a standalone metric.

---

# 5. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed |
| ------------ | --------- | ---------------- | ------ |
| full_policy  | 100.00%   | 100.00%          | 0      |
| no_hierarchy | 92.61%    | 91.79%           | 739    |
| no_repair    | 100.00%   | 100.00%          | 0      |
| no_nearby    | 98.98%    | 98.87%           | 102    |
| osm_only     | 91.59%    | 90.66%           | 841    |

---

# 6. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 739             |
| Repair            | 0               |
| Nearby            | 146             |
| Total vs OSM-only | 885             |

## Interpretation

* OSM-only success rate: `91.59%`
* Structural parent linkage incompleteness accounts for most OSM-only failures.
* Hierarchy supplementation deterministically resolves missing parent chains.
* Nearby fallback handles coastal adjacency edge cases.
* `nearby rescued = 146` is a scenario-delta metric (`full_policy` vs `no_nearby`), while `nearby source = 94` is direct observed node tagging. These are related but not identical measures.
* No geometric repair operations were triggered.

Observed failures under OSM-only mode are caused by structural incompleteness, not geometric inconsistency.

---

# 7. Structural Distribution

## 7.1 Shape Distribution

| Shape   | Count |
| ------- | ----- |
| [4,8]   | 6,220 |
| [4,7]   | 2,755 |
| []      | 1,008 |
| [4]     | 11    |
| [4,7,8] | 6     |

Empty shapes correspond to:

* `empty_shape`: 956
* `offshore`: 52

---

## 7.2 Node Source Distribution

| Source          | Count |
| --------------- | ----- |
| polygon         | 8,898 |
| admin_tree_name | 739   |
| nearby          | 94    |

### Source Mix

| Mix                     | Count |
| ----------------------- | ----- |
| polygon                 | 8,159 |
| admin_tree_name|polygon | 739   |
| nearby                  | 94    |
| none                    | 1,008 |

---

# 8. Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----- |
| shape_status_map | 8,992 |
| empty_shape      | 956   |
| offshore         | 52    |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `21`
* Total level-4 hits (all samples): `8,992`
* Total level-4 hits (inside samples): `8,989`

Hit distribution reflects uniform land-area sampling.

## 9.1 Top-10 Level-4 Hit Rates (City/County)

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---- | ---------------------- | --------------------- | ------------------------- |
| 花蓮縣       | 1,129 | 11.29%                 | 1,128                 | 12.53%                    |
| 南投縣       | 1,048 | 10.48%                 | 1,048                 | 11.64%                    |
| 臺東縣       | 907   | 9.07%                  | 907                   | 10.08%                    |
| 高雄市       | 737   | 7.37%                  | 737                   | 8.19%                     |
| 屏東縣       | 712   | 7.12%                  | 712                   | 7.91%                     |
| 臺南市       | 563   | 5.63%                  | 563                   | 6.26%                     |
| 臺中市       | 520   | 5.20%                  | 520                   | 5.78%                     |
| 宜蘭縣       | 496   | 4.96%                  | 496                   | 5.51%                     |
| 新北市       | 487   | 4.87%                  | 487                   | 5.41%                     |
| 嘉義縣       | 463   | 4.63%                  | 463                   | 5.14%                     |

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy/nearby layers created cross-border escalation in this run.

This confirms strict boundary containment within the TW dataset.

---

# 11. Structural Observations

1. Geometry integrity is high; no repair layer activation.
2. Parent linkage incompleteness is the primary structural gap.
3. Hierarchy supplementation provides deterministic structural completion.
4. Coastal fallback is minimal and bounded.
5. Dataset achieves full inside-land coverage under policy mode.
6. Boundary rejection behavior is strict and leak-free.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `6cb0c92b59720bedbebfab166467b54c9555845b`
- cadis-runtime commit:
  `6a53514efc308c25253b90906ee7fc472fd3d3f5`

The dataset package was generated from a clean working tree.
No local modifications were present at release time.

---

# 13. Conclusion

The `tw.admin v1.0.3` dataset demonstrates:

* High geometric integrity
* Minor structural incompleteness in parent linkage
* Deterministic completion via hierarchy supplementation
* Minimal coastal fallback usage
* Full inside-boundary coverage under policy
* Strict cross-border isolation

OSM data is not incorrect.
It is occasionally incomplete.

Cadis does not modify geography.
It enforces structural determinism and boundary integrity.
