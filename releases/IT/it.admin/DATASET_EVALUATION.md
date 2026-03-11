# IT_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `it.admin`
Version: `v1.0.0`
Country: `IT`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `it.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the observed runtime behavior of the packaged Italy dataset
* Quantifies structural incompleteness under random boundary-aware sampling
* Documents deterministic policy effects
* Validates boundary isolation under stress testing
* Provides reproducible integrity metrics

OSM-derived administrative data is not assumed geometrically invalid when hierarchy support is required.
Observed anomalies in this run reflect structural incompleteness or expected outside-boundary sampling outcomes.

Cadis does not alter geography.
It enforces deterministic structural interpretation on top of the packaged dataset.

---

# 2. Dataset Identity

| Field                   | Value                                                            |
| ----------------------- | ---------------------------------------------------------------- |
| Dataset ID              | `it.admin`                                                       |
| Dataset Version         | `v1.0.0`                                                         |
| Country                 | `IT`                                                             |
| Country Name            | `Italy`                                                          |
| Policy Version          | `1.0`                                                            |
| Hierarchy Required      | `True`                                                           |
| Repair Required         | `False`                                                          |
| Runtime Policy Detected | `True`                                                           |
| Dataset Dir             | `/Users/isempty/Projects/my_cadis/tmp/package/_release_stage/IT/it.admin/v1.0.0` |

---

# 3. Test Methodology

## 3.1 Sampling Strategy

* Total samples: `10,000`
* Lookup mode: `runtime`
* Sampling mode: mixed inside/outside stress testing
* Inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Boundary source: Natural Earth `ne_10m_admin_0_countries.shp`

The test intentionally injects about 10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification behavior
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over land area and is not population-weighted.

---

## 3.2 Observed Distribution

| Category                               | Count |
| -------------------------------------- | ----- |
| Sample labels: expected inside points  | 9,000 |
| Sample labels: expected outside points | 1,000 |
| Structural non-empty outcomes          | 9,002 |
| Empty-shape outcomes (`[]`)            | 998   |
| Offshore outcomes (subset of `[]`)     | 18    |

Outside-labeled samples are intentional and do not indicate dataset coverage gaps.
`expected inside/outside` comes from Natural Earth sampling labels; structural outcomes come from Cadis runtime evaluation.

---

# 4. Performance Metrics

| Metric                    | Value         |
| ------------------------- | ------------- |
| Throughput                | `387.330` QPS |
| Total Runtime             | `25.818 sec`  |
| Overall Pass Rate         | `100.00%`     |
| Inside Coverage Pass Rate | `100.00%`     |
| Policy Pass Rate          | `100.00%`     |

This run produced no policy failures and no inside-coverage failures across 10,000 sampled points.

---

# 5. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed |
| ------------ | --------- | ---------------- | ------ |
| full_policy  | 100.00%   | 100.00%          | 0      |
| no_hierarchy | 100.00%   | 100.00%          | 0      |
| no_repair    | 100.00%   | 100.00%          | 0      |
| no_nearby    | 99.53%    | 99.48%           | 47     |
| osm_only     | 99.53%    | 99.48%           | 47     |

---

# 6. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 67              |
| Total vs OSM-only | 67              |

## Interpretation

* OSM-only success rate is already high at `99.53%`.
* Hierarchy support changes some statuses from `ok` to `partial`, but does not rescue failed samples in this run.
* Nearby fallback is the only layer that materially improves pass/fail behavior in this sample set.
* `nearby rescued = 67` is a scenario-delta metric and is larger than direct `nearby` node-source tagging (`49`) because scenario effects and observed node-source attribution are related but not identical.
* No geometric repair operations were triggered.

Observed failures under `no_nearby` and `osm_only` are limited and are consistent with bounded coastal or adjacency edge cases rather than broad structural collapse.

---

# 7. Structural Distribution

## 7.1 Shape Distribution

| Shape   | Count |
| ------- | ----- |
| [4,6,8] | 8,882 |
| []      | 998   |
| [4,8]   | 101   |
| [4]     | 9     |
| [4,6]   | 6     |
| [8]     | 4     |

Empty shapes correspond to:

* `empty_shape`: 980
* `offshore`: 18

---

## 7.2 Node Source Distribution

| Source          | Count |
| --------------- | ----- |
| polygon         | 8,953 |
| nearby          | 49    |
| admin_tree_name | 13    |

### Source Mix

| Mix                     | Count |
| ----------------------- | ----- |
| polygon                 | 8,940 |
| none                    | 998   |
| nearby                  | 49    |
| admin_tree_name|polygon | 13    |

---

# 8. Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----- |
| shape_status_map | 9,002 |
| empty_shape      | 980   |
| offshore         | 18    |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `20`
* Total level-4 hits (all samples): `8,998`
* Total level-4 hits (inside samples): `8,996`

Hit distribution reflects uniform land-area sampling rather than administrative population weighting.
All 20 Italian level-4 regions were hit during random sampling, indicating complete sampled region-level coverage in this evaluation run.

## 9.1 Top-10 Level-4 Hit Rates

| Level-4 Unit         | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| -------------------- | ---- | ---------------------- | --------------------- | ------------------------- |
| Piemonte             | 840  | 8.40%                  | 840                   | 9.33%                     |
| Sicilia              | 752  | 7.52%                  | 752                   | 8.36%                     |
| Lombardia            | 717  | 7.17%                  | 717                   | 7.97%                     |
| Toscana              | 711  | 7.11%                  | 710                   | 7.89%                     |
| Sardegna             | 666  | 6.66%                  | 666                   | 7.40%                     |
| Emilia-Romagna       | 665  | 6.65%                  | 664                   | 7.38%                     |
| Veneto               | 557  | 5.57%                  | 557                   | 6.19%                     |
| Puglia               | 533  | 5.33%                  | 533                   | 5.92%                     |
| Lazio                | 492  | 4.92%                  | 492                   | 5.47%                     |
| Calabria             | 442  | 4.42%                  | 442                   | 4.91%                     |

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers caused cross-border escalation in this run.

This sampled run confirms strict boundary containment within the Italy dataset.

---

# 11. Structural Observations

1. Geometry integrity appears stable in runtime behavior; no repair layer activation occurred.
2. The dominant valid shape is `[4,6,8]`, indicating broad hierarchical completeness for sampled in-bound points.
3. Hierarchy supplementation is present but only lightly used in this run.
4. Nearby fallback is low-volume but operationally important for a small number of edge cases.
5. Dataset achieves full sampled inside-land coverage under full policy mode.
6. Boundary rejection behavior is strict and leak-free in this evaluation.

---

# 12. Reproducibility

All evaluation results in this report are reproducible using:

- cadis-dataset-engine commit:
  `493860cf9f3e8a4d02ca91a50db4c8a9ed6c69f9`
- cadis commit:
  `c3329ea493039a45716749487713d83645dc1c99`

The dataset package was generated from a clean working tree.
No local modifications were present at release time.

---

# 13. Conclusion

The `it.admin v1.0.0` dataset passes runtime mass validation cleanly under the Cadis policy model.
Across 10,000 sampled points, the dataset achieved:

* `100.00%` overall pass rate
* `100.00%` inside coverage pass rate
* Strict sampled boundary containment
* No observed need for repair-layer intervention

Italy therefore appears release-ready from the perspective of runtime lookup integrity and boundary isolation.
