# BF_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `bf.admin`
Version: `v1.0.0`
Country: `BF`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `bf.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect. Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `bf.admin` |
| Dataset Version | `v1.0.0` |
| Country | `BF` |
| Country Name | `Burkina Faso` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.102` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_bf_boundaries.py` |
| Generated Boundary | `tmp/bf_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 4 whose representative point is covered by the Natural Earth BF boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[-5.510322746178535, 9.412561965135565, 1.2826760823901127, 15.082004393681034]` |
| OSM Source | `geofabrik:africa/burkina-faso` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Burkina Faso administrative coverage represented by materialized OSM level 4 administrative relations, with lower-level detail where OSM administrative coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_region` | 12 | Coverage where available |
| 5 | `admin_district` | 40 | Coverage where available |
| 6 | `admin_municipality` | 215 | Coverage where available |
| 8 | `admin_locality` | 13 | Coverage where available |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:bf`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Dataset path: `BF/bf.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `5451.340` QPS |
| Total Runtime | `1.834 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,015 / 0 / 985 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,015 / 0 / 985 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,015 / 0 / 985 / 0 |
| no_nearby | 100.00% | 100.00% | 0 | 0 | 9,000 / 0 / 1,000 / 0 |
| osm_only | 100.00% | 100.00% | 0 | 0 | 9,000 / 0 / 1,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 15 |
| Total vs OSM-only | 15 |

No explicit repair layer is required for `bf.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[4,5,6]` | 5,963 |
| `[4,5]` | 3,034 |
| `[]` | 998 |
| `[4,5,6,8]` | 3 |
| `[4,6]` | 1 |
| `[4,5,8]` | 1 |

Empty shapes correspond to:

* `empty_shape`: 985
* `offshore`: 13

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 9,000 |
| admin_tree_id | 14 |
| nearby | 2 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,002 |
| empty_shape | 985 |
| offshore | 13 |

---

# 10. Level-4 Coverage

No level-4 city hit-rate section is required for this release. The runtime contract is anchored by level 4 administrative coverage and lower-level administrative detail where present in OSM.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/bf_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Burkina Faso dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The level 4 administrative fallback guarantees deterministic in-scope behavior.
3. Lower administrative levels provide valid detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges where observed.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `4b854aeb60768d6c1a7cdfbc2c536d5858c727bd`
- Cadis version:
  `0.8.102`
- Boundary builder: `scripts/build_bf_boundaries.py`
- Build boundary: `tmp/bf_country.json`
- Evaluation boundary: `tmp/bf_country.json`
- Staged dataset: `BF/bf.admin/v1.0.0`
- Source OSM SHA256:
  `218056314dc865f082acc7f90408d1598b6187eeb65bf4805381c168a9f1795c`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `bf.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for Burkina Faso.

The dataset is suitable for autonomous release.
