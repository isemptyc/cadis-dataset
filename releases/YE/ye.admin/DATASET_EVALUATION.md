# YE_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ye.admin`
Version: `v1.0.0`
Country: `YE`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ye.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect. Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `ye.admin` |
| Dataset Version | `v1.0.0` |
| Country | `YE` |
| Country Name | `Yemen` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.97` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_ye_boundaries.py` |
| Generated Boundary | `tmp/ye_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 2 whose representative point is covered by the Natural Earth YE boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[41.61025081556203, 11.910481669191809, 54.73693445552535, 18.997770619041525]` |
| OSM Source | `geofabrik:asia/yemen` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Yemen administrative coverage represented by materialized OSM level 2 administrative relations, with lower-level detail where OSM administrative coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 2 | `admin_country` | 1 | Coverage where available |
| 4 | `admin_region` | 22 | Coverage where available |
| 5 | `admin_district` | 1 | Coverage where available |
| 6 | `admin_municipality` | 334 | Coverage where available |
| 7 | `admin_locality` | 1 | Coverage where available |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:ye`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Dataset path: `YE/ye.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `12465.470` QPS |
| Total Runtime | `0.802 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,000 / 11 / 989 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,000 / 11 / 989 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,000 / 11 / 989 / 0 |
| no_nearby | 100.00% | 100.00% | 0 | 0 | 8,990 / 11 / 999 / 0 |
| osm_only | 100.00% | 100.00% | 0 | 0 | 8,990 / 11 / 999 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 10 |
| Total vs OSM-only | 10 |

No explicit repair layer is required for `ye.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[2,4,6]` | 8,569 |
| `[]` | 997 |
| `[2,4]` | 416 |
| `[6]` | 7 |
| `[2,4,5,6]` | 4 |
| `[4,6]` | 4 |
| `[2,6]` | 3 |

Empty shapes correspond to:

* `empty_shape`: 989
* `offshore`: 8

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 9,001 |
| nearby | 2 |
| admin_tree_id | 1 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,003 |
| empty_shape | 989 |
| offshore | 8 |

---

# 10. Level-4 Coverage

No level-4 city hit-rate section is required for this release. The runtime contract is anchored by level 2 administrative coverage and lower-level administrative detail where present in OSM.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ye_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Yemen dataset scope.

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
  `48cff39390135d684db845a29acb1f11311a4e2a`
- Cadis version:
  `0.8.97`
- Boundary builder: `scripts/build_ye_boundaries.py`
- Build boundary: `tmp/ye_country.json`
- Evaluation boundary: `tmp/ye_country.json`
- Staged dataset: `YE/ye.admin/v1.0.0`
- Source OSM SHA256:
  `31394ce593f268f70243b378e9c63d55f11aa9d3657463304a8e1d3f0f38e0bb`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ye.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for Yemen.

The dataset is suitable for autonomous release.
