# GA_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ga.admin`
Version: `v1.0.0`
Country: `GA`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ga.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect. Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `ga.admin` |
| Dataset Version | `v1.0.0` |
| Country | `GA` |
| Country Name | `Gabon` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.116` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_ga_boundaries.py` |
| Generated Boundary | `tmp/ga_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 2 whose representative point is covered by the Natural Earth GA boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[8.502226232618938, -4.108048776104373, 14.526980925957341, 2.3169077852048847]` |
| OSM Source | `geofabrik:africa/gabon` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Gabon administrative coverage represented by materialized OSM level 2 administrative relations, with lower-level detail where OSM administrative coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 2 | `admin_country` | 1 | Coverage where available |
| 4 | `admin_region` | 9 | Coverage where available |
| 6 | `admin_municipality` | 49 | Coverage where available |
| 9 | `admin_locality` | 2 | Coverage where available |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:ga`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Dataset path: `GA/ga.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `5418.240` QPS |
| Total Runtime | `1.846 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,017 / 9 / 974 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,017 / 9 / 974 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,017 / 9 / 974 / 0 |
| no_nearby | 99.97% | 99.97% | 3 | 3 | 8,989 / 9 / 1,002 / 0 |
| osm_only | 99.97% | 99.97% | 3 | 3 | 8,989 / 9 / 1,002 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 28 |
| Total vs OSM-only | 28 |

No explicit repair layer is required for `ga.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[2,4,6]` | 8,329 |
| `[]` | 993 |
| `[2,4]` | 654 |
| `[2,4,6,9]` | 11 |
| `[6]` | 9 |
| `[2,6]` | 3 |
| `[2]` | 1 |

Empty shapes correspond to:

* `empty_shape`: 974
* `offshore`: 19

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 8,998 |
| nearby | 9 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,007 |
| empty_shape | 974 |
| offshore | 19 |

---

# 10. Level-4 Coverage

No level-4 city hit-rate section is required for this release. The runtime contract is anchored by level 2 administrative coverage and lower-level administrative detail where present in OSM.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ga_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Gabon dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The level 2 administrative fallback guarantees deterministic in-scope behavior.
3. Lower administrative levels provide valid detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges where observed.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `e46e028453c11d87adb31222cb02fee10c776608`
- Cadis version:
  `0.8.116`
- Boundary builder: `scripts/build_ga_boundaries.py`
- Build boundary: `tmp/ga_country.json`
- Evaluation boundary: `tmp/ga_country.json`
- Staged dataset: `GA/ga.admin/v1.0.0`
- Source OSM SHA256:
  `224f170ff02d8bc31666a6f1b19e2ff4cbbf0c166f5bd268cb0c148e0e7db778`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ga.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for Gabon.

The dataset is suitable for autonomous release.
