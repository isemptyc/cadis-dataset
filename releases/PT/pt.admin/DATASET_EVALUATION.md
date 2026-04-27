# PT_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `pt.admin`
Version: `v1.0.0`
Country: `PT`
Scope: full Portugal ISO2 administrative coverage represented by OSM municipality/parish polygons
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `pt.admin v1.0.0` dataset under Cadis Runtime.

The Portugal source extract contains neighboring Spanish administrative relations. The dataset therefore uses a tracked boundary builder to derive a deterministic Portugal administrative coverage boundary before build and evaluation.

This report:

* Describes the configured dataset scope
* Quantifies structural completeness under the runtime policy
* Documents deterministic policy effects
* Validates boundary isolation under stress testing
* Provides reproducible integrity metrics

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value          |
| ----------------------- | -------------- |
| Dataset ID              | `pt.admin`     |
| Dataset Version         | `v1.0.0`       |
| Country                 | `PT`           |
| Country Name            | `Portugal`     |
| Policy Version          | `1.0`          |
| Hierarchy Required      | `True`         |
| Repair Required         | `False`        |
| Runtime Policy Detected | `True`         |
| Cadis Version           | `0.6.3`        |

---

# 3. Dataset Scope

Deterministic selection rule:

* Build the Natural Earth `PT` admin-0 polygon as a reference boundary.
* Scan the Portugal OSM PBF for administrative relation areas at levels `7` and `8`.
* Keep relations whose representative point is covered by the Natural Earth `PT` boundary.
* Union those selected municipality/parish polygons into the release build/evaluation boundary.

Generated scoped boundary:

| Field | Value |
| ----- | ----- |
| Boundary name | `Portugal admin coverage` |
| BBox | `[-31.2688154, 32.4036871, -6.1891593, 42.1543112]` |
| Geometry type | `MultiPolygon` |
| Level-7 source polygons | `307` |
| Level-8 source polygons | `3238` |

Scope note:

* The intended product scope is full Portugal ISO2 administrative coverage represented in the source PBF, including mainland Portugal, Acores, and Madeira where covered by OSM admin polygons.
* The administrative coverage boundary is used for both build scoping and evaluation sampling.
* The boundary avoids false inside samples from broader admin-0/coastal water geometry while preserving the resolvable administrative coverage surface.

---

# 4. Source and Build Summary

| Field | Value |
| ----- | ----- |
| OSM Source | `geofabrik:europe/Portugal` |
| OSM PBF | `portugal-260331.osm.pbf` |
| OSM replication timestamp | `2026-03-31T20:21:06Z` |

Observed build distribution:

| Admin Level | Count |
| ----------- | ----: |
| 4 | 1 |
| 6 | 18 |
| 7 | 307 |
| 8 | 3257 |

Level labels:

* `4`: autonomous region
* `6`: district
* `7`: municipality
* `8`: civil parish

---

# 5. Test Methodology

Sampling:

| Category | Count |
| -------- | ----: |
| Total samples | 10,000 |
| Expected inside points | 9,000 |
| Expected outside points | 1,000 |
| Inside ratio | 0.9 |
| Workers | 32 |

The test intentionally injects 10% out-of-bound points to validate boundary rejection and cross-border isolation.

---

# 6. Performance Metrics

| Metric | Value |
| ------ | ----: |
| Throughput | `878.940` QPS |
| Total runtime | `11.377 sec` |
| Overall pass rate | `100.00%` |
| Policy pass rate | `100.00%` |
| Inside coverage pass rate | `100.00%` |
| Failed samples | `0` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed |
| -------- | --------: | ---------------: | -----: |
| full_policy | 100.00% | 100.00% | 0 |
| no_hierarchy | 100.00% | 100.00% | 0 |
| no_repair | 100.00% | 100.00% | 0 |
| no_nearby | 100.00% | 100.00% | 0 |
| osm_only | 100.00% | 100.00% | 0 |

Layer effect counters:

| Layer | Rescued Samples |
| ----- | --------------: |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 0 |
| Total vs OSM-only | 0 |

Branch-constrained hierarchy supplementation contributed ID-based nodes in 6 samples, but in this run it did not change pass/fail outcome counts.

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[6,7,8]` | 8,684 |
| `[]` | 1,000 |
| `[4,7,8]` | 231 |
| `[7,8]` | 68 |
| `[8]` | 11 |
| `[6,8]` | 4 |
| `[4,7]` | 2 |

Empty shapes correspond to the intentionally sampled outside-boundary points.

## 8.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 9,000 |
| admin_tree_id | 6 |

## 8.3 Source Mix

| Source Mix | Count |
| ---------- | ----: |
| polygon | 8,994 |
| none | 1,000 |
| admin_tree_id\|polygon | 6 |

## 8.4 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,000 |
| empty_shape | 1,000 |

---

# 9. Level-4 Coverage

Only `Acores` appears as a level-4 administrative unit in the sampled run. Mainland Portugal is modeled primarily through districts at level 6.

| Level-4 Unit | Hits | Hit Rate (All Points) | Hits (Inside) | Hit Rate (Inside Points) |
| ------------ | ---: | --------------------: | ------------: | -----------------------: |
| `Açores` | 233 | 2.33% | 233 | 2.59% |

---

# 10. Boundary Isolation Validation

Under stress testing with 1,000 forced out-of-bound samples:

* No policy failure was observed.
* No inside-boundary coverage failure was observed.
* Empty-shape outcomes matched the intentionally outside sample group.
* No evidence was observed that hierarchy supplementation created cross-border escalation.

This confirms strict boundary containment for the staged `pt.admin v1.0.0` dataset under the configured administrative coverage scope.

---

# 11. Structural Observations

1. Portugal requires levels `4/6/7/8` because the mainland and autonomous-region structures differ.
2. Mainland samples mostly resolve as `[6,7,8]`.
3. Acores samples resolve through `[4,7,8]` or `[4,7]`.
4. Madeira is covered through municipality/parish structures without a level-4 polygon in this source build.
5. The repair layer is not required.
6. Hierarchy supplementation is branch-constrained by feature ID/path. Name-based fallback is only temporary and only available for globally unique names when no polygon-derived branch can be established.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit:
  `76ac76a8578573439e6ab5e7bd73fa30e7b5b7d7`
* Cadis version:
  `0.6.3`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 13. Conclusion

The `pt.admin v1.0.0` staged dataset demonstrates:

* Full sampled inside-boundary coverage under the configured Portugal administrative coverage scope
* Strict outside-boundary rejection in the sampled stress test
* No required repair layer
* Stable behavior across mainland, Acores, and Madeira administrative structures

The dataset is ready for release review, subject to the normal publish-phase checks.
