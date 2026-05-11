# MW_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `mw.admin`
Version: `v1.0.0`
Country: `MW`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `mw.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect. Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `mw.admin` |
| Dataset Version | `v1.0.0` |
| Country | `MW` |
| Country Name | `Malawi` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.126` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_mw_boundaries.py` |
| Generated Boundary | `tmp/mw_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 2 whose representative point is covered by the Natural Earth MW boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[32.67414524657233, -17.12759973745573, 35.91540381950424, -9.370302589121499]` |
| OSM Source | `geofabrik:africa/malawi` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Malawi administrative coverage represented by materialized OSM level 2 administrative relations, with lower-level detail where OSM administrative coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 2 | `admin_country` | 1 | Coverage where available |
| 3 | `admin_region` | 3 | Coverage where available |
| 4 | `admin_region` | 28 | Coverage where available |
| 5 | `admin_district` | 54 | Coverage where available |
| 10 | `admin_detail` | 1 | Coverage where available |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:mw`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Dataset path: `MW/mw.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `4097.030` QPS |
| Total Runtime | `2.441 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,020 / 0 / 980 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,020 / 0 / 980 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,020 / 0 / 980 / 0 |
| no_nearby | 99.91% | 99.90% | 9 | 9 | 8,991 / 0 / 1,009 / 0 |
| osm_only | 99.91% | 99.90% | 9 | 9 | 8,991 / 0 / 1,009 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 29 |
| Total vs OSM-only | 29 |

No explicit repair layer is required for `mw.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[2,3,4]` | 6,408 |
| `[2]` | 1,515 |
| `[]` | 998 |
| `[2,3,4,5]` | 888 |
| `[2,3]` | 175 |
| `[2,4]` | 6 |
| `[2,3,4,5,10]` | 5 |
| `[2,3,5]` | 4 |
| `[2,3,4,10]` | 1 |

Empty shapes correspond to:

* `empty_shape`: 980
* `offshore`: 18

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 8,991 |
| nearby | 11 |
| admin_tree_id | 5 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,002 |
| empty_shape | 980 |
| offshore | 18 |

---

# 10. Level-4 Coverage

No level-4 city hit-rate section is required for this release. The runtime contract is anchored by level 2 administrative coverage and lower-level administrative detail where present in OSM.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/mw_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Malawi dataset scope.

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
  `ab4860c3aab9b5c91f8022c1e0a87342d6fd1148`
- Cadis version:
  `0.8.126`
- Boundary builder: `scripts/build_mw_boundaries.py`
- Build boundary: `tmp/mw_country.json`
- Evaluation boundary: `tmp/mw_country.json`
- Staged dataset: `MW/mw.admin/v1.0.0`
- Source OSM SHA256:
  `769823386e66098f32c0cf697f509e29d7a7fe711f1deee659f5b174035ea9c5`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `mw.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for Malawi.

The dataset is suitable for autonomous release.
