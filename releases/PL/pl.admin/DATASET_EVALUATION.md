# PL_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `pl.admin`
Version: `v1.0.0`
Country: `PL`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `pl.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the selected Poland administrative contract
* Documents boundary scoping and source inputs
* Quantifies runtime coverage under mixed inside/outside sampling
* Validates deterministic policy behavior before publish

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value      |
| ----------------------- | ---------- |
| Dataset ID              | `pl.admin` |
| Dataset Version         | `v1.0.0`   |
| Country                 | `PL`       |
| Country Name            | `Poland`   |
| Policy Version          | `1.0`      |
| Hierarchy Required      | `True`     |
| Repair Required         | `False`    |
| Runtime Policy Detected | `True`     |
| Minimum Cadis Version   | `0.8.6`    |

---

# 3. Source and Scope

| Field                  | Value |
| ---------------------- | ----- |
| OSM Source Label       | `geofabrik:europe/Poland` |
| OSM Snapshot Timestamp | `2026-04-26T20:21:04Z` |
| Dataset Scope          | Full ISO2 country |
| Boundary Source        | Natural Earth admin-0 country boundary for `PL` |
| Boundary BBox          | `[14.12392297300002, 48.994013164000094, 24.143156372000107, 54.838324286000045]` |
| Boundary Polygon Count | `1` |

---

# 4. Administrative Contract

The Poland engine exposes the stable three-level civil administrative model observed in the scoped build:

| Level | Label | Count |
| ----- | ----- | ----: |
| 4 | Voivodeship | 16 |
| 6 | County / city-county | 377 |
| 7 | Gmina | 2,470 |

Level 8 was intentionally excluded from the first release contract. A scoped probe found 26,409 level-8 features and a much larger intermediate dataset; level 4/6/7 produced complete parent coverage and a compact runtime package.

---

# 5. Test Methodology

* Total samples: `100,000`
* Sampling mode: mixed inside/outside stress testing
* Inside ratio: `0.9`
* Expected inside points: `90,000`
* Expected outside points: `10,000`
* Evaluation boundary: same generated PL Natural Earth boundary used for build scoping
* Runtime mode: local Cadis runtime against staged dataset
* Evaluation workers: `1`

The single-worker evaluation path was used so every sampled point is counted by the current tester implementation.

---

# 6. Performance and Pass Metrics

| Metric                    | Value           |
| ------------------------- | --------------- |
| Throughput                | `11836.800` QPS |
| Total Runtime             | `8.448 sec`     |
| Overall Pass Rate         | `100.00%`       |
| Policy Pass Rate          | `100.00%`       |
| Inside Coverage Pass Rate | `100.00%`       |
| Failed Samples            | `0`             |
| Inside Failures           | `0`             |

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| ------------ | --------- | ---------------- | -----: | ------------: |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             |
| no_nearby    | 99.46%    | 99.40%           | 538    | 538           |
| osm_only     | 99.46%    | 99.40%           | 538    | 538           |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------: |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 680             |
| Total vs OSM-only | 680             |

No explicit repair layer is required for the initial Poland contract. Nearby fallback accounts for bounded coastal and edge cases.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape   | Count |
| ------- | ----: |
| [4,6,7] | 89,419 |
| []      | 10,115 |
| [4]     | 213 |
| [4,6]   | 156 |
| [4,7]   | 90 |
| [6,7]   | 7 |

Empty shapes correspond to expected outside or offshore samples and did not create inside coverage failures.

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 89,465 |
| nearby        | 420 |
| admin_tree_id | 111 |

## 9.3 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 89,885 |
| empty_shape      | 9,855 |
| offshore         | 260 |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `16`
* Total level-4 hits: `89,878`
* Total inside level-4 hits: `89,869`

All 16 voivodeships were exercised during the sampled run.

| Voivodeship | Hits | Hit Rate | Inside Hits | Inside Hit Rate |
| ----------- | ---: | -------: | ----------: | --------------: |
| `województwo mazowieckie` | 10,331 | 10.33% | 10,331 | 11.48% |
| `województwo wielkopolskie` | 8,504 | 8.50% | 8,504 | 9.45% |
| `województwo warmińsko-mazurskie` | 7,374 | 7.37% | 7,374 | 8.19% |
| `województwo lubelskie` | 7,015 | 7.02% | 7,014 | 7.79% |
| `województwo zachodniopomorskie` | 6,649 | 6.65% | 6,647 | 7.39% |
| `województwo podlaskie` | 5,862 | 5.86% | 5,861 | 6.51% |
| `województwo dolnośląskie` | 5,568 | 5.57% | 5,568 | 6.19% |
| `województwo pomorskie` | 5,531 | 5.53% | 5,530 | 6.14% |
| `województwo kujawsko-pomorskie` | 5,240 | 5.24% | 5,240 | 5.82% |
| `województwo łódzkie` | 5,224 | 5.22% | 5,224 | 5.80% |
| `województwo podkarpackie` | 4,966 | 4.97% | 4,964 | 5.52% |
| `województwo małopolskie` | 4,266 | 4.27% | 4,266 | 4.74% |
| `województwo lubuskie` | 3,938 | 3.94% | 3,937 | 4.37% |
| `województwo śląskie` | 3,579 | 3.58% | 3,579 | 3.98% |
| `województwo świętokrzyskie` | 3,186 | 3.19% | 3,186 | 3.54% |
| `województwo opolskie` | 2,645 | 2.65% | 2,644 | 2.94% |

---

# 11. Boundary Isolation Validation

Under mixed sampling with 10% expected outside points:

* No inside-boundary failures were observed.
* Empty-shape outcomes were limited to outside/offshore classification.
* No evidence was observed that hierarchy or nearby behavior caused cross-border escalation.
* Build and evaluation used the same generated PL boundary.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `a206088906fd906432c19ba33ffe10aa9c8c1276`
- Cadis version:
  `0.8.6`

The staged dataset was generated from a clean `cadis_dataset_engine` working tree.

---

# 13. Conclusion

The `pl.admin v1.0.0` dataset is suitable for publish review:

* Stable level 4/6/7 administrative contract
* Full sampled inside-boundary coverage
* Strict mixed-sample boundary behavior
* No required explicit repair layer
* Bounded nearby fallback contribution
