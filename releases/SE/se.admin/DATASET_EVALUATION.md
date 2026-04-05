# SE_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `se.admin`
Version: `v1.0.1`
Country: `SE`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `se.admin v1.0.1` dataset under Cadis Runtime.

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
| Dataset ID              | `se.admin` |
| Dataset Version         | `v1.0.1`   |
| Country                 | `SE`       |
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
| Structural non-empty outcomes          | 9,008  |
| Empty-shape outcomes (`[]`)            | 992    |
| Offshore outcomes (subset of `[]`)     | 9      |

Outside-labeled samples are intentional and do not indicate dataset coverage gaps.
`expected inside/outside` comes from Natural Earth sampling labels; structural outcomes come from Cadis runtime evaluation.

---

# 4. Performance Metrics

| Metric                    | Value          |
| ------------------------- | -------------- |
| Throughput                | `2155.710` QPS |
| Total Runtime             | `4.639 sec`    |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |

This run confirms no policy/coverage failures under full policy mode.

---

# 5. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed |
| ------------ | --------- | ---------------- | ------ |
| full_policy  | 100.00%   | 100.00%          | 0      |
| no_hierarchy | 100.00%   | 100.00%          | 0      |
| no_repair    | 100.00%   | 100.00%          | 0      |
| no_nearby    | 99.89%    | 99.88%           | 11     |
| osm_only     | 99.89%    | 99.88%           | 11     |

---

# 6. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 20              |
| Total vs OSM-only | 20              |

## Interpretation

* OSM-only success rate: `99.89%`
* Sweden OSM structure is already highly complete at the county/municipality chain.
* Hierarchy supplementation does not rescue failures in this sample, but it upgrades `4` samples from `partial` to `ok`.
* Nearby fallback handles a small but real coastal/island edge case population and is the only layer affecting pass/fail in this run.
* `nearby rescued = 20` is a scenario-delta metric (`full_policy` vs `no_nearby`), while `nearby source = 11` is direct observed node tagging. These are related but not identical measures.
* No geometric repair operations were triggered.

Observed failures under no-nearby / OSM-only mode are caused by structural edge-case incompleteness, not geometric inconsistency.

---

# 7. Structural Distribution

## 7.1 Shape Distribution

| Shape | Count |
| ----- | ----- |
| [4,7] | 9,006 |
| []    | 992   |
| [4]   | 2     |

Empty shapes correspond to:

* `empty_shape`: 983
* `offshore`: 9

The two `[4]` outcomes are policy-valid partial county-only results.

---

## 7.2 Node Source Distribution

| Source          | Count |
| --------------- | ----- |
| polygon         | 8,997 |
| nearby          | 11    |
| admin_tree_name | 4     |

### Source Mix

| Mix                     | Count |
| ----------------------- | ----- |
| polygon                 | 8,993 |
| admin_tree_name|polygon | 4     |
| nearby                  | 11    |
| none                    | 992   |

---

# 8. Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----- |
| shape_status_map | 9,008 |
| empty_shape      | 983   |
| offshore         | 9     |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `21`
* Total level-4 hits (all samples): `9,008`
* Total level-4 hits (inside samples): `8,999`

Hit distribution reflects uniform land-area sampling.

## 9.1 Top-10 Level-4 Hit Rates (County)

| Level-4 Unit         | Hits  | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| -------------------- | ----- | ---------------------- | --------------------- | ------------------------- |
| Norrbottens län      | 2,467 | 24.67%                 | 2,465                 | 27.39%                    |
| Västerbottens län    | 1,249 | 12.49%                 | 1,247                 | 13.86%                    |
| Jämtlands län        | 1,107 | 11.07%                 | 1,107                 | 12.30%                    |
| Dalarnas län         | 592   | 5.92%                  | 592                   | 6.58%                     |
| Västra Götalands län | 513   | 5.13%                  | 512                   | 5.69%                     |
| Västernorrlands län  | 462   | 4.62%                  | 462                   | 5.13%                     |
| Värmlands län        | 403   | 4.03%                  | 403                   | 4.48%                     |
| Gävleborgs län       | 391   | 3.91%                  | 391                   | 4.34%                     |
| Östergötlands län    | 196   | 1.96%                  | 196                   | 2.18%                     |
| Kalmar län           | 190   | 1.90%                  | 189                   | 2.10%                     |

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy/nearby layers created cross-border escalation in this run.

This confirms strict boundary containment within the SE dataset.

---

# 11. Structural Observations

1. Geometry integrity is high; no repair layer activation.
2. County to municipality parent linkage is already structurally strong in source data.
3. Hierarchy supplementation is low-impact for coverage and mainly improves completeness classification.
4. Nearby fallback is minimal but operationally important for a small set of coastal/island edge cases.
5. Dataset achieves full inside-boundary coverage under policy mode.
6. Boundary rejection behavior is strict and leak-free.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `5ff06d4d5aafb8b5bf958cbcc45ffb42013db285`
- cadis version:
  `v0.4.6`

---

# 13. Conclusion

The `se.admin v1.0.1` dataset demonstrates:

* High geometric integrity
* Strong county/municipality structural completeness
* Minimal hierarchy supplementation usage
* Small but meaningful nearby fallback usage
* Full inside-boundary coverage under policy
* Strict cross-border isolation

OSM data is not incorrect.
It is occasionally incomplete at runtime edge cases.

Cadis does not modify geography.
It enforces structural determinism and boundary integrity.
