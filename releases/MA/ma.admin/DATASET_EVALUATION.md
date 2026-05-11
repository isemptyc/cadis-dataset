# MA_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ma.admin`
Version: `v1.0.0`
Country: `MA`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ma.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect. Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `ma.admin` |
| Dataset Version | `v1.0.0` |
| Country | `MA` |
| Country Name | `Morocco` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.130` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_ma_boundaries.py` |
| Generated Boundary | `tmp/ma_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 2 whose representative point is covered by the Natural Earth MA boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[-17.242556729354973, 21.33546968241329, -1.000448441966006, 35.994075519683825]` |
| OSM Source | `geofabrik:africa/morocco` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Morocco administrative coverage represented by materialized OSM level 2 administrative relations, with lower-level detail where OSM administrative coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 2 | `admin_country` | 1 | Coverage where available |
| 4 | `admin_region` | 11 | Coverage where available |
| 5 | `admin_district` | 74 | Coverage where available |
| 6 | `admin_municipality` | 423 | Coverage where available |
| 7 | `admin_locality` | 683 | Coverage where available |
| 8 | `admin_locality` | 1496 | Coverage where available |
| 9 | `admin_locality` | 21 | Coverage where available |
| 10 | `admin_detail` | 99 | Coverage where available |
| 11 | `admin_unit` | 8 | Coverage where available |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:ma`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Dataset path: `MA/ma.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `3833.900` QPS |
| Total Runtime | `2.608 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,009 / 1 / 990 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,009 / 1 / 990 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,009 / 1 / 990 / 0 |
| no_nearby | 99.97% | 99.97% | 3 | 3 | 8,998 / 1 / 1,001 / 0 |
| osm_only | 99.97% | 99.97% | 3 | 3 | 8,998 / 1 / 1,001 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 11 |
| Total vs OSM-only | 11 |

No explicit repair layer is required for `ma.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[2,4,5,6,7,8]` | 5,569 |
| `[2,5,6,7,8]` | 1,410 |
| `[]` | 996 |
| `[2]` | 847 |
| `[2,4,6,7,8]` | 324 |
| `[2,4,5,7,8]` | 299 |
| `[2,4,5,6,8]` | 210 |
| `[2,4,7,8]` | 202 |
| `[2,4]` | 67 |
| `[2,4,5]` | 28 |
| `[2,4,5,6,7,8,10]` | 15 |
| `[2,4,5,6,8,10]` | 8 |
| `[2,5,6,8]` | 6 |
| `[2,4,5,6,7,8,9,10]` | 4 |
| `[2,4,5,6,8,9,10]` | 3 |
| `[2,5,6]` | 2 |
| `[2,7,8]` | 2 |
| `[2,4,5,6]` | 1 |
| `[2,4,5,6,8,9]` | 1 |
| `[2,4,6,8]` | 1 |
| `[2,6,8]` | 1 |
| `[5,6,7,8]` | 1 |
| `[2,4,5,6,7]` | 1 |
| `[2,4,5,6,7,8,9]` | 1 |
| `[2,5]` | 1 |

Empty shapes correspond to:

* `empty_shape`: 990
* `offshore`: 6

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 8,999 |
| nearby | 5 |
| admin_tree_id | 4 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,004 |
| empty_shape | 990 |
| offshore | 6 |

---

# 10. Level-4 Coverage

No level-4 city hit-rate section is required for this release. The runtime contract is anchored by level 2 administrative coverage and lower-level administrative detail where present in OSM.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ma_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Morocco dataset scope.

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
  `f8f20ade0c11e38ec4654fec34db2efa63babad1`
- Cadis version:
  `0.8.130`
- Boundary builder: `scripts/build_ma_boundaries.py`
- Build boundary: `tmp/ma_country.json`
- Evaluation boundary: `tmp/ma_country.json`
- Staged dataset: `MA/ma.admin/v1.0.0`
- Source OSM SHA256:
  `189742edc02b44ff242c5bb37e1ac095dd48bddc85a3f24e7c3c0098f88cfbc7`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ma.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for Morocco.

The dataset is suitable for autonomous release.
