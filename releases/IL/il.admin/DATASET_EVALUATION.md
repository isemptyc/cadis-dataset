# IL_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `il.admin`
Version: `v1.0.0`
Country: `IL`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `il.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect. Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `il.admin` |
| Dataset Version | `v1.0.0` |
| Country | `IL` |
| Country Name | `Israel` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.78` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_il_boundaries.py` |
| Generated Boundary | `tmp/il_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 2 whose representative point is covered by the Natural Earth IL boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[34.26992175093548, 29.456870624868344, 35.89257249711742, 33.33356859631801]` |
| OSM Source | `geofabrik:asia/israel-and-palestine` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Israel administrative coverage represented by materialized OSM level 2 administrative relations, with lower-level detail where OSM administrative coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 2 | `admin_country` | 1 | Coverage where available |
| 4 | `admin_region` | 6 | Coverage where available |
| 5 | `admin_district` | 15 | Coverage where available |
| 8 | `admin_locality` | 235 | Coverage where available |
| 9 | `admin_locality` | 10 | Coverage where available |
| 10 | `admin_detail` | 95 | Coverage where available |
| 11 | `admin_unit` | 3 | Coverage where available |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:il`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Dataset path: `IL/il.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `7552.050` QPS |
| Total Runtime | `1.324 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,031 / 14 / 955 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,031 / 14 / 955 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,031 / 14 / 955 / 0 |
| no_nearby | 99.99% | 99.99% | 1 | 1 | 8,985 / 14 / 1,001 / 0 |
| osm_only | 99.99% | 99.99% | 1 | 1 | 8,985 / 14 / 1,001 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 46 |
| Total vs OSM-only | 46 |

No explicit repair layer is required for `il.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[2,4,5,8]` | 7,165 |
| `[2]` | 1,366 |
| `[]` | 996 |
| `[2,4,5]` | 372 |
| `[2,4,5,8,10]` | 25 |
| `[2,4,5,8,9]` | 21 |
| `[2,5,8]` | 13 |
| `[2,4,5,8,9,10]` | 10 |
| `[2,4]` | 9 |
| `[5,8]` | 8 |
| `[2,4,8]` | 6 |
| `[4,5,8]` | 6 |
| `[2,8]` | 2 |
| `[2,5]` | 1 |

Empty shapes correspond to:

* `empty_shape`: 955
* `offshore`: 41

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 8,999 |
| nearby | 5 |
| admin_tree_id | 1 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,004 |
| empty_shape | 955 |
| offshore | 41 |

---

# 10. Level-4 Coverage

No level-4 city hit-rate section is required for this release. The runtime contract is anchored by level 2 administrative coverage and lower-level administrative detail where present in OSM.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/il_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Israel dataset scope.

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
  `9b088b23ede60614b5ea4cb7cda8ab1c0f798562`
- Cadis version:
  `0.8.78`
- Boundary builder: `scripts/build_il_boundaries.py`
- Build boundary: `tmp/il_country.json`
- Evaluation boundary: `tmp/il_country.json`
- Staged dataset: `IL/il.admin/v1.0.0`
- Source OSM SHA256:
  `5a2cdd4a9b9a22ad947e70810466319e4c5f730612ba889eafa77b51d4f109a2`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `il.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for Israel.

The dataset is suitable for autonomous release.
