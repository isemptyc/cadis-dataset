# CL_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `cl.admin`
Version: `v1.0.0`
Country: `CL`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `cl.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the selected Chile dataset scope
* Quantifies structural completeness under randomized lookup stress testing
* Documents deterministic policy effects
* Validates boundary isolation against the declared evaluation boundary
* Provides reproducible integrity metrics for release review

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value      |
| ----------------------- | ---------- |
| Dataset ID              | `cl.admin` |
| Dataset Version         | `v1.0.0`   |
| Country                 | `CL`       |
| Country Name            | `Chile`    |
| Policy Version          | `1.0`      |
| Hierarchy Required      | `True`     |
| Repair Required         | `False`    |
| Runtime Policy Detected | `True`     |
| Cadis Version           | `0.8.21`   |

---

# 3. Dataset Scope

The `cl.admin v1.0.0` scope is Chile administrative coverage derived from OSM administrative levels 4 and 6 within the CL Natural Earth admin-0 country boundary.

The tracked boundary builder is:

* `scripts/build_cl_boundaries.py`

Generated boundary files:

* Natural Earth CL boundary: `tmp/cl_country_ne.json`
* Evaluation/build boundary: `tmp/cl_country.json`

The deterministic boundary selection rule is:

* union OSM administrative relation areas at levels 4 and 6 whose representative point is covered by the Natural Earth CL boundary

Level 6 provinces are included in the scope boundary because the source PBF has province coverage in Los Lagos where the level 4 regional polygon is absent. This prevents Natural Earth coastline/island sampling from being treated as an administrative coverage failure when no released admin polygon exists there.

Generated scoped boundary bbox:

* `[-109.4548678, -56.5382985, -66.4159879, -17.4983832]`

Boundary source counts:

| Admin Level | Count |
| ----------- | ----: |
| 4           | 15    |
| 6           | 54    |

Evaluation sampled against the same scoped boundary used for the build.

---

# 4. Administrative Model

The engine exposes OSM administrative levels:

| Level | Chile Model |
| ----: | ----------- |
| 4     | Region      |
| 6     | Province    |
| 8     | Commune     |

Allowed runtime shapes:

* `[4]`
* `[4,6]`
* `[4,6,8]`
* `[4,8]`
* `[6]`
* `[6,8]`
* `[8]`

`[4,6,8]` is the complete `ok` shape. Sparse valid shapes are treated as `partial`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside ratio: `0.9`
* Expected inside samples: `9,000`
* Expected outside samples: `1,000`
* Boundary: `tmp/cl_country.json`

The test intentionally injects out-of-boundary points to validate boundary rejection, offshore classification, policy containment, and cross-border isolation.

---

# 6. Performance Metrics

| Metric                    | Value        |
| ------------------------- | ------------ |
| Throughput                | `539.430` QPS |
| Total Runtime             | `18.538 sec` |
| Overall Pass Rate         | `100.00%`    |
| Inside Coverage Pass Rate | `100.00%`    |
| Policy Pass Rate          | `100.00%`    |

No inside-boundary lookup failures were observed in this run.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed |
| ------------ | --------- | ---------------- | -----: |
| full_policy  | 100.00%   | 100.00%          | 0      |
| no_hierarchy | 100.00%   | 100.00%          | 0      |
| no_repair    | 100.00%   | 100.00%          | 0      |
| no_nearby    | 100.00%   | 100.00%          | 0      |
| osm_only     | 100.00%   | 100.00%          | 0      |

## 7.1 Layer Effects

| Layer             | Rescued Samples |
| ----------------- | --------------: |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 2               |
| Total vs OSM-only | 2               |

Hierarchy is required by policy because sparse shapes can be structurally completed when parent evidence is present, but this randomized run did not require hierarchy rescue to pass coverage.

---

# 8. Structural Distribution

## 8.1 Lookup Status Counts

| Status  | Count |
| ------- | ----: |
| ok      | 8,997 |
| partial | 5     |
| failed  | 998   |

The `failed` outcomes are expected outside-boundary samples, except for one offshore sample handled by policy.

## 8.2 Shape Distribution

| Shape   | Count |
| ------- | ----: |
| [4,6,8] | 8,996 |
| []      | 999   |
| [4,8]   | 4     |
| [4,6]   | 1     |

## 8.3 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 9,000 |
| admin_tree_id | 12    |
| nearby        | 1     |

## 8.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,001 |
| empty_shape      | 998   |
| offshore         | 1     |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `16`
* Total level-4 hits: `9,001`
* Total level-4 inside hits: `9,000`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside) | Hit Rate (Inside Samples) |
| ------------ | ---: | ---------------------: | ------------: | ------------------------: |
| Región de Magallanes y de la Antártica Chilena | 2,651 | 26.51% | 2,650 | 29.44% |
| Región Aysén del General Carlos Ibáñez del Campo | 1,554 | 15.54% | 1,554 | 17.27% |
| Región de Antofagasta | 1,059 | 10.59% | 1,059 | 11.77% |
| Región de Los Lagos | 686 | 6.86% | 686 | 7.62% |
| Región de Atacama | 663 | 6.63% | 663 | 7.37% |
| Región de Coquimbo | 365 | 3.65% | 365 | 4.06% |
| Región de Tarapacá | 363 | 3.63% | 363 | 4.03% |
| Región de la Araucanía | 310 | 3.10% | 310 | 3.44% |
| Región del Maule | 287 | 2.87% | 287 | 3.19% |
| Región del Biobío | 214 | 2.14% | 214 | 2.38% |
| Región de Los Ríos | 191 | 1.91% | 191 | 2.12% |
| Región de Valparaíso | 143 | 1.43% | 143 | 1.59% |
| Región del Libertador General Bernardo O'Higgins | 137 | 1.37% | 137 | 1.52% |
| Región Metropolitana de Santiago | 130 | 1.30% | 130 | 1.44% |
| Región de Arica y Parinacota | 127 | 1.27% | 127 | 1.41% |
| Región de Ñuble | 121 | 1.21% | 121 | 1.34% |

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-boundary samples:

* No boundary-leak failure was observed.
* No inside-boundary coverage failure was observed.
* Empty-shape outcomes are concentrated in expected outside-boundary samples.
* Nearby fallback was minimal and bounded.

This confirms strict boundary containment within the declared Chile administrative coverage boundary.

---

# 11. Structural Observations

1. Chile has a clear OSM administrative model at levels 4, 6, and 8.
2. The initial plain Natural Earth boundary over-sampled areas without released OSM admin coverage; the tracked CL boundary builder now derives administrative coverage deterministically.
3. Los Lagos coverage depends on level 6 province geometry because the source PBF does not provide the corresponding level 4 regional polygon as releaseable geometry.
4. Geometry coverage is high under the declared scope.
5. No explicit repair layer is needed for this release.
6. Nearby fallback contribution is minimal.

---

# 12. Reproducibility

Dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine release revision: recorded in `dataset_release_manifest.json`
* Cadis version: `0.8.21`
* OSM PBF: `chile-260509.osm.pbf`
* OSM file SHA256: `4de865dc75244dfb476c48821da65d63eed86ae0b7371bd2eb6b497baa58f185`
* OSM replication timestamp UTC: `2026-05-09T20:21:02Z`
* Boundary builder: `scripts/build_cl_boundaries.py`

The staged dataset evaluated was `CL/cl.admin/v1.0.0`.

---

# 13. Conclusion

The `cl.admin v1.0.0` dataset is release-ready under the declared Chile administrative coverage scope.

The staged evaluation demonstrates:

* 100% overall pass rate
* 100% inside coverage pass rate
* 100% policy pass rate
* deterministic level 4/6/8 administrative output
* no required semantic repair layer
* strict boundary isolation under mixed inside/outside stress testing
