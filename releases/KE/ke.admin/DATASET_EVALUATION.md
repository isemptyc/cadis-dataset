# KE_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ke.admin`
Version: `v1.0.0`
Country: `KE`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ke.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect. Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `ke.admin` |
| Dataset Version | `v1.0.0` |
| Country | `KE` |
| Country Name | `Kenya` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.121` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_ke_boundaries.py` |
| Generated Boundary | `tmp/ke_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 2 whose representative point is covered by the Natural Earth KE boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[33.91172571857283, -4.897512712481314, 41.90137200762928, 4.617995298649913]` |
| OSM Source | `geofabrik:africa/kenya` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Kenya administrative coverage represented by materialized OSM level 2 administrative relations, with lower-level detail where OSM administrative coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 2 | `admin_country` | 1 | Coverage where available |
| 3 | `admin_region` | 1 | Coverage where available |
| 4 | `admin_region` | 47 | Coverage where available |
| 6 | `admin_municipality` | 291 | Coverage where available |
| 7 | `admin_locality` | 14 | Coverage where available |
| 8 | `admin_locality` | 968 | Coverage where available |
| 9 | `admin_locality` | 29 | Coverage where available |
| 10 | `admin_detail` | 221 | Coverage where available |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:ke`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Dataset path: `KE/ke.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `9918.170` QPS |
| Total Runtime | `1.008 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,007 / 2 / 991 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,007 / 2 / 991 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,007 / 2 / 991 / 0 |
| no_nearby | 100.00% | 100.00% | 0 | 0 | 8,998 / 2 / 1,000 / 0 |
| osm_only | 100.00% | 100.00% | 0 | 0 | 8,998 / 2 / 1,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 9 |
| Total vs OSM-only | 9 |

No explicit repair layer is required for `ke.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[2,4,6]` | 4,114 |
| `[2,4,6,8]` | 3,547 |
| `[2,4]` | 1,061 |
| `[]` | 1,000 |
| `[2]` | 188 |
| `[2,4,6,8,10]` | 65 |
| `[2,3,4,6,8]` | 6 |
| `[2,4,6,7,8,10]` | 3 |
| `[2,6]` | 3 |
| `[2,4,6,8,9,10]` | 3 |
| `[2,6,8]` | 3 |
| `[2,3,4,6,7,10]` | 1 |
| `[4,6]` | 1 |
| `[2,4,6,9]` | 1 |
| `[2,3,4,6,7,9,10]` | 1 |
| `[6,8]` | 1 |
| `[2,4,6,7,8,9,10]` | 1 |
| `[2,3,4,6]` | 1 |

Empty shapes correspond to:

* `empty_shape`: 991
* `offshore`: 9

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 9,000 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,000 |
| empty_shape | 991 |
| offshore | 9 |

---

# 10. Level-4 Coverage

No level-4 city hit-rate section is required for this release. The runtime contract is anchored by level 2 administrative coverage and lower-level administrative detail where present in OSM.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ke_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Kenya dataset scope.

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
  `eb26ac20fcec88e09ae270e776687b81ebf0cc70`
- Cadis version:
  `0.8.121`
- Boundary builder: `scripts/build_ke_boundaries.py`
- Build boundary: `tmp/ke_country.json`
- Evaluation boundary: `tmp/ke_country.json`
- Staged dataset: `KE/ke.admin/v1.0.0`
- Source OSM SHA256:
  `29da78ffd8e697fe3e9a3f103fab17001245edc5361acd3367b661c00f0eade7`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ke.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for Kenya.

The dataset is suitable for autonomous release.
