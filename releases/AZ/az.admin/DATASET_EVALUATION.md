# AZ_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `az.admin`
Version: `v1.0.0`
Country: `AZ`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `az.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect.
Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `az.admin` |
| Dataset Version | `v1.0.0` |
| Country | `AZ` |
| Country Name | `Azerbaijan` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.65` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_az_boundaries.py` |
| Generated Boundary | `tmp/az_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use the OSM administrative relation area at level 2 whose representative point is covered by the Natural Earth AZ boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[44.76570538923978, 38.39501244573249, 51.174555254555536, 41.962341258483825]` |
| OSM Source | `geofabrik:asia/azerbaijan` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Azerbaijan administrative coverage represented by the materialized OSM country relation, with district-level detail where OSM level 5 coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 2 | `admin_country` | 1 | Country coverage fallback |
| 3 | `admin_autonomous_republic` | 1 | Nakhchivan autonomous republic coverage |
| 4 | `admin_region` | 0 | No materialized level-4 units after scoped filtering |
| 5 | `admin_district` | 78 | District/city coverage where available |
| 6 | `admin_municipality` | 1 | Municipality coverage where available |
| 7 | `admin_city_district` | 13 | City district coverage where available |
| 8 | `admin_locality` | 1 | Locality coverage where available |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:az`, and `name:ru`, with bounded multilingual aliases for `en`, `az`, and `ru`.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Dataset path: `AZ/az.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `7842.010` QPS |
| Total Runtime | `1.275 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,010 / 20 / 970 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,010 / 20 / 970 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,010 / 20 / 970 / 0 |
| no_nearby | 100.00% | 100.00% | 0 | 0 | 8,981 / 20 / 999 / 0 |
| osm_only | 100.00% | 100.00% | 0 | 0 | 8,981 / 20 / 999 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 29 |
| Total vs OSM-only | 29 |

No explicit repair layer is required for `az.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[2,5]` | 7,573 |
| `[2,5,7]` | 1,003 |
| `[]` | 996 |
| `[2,3,5]` | 399 |
| `[5]` | 13 |
| `[5,7]` | 7 |
| `[2,5,8]` | 6 |
| `[2]` | 2 |
| `[2,5,6]` | 1 |

Empty shapes correspond to:

* `empty_shape`: 970
* `offshore`: 26

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 9,001 |
| nearby | 3 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,004 |
| empty_shape | 970 |
| offshore | 26 |

---

# 10. Level-4 Coverage

No level-4 city hits were recorded in this report.

Azerbaijan's sampled runtime structure is anchored by the level-2 country fallback and level-5 district coverage. The source PBF includes only sparse scoped level-4 material after filtering, so level 4 is not part of the practical v1.0.0 runtime coverage signal.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/az_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Azerbaijan dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[2,5]`, reflecting country fallback coverage plus district detail.
3. Level 3, 6, 7, and 8 provide valid lower-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `959bfc913c4fa7ebb50cd5de92f5618b2199b5a2`
- Cadis version:
  `0.8.65`
- Boundary builder: `scripts/build_az_boundaries.py`
- Build boundary: `tmp/az_country.json`
- Evaluation boundary: `tmp/az_country.json`
- Staged dataset: `AZ/az.admin/v1.0.0`
- Source OSM SHA256:
  `4b805dfd972cf0d5e9029624455f5eabb2ecb18f1fcfa21923084cfb30353ea0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `az.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for Azerbaijan levels 2, 3, 5, 6, 7, and 8.

The dataset is suitable for autonomous release.
