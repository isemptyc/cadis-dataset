# MU_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `mu.admin`
Version: `v1.0.0`
Country: `MU`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `mu.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect. Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `mu.admin` |
| Dataset Version | `v1.0.0` |
| Country | `MU` |
| Country Name | `Mauritius` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.129` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_mu_boundaries.py` |
| Generated Boundary | `tmp/mu_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 2 whose representative point is covered by the Natural Earth MU boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[56.383609356472654, -20.726604347538274, 63.71381348719999, -10.137244549664844]` |
| OSM Source | `geofabrik:africa/mauritius` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Mauritius administrative coverage represented by materialized OSM level 2 administrative relations, with lower-level detail where OSM administrative coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 2 | `admin_country` | 1 | Coverage where available |
| 3 | `admin_region` | 3 | Coverage where available |
| 4 | `admin_region` | 9 | Coverage where available |
| 8 | `admin_locality` | 164 | Coverage where available |
| 9 | `admin_locality` | 35 | Coverage where available |
| 10 | `admin_detail` | 38 | Coverage where available |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:mu`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Dataset path: `MU/mu.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `19803.860` QPS |
| Total Runtime | `0.505 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,002 / 0 / 998 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,002 / 0 / 998 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,002 / 0 / 998 / 0 |
| no_nearby | 99.37% | 99.30% | 63 | 63 | 8,937 / 0 / 1,063 / 0 |
| osm_only | 99.37% | 99.30% | 63 | 63 | 8,937 / 0 / 1,063 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 65 |
| Total vs OSM-only | 65 |

No explicit repair layer is required for `mu.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[2,3]` | 5,052 |
| `[2]` | 3,128 |
| `[]` | 999 |
| `[2,4,8]` | 660 |
| `[2,4,8,9]` | 51 |
| `[2,3,8]` | 33 |
| `[2,8]` | 28 |
| `[2,4,9]` | 25 |
| `[2,4]` | 15 |
| `[2,4,8,10]` | 7 |
| `[2,9]` | 2 |

Empty shapes correspond to:

* `empty_shape`: 998
* `offshore`: 1

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 8,937 |
| nearby | 64 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,001 |
| empty_shape | 998 |
| offshore | 1 |

---

# 10. Level-4 Coverage

No level-4 city hit-rate section is required for this release. The runtime contract is anchored by level 2 administrative coverage and lower-level administrative detail where present in OSM.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/mu_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Mauritius dataset scope.

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
  `8ecad1f7a771c66a4737bbc8fdcbae31f9ed4291`
- Cadis version:
  `0.8.129`
- Boundary builder: `scripts/build_mu_boundaries.py`
- Build boundary: `tmp/mu_country.json`
- Evaluation boundary: `tmp/mu_country.json`
- Staged dataset: `MU/mu.admin/v1.0.0`
- Source OSM SHA256:
  `c20ae78142c813b2509414d3073ec0a7b017e261932d69f6cf77222c63dd608b`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `mu.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for Mauritius.

The dataset is suitable for autonomous release.
