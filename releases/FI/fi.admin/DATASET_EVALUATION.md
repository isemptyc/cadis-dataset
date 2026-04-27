# FI_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `fi.admin`
Version: `v1.0.0`
Country: `FI`
Scope: full Finland ISO2 administrative coverage represented by OSM region, subregion, and municipality polygons
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `fi.admin v1.0.0` dataset under Cadis Runtime.

The Finland source extract contains neighboring Swedish, Norwegian, and Russian administrative relations in the raw relation graph. The dataset therefore uses a tracked Natural Earth boundary builder to derive deterministic Finland build and evaluation scope.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value      |
| ----------------------- | ---------- |
| Dataset ID              | `fi.admin` |
| Dataset Version         | `v1.0.0`   |
| Country                 | `FI`       |
| Country Name            | `Finland`  |
| Policy Version          | `1.0`      |
| Hierarchy Required      | `True`     |
| Repair Required         | `False`    |
| Runtime Policy Detected | `True`     |
| Cadis Version           | `0.6.5`    |

---

# 3. Dataset Scope

Deterministic selection rule:

* Build the Natural Earth `FI` admin-0 polygon using `scripts/build_fi_boundaries.py`.
* Extract the OSM `FI` admin-2 country relation from the Finland PBF.
* Union OSM administrative relation areas at levels `7` and `8` whose representative point is covered by the OSM `FI` country boundary and whose geometry intersects the Natural Earth `FI` boundary.

Generated scoped boundary:

| Field | Value |
| ----- | ----- |
| BBox | `[20.1635668, 59.472654, 31.5867071, 70.092293]` |
| Geometry type | `Polygon` |
| Level-7 source polygons | `66` |
| Level-8 source polygons | `292` |

Scope note:

* The intended product scope is full Finland ISO2 administrative coverage, including Aland where represented in the source administrative polygons.
* The same scoped boundary is used for build and evaluation.
* The boundary removes neighboring-country relation leakage observed in the raw Finland PBF probe.

---

# 4. Source and Build Summary

| Field | Value |
| ----- | ----- |
| OSM Source | `geofabrik:europe/Finland` |
| OSM PBF | `finland-260426.osm.pbf` |
| OSM replication timestamp | `2026-04-26T20:21:04Z` |
| Engine commit | `fb36a24c8cb2f6309a88abc9b3184cd4017f5d2f` |

Engine structural probe after scoped build:

| Level | Meaning | Count |
| ----- | ------- | ----: |
| `4` | Region | 18 |
| `7` | Subregion | 66 |
| `8` | Municipality | 292 |

No unresolved `is_in` edges remained after scoped projection.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`

The test intentionally injects out-of-country points to validate boundary rejection, offshore classification, nearby behavior, and cross-border isolation.

---

# 6. Performance Metrics

| Metric                    | Value         |
| ------------------------- | ------------- |
| Throughput                | `1348.310` QPS |
| Total Runtime             | `7.417 sec`   |
| Overall Pass Rate         | `100.00%`     |
| Inside Coverage Pass Rate | `100.00%`     |
| Policy Pass Rate          | `100.00%`     |

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| ------------ | --------: | ---------------: | -----: | ------------: |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             |
| no_nearby    | 100.00%   | 100.00%          | 0      | 0             |
| osm_only     | 100.00%   | 100.00%          | 0      | 0             |

Layer contribution:

| Layer             | Rescued Samples |
| ----------------- | --------------: |
| Hierarchy         | 0 |
| Repair            | 0 |
| Nearby            | 19 |
| Total vs OSM-only | 19 |

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[4,7,8]` | 9,000 |
| `[]` | 998 |
| `[4,8]` | 2 |

Empty shapes correspond to expected outside samples and offshore outcomes:

* `empty_shape`: 981
* `offshore`: 17

## 8.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 9,000 |
| admin_tree_id | 2 |
| nearby | 2 |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `18`
* Total level-4 hits: `9,002`
* Total level-4 hits on inside samples: `9,000`

Top level-4 hit rates reflect uniform land-area sampling and Finland's large northern regions.

| Level-4 Unit | Hits | Hit Rate | Inside Hits | Inside Hit Rate |
| ------------ | ---: | -------: | ----------: | --------------: |
| `Lappi` | 2,757 | 27.57% | 2,756 | 30.62% |
| `Pohjois-Pohjanmaa` | 1,123 | 11.23% | 1,123 | 12.48% |
| `Kainuu` | 537 | 5.37% | 537 | 5.97% |
| `Pohjois-Karjala` | 522 | 5.22% | 522 | 5.80% |
| `Pohjois-Savo` | 507 | 5.07% | 507 | 5.63% |
| `Varsinais-Suomi` | 432 | 4.32% | 432 | 4.80% |
| `Keski-Suomi` | 406 | 4.06% | 406 | 4.51% |
| `Pohjanmaa` | 405 | 4.05% | 405 | 4.50% |
| `Etelä-Savo` | 381 | 3.81% | 381 | 4.23% |
| `Etelä-Pohjanmaa` | 335 | 3.35% | 335 | 3.72% |

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed.
* No neighboring-country administrative relation appeared in the scoped build tree.
* Nearby fallback rescued 19 sampled cases compared with OSM-only mode.
* Offshore classification remained bounded at 17 samples.

---

# 11. Structural Observations

1. Finland uses a stable `[4,7,8]` hierarchy: regions, subregions, municipalities.
2. The raw PBF relation graph includes neighboring-country leakage, but scoped projection removes it while preserving coastal municipalities such as Helsinki.
3. The repair layer is not needed for the initial dataset.
4. Hierarchy supplementation was available but did not rescue sampled failures in this run.
5. The dataset reached 100% inside coverage and 100% policy pass rate.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit:
  `fb36a24c8cb2f6309a88abc9b3184cd4017f5d2f`
* Cadis version:
  `0.6.5`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 13. Conclusion

The `fi.admin v1.0.0` staged dataset is promotion-ready from this evaluation pass:

* 10,000/10,000 sampled lookups passed
* 9,000/9,000 inside samples passed coverage checks with full `[4,7,8]` hierarchy
* no policy failures were observed
* scoped boundary isolation removed raw cross-border relation leakage
* no repair layer is required
