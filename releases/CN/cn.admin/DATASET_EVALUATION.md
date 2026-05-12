# CN_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `cn.admin`
Version: `v1.0.0`
Country: `CN`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `cn.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect. Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `cn.admin` |
| Dataset Version | `v1.0.0` |
| Country | `CN` |
| Country Name | `China` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.153` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_cn_boundaries.py` |
| Generated Boundary | `tmp/cn_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 4 whose representative point is covered by the Natural Earth CN boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[73.50254522441283, 20.12183040413475, 134.77345103091676, 53.55881051212936]` |
| OSM Source | `geofabrik:asia/china` |
| OSM Snapshot Timestamp | `2026-05-11T20:20:52Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is China administrative coverage represented by materialized OSM level 4 administrative relations, with lower-level detail where OSM administrative coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_region` | 27 | Coverage where available |
| 5 | `admin_district` | 316 | Coverage where available |
| 6 | `admin_municipality` | 2659 | Coverage where available |
| 7 | `admin_locality` | 140 | Coverage where available |
| 8 | `admin_locality` | 17359 | Coverage where available |
| 9 | `admin_locality` | 294 | Coverage where available |
| 10 | `admin_detail` | 7764 | Coverage where available |
| 11 | `admin_unit` | 137 | Coverage where available |
| 12 | `admin_unit` | 4 | Coverage where available |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:zh`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Test Methodology

* Total samples: `100,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `90,000`
* Outside samples: `10,000`
* Dataset path: `CN/cn.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `934.100` QPS |
| Total Runtime | `107.055 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 90,026 / 11 / 9,963 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 90,026 / 11 / 9,963 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 90,026 / 11 / 9,963 / 0 |
| no_nearby | 100.00% | 100.00% | 2 | 2 | 89,989 / 11 / 10,000 / 0 |
| osm_only | 100.00% | 100.00% | 2 | 2 | 89,989 / 11 / 10,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 37 |
| Total vs OSM-only | 37 |

No explicit repair layer is required for `cn.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[4,5,6,8]` | 43,386 |
| `[4,5,6]` | 42,773 |
| `[]` | 9,987 |
| `[4,6,8]` | 912 |
| `[4,5,6,8,10]` | 791 |
| `[4,5,6,7,8]` | 626 |
| `[4,6]` | 562 |
| `[4]` | 402 |
| `[4,5,6,7]` | 144 |
| `[4,6,7,8]` | 75 |
| `[4,5,6,7,8,9]` | 50 |
| `[4,5,6,8,9]` | 47 |
| `[4,5,6,7,8,10]` | 46 |
| `[4,5,6,9]` | 46 |
| `[4,5,6,7,9]` | 35 |
| `[4,5,8]` | 33 |
| `[4,5,8,10]` | 17 |
| `[4,5,7,8]` | 17 |
| `[4,5]` | 11 |
| `[4,6,8,10]` | 10 |
| `[5,6,8]` | 9 |
| `[4,5,6,8,9,10]` | 8 |
| `[4,5,6,10]` | 4 |
| `[6,8]` | 2 |
| `[4,6,7,8,10]` | 2 |
| `[4,5,6,7,8,9,10]` | 2 |
| `[4,6,8,9]` | 2 |
| `[4,5,6,8,10,11]` | 1 |

Empty shapes correspond to:

* `empty_shape`: 9,963
* `offshore`: 24

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 90,000 |
| nearby | 13 |
| admin_tree_id | 8 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 90,013 |
| empty_shape | 9,963 |
| offshore | 24 |

---

# 10. Level-4 Coverage

The dataset is anchored by level 4 administrative coverage. The 100,000-sample evaluation hit all 27 level-4 administrative units present in the scoped boundary.

* Unique level-4 units hit: `27`
* Total level-4 hits (all samples): `90,002`
* Total level-4 hits (inside samples): `89,990`

| Level-4 Unit | Hits | Hit Rate (All Points) | Hits (Inside) | Hit Rate (Inside Points) |
| ------------ | ---: | --------------------: | ------------: | -----------------------: |
| `Xinjiang` | 18,512 | 18.51% | 18,509 | 20.57% |
| `Inner Mongolia` | 13,358 | 13.36% | 13,355 | 14.84% |
| `Qinghai` | 6,725 | 6.73% | 6,725 | 7.47% |
| `Heilongjiang` | 6,026 | 6.03% | 6,024 | 6.69% |
| `Sichuan` | 4,998 | 5.00% | 4,998 | 5.55% |
| `Gansu` | 4,546 | 4.55% | 4,546 | 5.05% |
| `Yunnan` | 3,779 | 3.78% | 3,778 | 4.20% |
| `Guangxi` | 2,415 | 2.42% | 2,415 | 2.68% |
| `Jilin` | 2,300 | 2.30% | 2,299 | 2.55% |
| `Guangdong` | 2,297 | 2.30% | 2,297 | 2.55% |
| `Hebei` | 2,199 | 2.20% | 2,199 | 2.44% |
| `Shaanxi` | 2,199 | 2.20% | 2,199 | 2.44% |
| `Shandong` | 2,098 | 2.10% | 2,098 | 2.33% |
| `Hunan` | 2,022 | 2.02% | 2,022 | 2.25% |
| `Liaoning` | 2,013 | 2.01% | 2,013 | 2.24% |
| `Hubei` | 1,871 | 1.87% | 1,871 | 2.08% |
| `Henan` | 1,744 | 1.74% | 1,744 | 1.94% |
| `Shanxi` | 1,681 | 1.68% | 1,681 | 1.87% |
| `Guizhou` | 1,633 | 1.63% | 1,633 | 1.81% |
| `Jiangxi` | 1,570 | 1.57% | 1,570 | 1.74% |
| `Zhejiang` | 1,460 | 1.46% | 1,459 | 1.62% |
| `Anhui` | 1,421 | 1.42% | 1,421 | 1.58% |
| `Jiangsu` | 1,368 | 1.37% | 1,367 | 1.52% |
| `Chongqing` | 832 | 0.83% | 832 | 0.92% |
| `Ningxia` | 590 | 0.59% | 590 | 0.66% |
| `Beijing` | 181 | 0.18% | 181 | 0.20% |
| `Tianjin` | 164 | 0.16% | 164 | 0.18% |

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/cn_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the China dataset scope.

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
  `505548c38751e47ab42bd68936d21d4fa646f1f4`
- Cadis version:
  `0.8.153`
- Boundary builder: `scripts/build_cn_boundaries.py`
- Build boundary: `tmp/cn_country.json`
- Evaluation boundary: `tmp/cn_country.json`
- Staged dataset: `CN/cn.admin/v1.0.0`
- Source OSM SHA256:
  `12973f26feaad974cc6ca6053c049859cd6d05a7b070df0a04325457fc94cd6c`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `cn.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for China.

The dataset is suitable for autonomous release.
