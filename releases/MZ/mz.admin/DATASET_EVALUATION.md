# MZ_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `mz.admin`
Version: `v1.0.0`
Country: `MZ`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `mz.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect. Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `mz.admin` |
| Dataset Version | `v1.0.0` |
| Country | `MZ` |
| Country Name | `Mozambique` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.131` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_mz_boundaries.py` |
| Generated Boundary | `tmp/mz_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 2 whose representative point is covered by the Natural Earth MZ boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[30.21419284681168, -26.91880743464732, 41.05258751943839, -10.327606647473688]` |
| OSM Source | `geofabrik:africa/mozambique` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Mozambique administrative coverage represented by materialized OSM level 2 administrative relations, with lower-level detail where OSM administrative coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 2 | `admin_country` | 1 | Coverage where available |
| 3 | `admin_region` | 3 | Coverage where available |
| 4 | `admin_region` | 11 | Coverage where available |
| 5 | `admin_district` | 161 | Coverage where available |
| 6 | `admin_municipality` | 120 | Coverage where available |
| 8 | `admin_locality` | 99 | Coverage where available |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:mz`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Dataset path: `MZ/mz.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `4017.000` QPS |
| Total Runtime | `2.489 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,007 / 0 / 993 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,007 / 0 / 993 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,007 / 0 / 993 / 0 |
| no_nearby | 99.97% | 99.97% | 3 | 3 | 8,998 / 0 / 1,002 / 0 |
| osm_only | 99.97% | 99.97% | 3 | 3 | 8,998 / 0 / 1,002 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 9 |
| Total vs OSM-only | 9 |

No explicit repair layer is required for `mz.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[2,3,4,5]` | 5,378 |
| `[2,3,4,5,6]` | 2,806 |
| `[]` | 998 |
| `[2]` | 729 |
| `[2,3,4]` | 63 |
| `[2,3,4,5,8]` | 11 |
| `[2,5]` | 5 |
| `[2,3,5]` | 3 |
| `[2,3,5,6]` | 2 |
| `[2,3,4,6]` | 1 |
| `[2,3]` | 1 |
| `[2,4,5]` | 1 |
| `[2,4]` | 1 |
| `[2,3,4,5,6,8]` | 1 |

Empty shapes correspond to:

* `empty_shape`: 993
* `offshore`: 5

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 8,998 |
| nearby | 4 |
| admin_tree_id | 4 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,002 |
| empty_shape | 993 |
| offshore | 5 |

---

# 10. Level-4 Coverage

No level-4 city hit-rate section is required for this release. The runtime contract is anchored by level 2 administrative coverage and lower-level administrative detail where present in OSM.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/mz_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Mozambique dataset scope.

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
  `5452a4bb584fa290a4390c89b73360e74e72e99e`
- Cadis version:
  `0.8.131`
- Boundary builder: `scripts/build_mz_boundaries.py`
- Build boundary: `tmp/mz_country.json`
- Evaluation boundary: `tmp/mz_country.json`
- Staged dataset: `MZ/mz.admin/v1.0.0`
- Source OSM SHA256:
  `8ca61d4c4916b6508c61c188aa7d1aa90ba9d7f292d37025b855b0b8f0677d5a`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `mz.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for Mozambique.

The dataset is suitable for autonomous release.
