# CI_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ci.admin`
Version: `v1.0.0`
Country: `CI`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ci.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect. Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `ci.admin` |
| Dataset Version | `v1.0.0` |
| Country | `CI` |
| Country Name | `Ivory Coast` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.120` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_ci_boundaries.py` |
| Generated Boundary | `tmp/ci_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 2 whose representative point is covered by the Natural Earth CI boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[-8.59867934698707, 4.16412064105843, -2.4950105903562982, 10.738078241679208]` |
| OSM Source | `geofabrik:africa/ivory-coast` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Ivory Coast administrative coverage represented by materialized OSM level 2 administrative relations, with lower-level detail where OSM administrative coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 2 | `admin_country` | 1 | Coverage where available |
| 4 | `admin_region` | 14 | Coverage where available |
| 5 | `admin_district` | 31 | Coverage where available |
| 6 | `admin_municipality` | 9 | Coverage where available |
| 7 | `admin_locality` | 42 | Coverage where available |
| 8 | `admin_locality` | 30 | Coverage where available |
| 9 | `admin_locality` | 26 | Coverage where available |
| 10 | `admin_detail` | 27 | Coverage where available |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:ci`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Dataset path: `CI/ci.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `5758.690` QPS |
| Total Runtime | `1.737 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,017 / 1 / 982 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,017 / 1 / 982 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,017 / 1 / 982 / 0 |
| no_nearby | 100.00% | 100.00% | 0 | 0 | 9,000 / 1 / 999 / 0 |
| osm_only | 100.00% | 100.00% | 0 | 0 | 9,000 / 1 / 999 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 17 |
| Total vs OSM-only | 17 |

No explicit repair layer is required for `ci.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[2,4,5]` | 7,403 |
| `[]` | 997 |
| `[2,4,5,7]` | 455 |
| `[2,4,5,6]` | 435 |
| `[2]` | 309 |
| `[2,4,5,6,7]` | 87 |
| `[2,4,5,7,8]` | 84 |
| `[2,4,5,8]` | 53 |
| `[2,4,7]` | 51 |
| `[2,4,5,6,7,8]` | 36 |
| `[2,4]` | 34 |
| `[2,4,5,6,8]` | 19 |
| `[2,5]` | 10 |
| `[2,4,8]` | 9 |
| `[2,4,8,9]` | 8 |
| `[2,4,5,10]` | 6 |
| `[2,5,6,7]` | 1 |
| `[2,5,7]` | 1 |
| `[2,4,8,10]` | 1 |
| `[5,7]` | 1 |

Empty shapes correspond to:

* `empty_shape`: 982
* `offshore`: 15

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 9,001 |
| admin_tree_id | 6 |
| nearby | 2 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,003 |
| empty_shape | 982 |
| offshore | 15 |

---

# 10. Level-4 Coverage

No level-4 city hit-rate section is required for this release. The runtime contract is anchored by level 2 administrative coverage and lower-level administrative detail where present in OSM.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ci_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Ivory Coast dataset scope.

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
  `375ee942910f1a673700b83e2f8b7892f024c8b5`
- Cadis version:
  `0.8.120`
- Boundary builder: `scripts/build_ci_boundaries.py`
- Build boundary: `tmp/ci_country.json`
- Evaluation boundary: `tmp/ci_country.json`
- Staged dataset: `CI/ci.admin/v1.0.0`
- Source OSM SHA256:
  `86544ddb2861d3de1c8c78e249a6660a3d8acf4ef19c28f0ed5efdd12baca9d7`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ci.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for Ivory Coast.

The dataset is suitable for autonomous release.
