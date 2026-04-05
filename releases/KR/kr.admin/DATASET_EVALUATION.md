# KR_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `kr.admin`
Version: `v1.0.1`
Country: `KR`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `kr.admin v1.0.1` dataset under Cadis Runtime.

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
| Dataset ID              | `kr.admin` |
| Dataset Version         | `v1.0.1`   |
| Country                 | `KR`       |
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

| Category                               | Count |
| -------------------------------------- | ----- |
| Sample labels: expected inside points  | 9,000 |
| Sample labels: expected outside points | 1,000 |
| Structural non-empty outcomes          | 9,026 |
| Empty-shape outcomes (`[]`)            | 974   |
| Offshore outcomes (subset of `[]`)     | 11    |

Outside-labeled samples are intentional and do not indicate dataset coverage gaps.
`expected inside/outside` comes from Natural Earth sampling labels; structural outcomes come from Cadis runtime evaluation.

---

# 4. Performance Metrics

| Metric                    | Value          |
| ------------------------- | -------------- |
| Throughput                | `1294.180` QPS |
| Total Runtime             | `7.727 sec`    |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |

This run confirms no policy or coverage failures under full-policy runtime evaluation.

---

# 5. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed |
| ------------ | --------- | ---------------- | ------ |
| full_policy  | 100.00%   | 100.00%          | 0      |
| no_hierarchy | 100.00%   | 100.00%          | 0      |
| no_repair    | 100.00%   | 100.00%          | 0      |
| no_nearby    | 99.87%    | 99.86%           | 13     |
| osm_only     | 99.87%    | 99.86%           | 13     |

---

# 6. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 27              |
| Total vs OSM-only | 27              |

## Interpretation

* OSM-only success rate: `99.87%`
* Polygon coverage is already structurally strong across the sampled land area.
* Hierarchy supplementation is present but not required to prevent failures in this run.
* Nearby fallback resolves a small number of otherwise empty-shape interior cases.
* `nearby rescued = 27` is a scenario-delta metric (`full_policy` vs `no_nearby`), while `nearby source = 16` is direct observed node tagging. These are related but not identical measures.
* No geometric repair operations were triggered.

Observed OSM-only failures are caused by structural incompleteness, not geometric inconsistency.

---

# 7. Structural Distribution

## 7.1 Shape Distribution

| Shape   | Count |
| ------- | ----- |
| [4,6]   | 4,660 |
| [4,6,8] | 4,311 |
| []      | 974   |
| [4,8]   | 47    |
| [4]     | 8     |

Interpretation:

* Runtime resolution often terminates at Level-6.
* This indicates that Level-8 coverage is incomplete or unavailable for a meaningful subset of sampled land points.
* The observed result supports Level-8 structural incompleteness in runtime outcomes, but does not by itself prove that polygon incompleteness is the sole cause.

Empty shapes correspond to:

* `empty_shape`: 963
* `offshore`: 11

---

## 7.2 Node Source Distribution

| Source          | Count |
| --------------- | ----- |
| polygon         | 9,010 |
| nearby          | 16    |
| admin_tree_name | 6     |

### Source Mix

| Mix                     | Count |
| ----------------------- | ----- |
| polygon                 | 9,004 |
| none                    | 974   |
| nearby                  | 16    |
| admin_tree_name|polygon | 6     |

---

# 8. Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----- |
| shape_status_map | 9,026 |
| empty_shape      | 963   |
| offshore         | 11    |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `17`
* Total level-4 hits (all samples): `9,026`
* Total level-4 hits (inside samples): `9,000`

Hit distribution reflects uniform land-area sampling.

## 9.1 Top-10 Level-4 Hit Rates (Province/Metro)

| Level-4 Unit     | Hits  | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ---------------- | ----- | ---------------------- | --------------------- | ------------------------- |
| 경상북도         | 1,639 | 16.39%                 | 1,637                 | 18.19%                    |
| 강원특별자치도   | 1,580 | 15.80%                 | 1,577                 | 17.52%                    |
| 전라남도         | 1,021 | 10.21%                 | 1,014                 | 11.27%                    |
| 경상남도         | 978   | 9.78%                  | 977                   | 10.86%                    |
| 경기도           | 917   | 9.17%                  | 916                   | 10.18%                    |
| 전북특별자치도   | 755   | 7.55%                  | 754                   | 8.38%                     |
| 충청남도         | 716   | 7.16%                  | 714                   | 7.93%                     |
| 충청북도         | 674   | 6.74%                  | 674                   | 7.49%                     |
| 제주특별자치도   | 163   | 1.63%                  | 159                   | 1.77%                     |
| 대구광역시       | 130   | 1.30%                  | 130                   | 1.44%                     |

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation in this run.

This confirms strict boundary containment within the KR dataset.

---

# 11. Structural Observations

1. Geometry integrity is high; no repair layer activation.
2. Polygon coverage provides near-complete structural resolution across sampled interior land points.
3. Hierarchy supplementation is minimal and non-critical for pass/fail in this run.
4. Nearby fallback is small but materially rescues a limited coastal or adjacency subset.
5. Dataset achieves full inside-land coverage under full policy mode.
6. Boundary rejection behavior is strict and leak-free.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `5ff06d4d5aafb8b5bf958cbcc45ffb42013db285`
- cadis version:
  `v0.4.6`

The dataset package was generated from a clean working tree.
No local modifications were present at release time.

---

# 13. Conclusion

The `kr.admin v1.0.1` dataset demonstrates:

* High geometric integrity
* Very low structural incompleteness in sampled runtime behavior
* Minimal but useful nearby fallback contribution
* No need for geometric repair
* Full inside-boundary coverage under policy
* Strict cross-border isolation

OSM data is not incorrect.
It is occasionally incomplete.

Cadis does not modify geography.
It enforces structural determinism and boundary integrity.
