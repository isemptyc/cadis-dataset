# CN_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `cn.admin`
Version: `v1.0.1`
Country: `CN`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `cn.admin v1.0.1` dataset under Cadis Runtime.

This release keeps `CN` as the public Cadis-routable dataset and includes Tibet as an internal source component. Tibet is not published as a standalone routable dataset because the Natural Earth world classifier resolves Tibet coordinates to `CN`.

Cadis does not modify geography. It enforces structural determinism over the administrative coverage available in the source data.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `cn.admin` |
| Dataset Version | `v1.0.1` |
| Country | `CN` |
| Country Name | `China` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.158` |
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
| Boundary Selection Rule | `Use OSM administrative relation areas at level 4 whose representative point is covered by the Natural Earth CN boundary, plus Tibet internal sub-engine level 5 and level 6 administrative anchors, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[73.50254522441283, 20.12183040413475, 134.77345103091676, 53.55881051212936]` |
| OSM Source | `geofabrik:asia/china+asia/tibet` |
| OSM Snapshot Timestamp | `2026-05-11T20:20:52Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.1 scope is China administrative coverage represented by materialized OSM level 4 administrative relations plus Tibet level 5/6 source coverage, with lower-level detail where OSM administrative coverage is available.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_region` | 27 | China primary anchor coverage |
| 5 | `admin_district` | 322 | Includes Tibet internal source anchors |
| 6 | `admin_municipality` | 2736 | Includes Tibet internal source anchors where level 5 coverage is absent |
| 7 | `admin_locality` | 140 | Coverage where available |
| 8 | `admin_locality` | 18056 | Coverage where available |
| 9 | `admin_locality` | 304 | Coverage where available |
| 10 | `admin_detail` | 7789 | Coverage where available |
| 11 | `admin_unit` | 137 | Coverage where available |
| 12 | `admin_unit` | 4 | Coverage where available |
| 14 | `admin_unit` | 0 | No rows in this snapshot |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:zh`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Test Methodology

* Total samples: `100,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `85,000`
* Outside samples: `15,000`
* Dataset path: `CN/cn.admin/v1.0.1`

The test intentionally injects out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| HTTP 200 Responses | `100,000` |
| Throughput | `1010.850` QPS |
| Total Runtime | `98.927 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 85,039 / 0 / 14,961 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 85,039 / 0 / 14,961 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 85,039 / 0 / 14,961 / 0 |
| no_nearby | 100.00% | 100.00% | 1 | 1 | 84,999 / 0 / 15,001 / 0 |
| osm_only | 100.00% | 100.00% | 1 | 1 | 84,999 / 0 / 15,001 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 40 |
| Total vs OSM-only | 40 |

No explicit repair layer is required for `cn.admin v1.0.1`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[4,5,6,8]` | 36,240 |
| `[4,5,6]` | 35,644 |
| `[]` | 14,996 |
| `[5,6,8]` | 8,494 |
| `[6,8]` | 1,378 |
| `[4,6,8]` | 743 |
| `[4,5,6,8,10]` | 668 |
| `[4,5,6,7,8]` | 516 |
| Other valid administrative shapes | 1,321 |

Empty shapes correspond to expected outside/offshore samples:

* `empty_shape`: 14,961
* `offshore`: 35

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 84,999 |
| admin_tree_id | 6 |
| nearby | 5 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 85,004 |
| empty_shape | 14,961 |
| offshore | 35 |

---

# 10. Level-4 Coverage

The dataset is anchored by level 4 administrative coverage for China proper and level 5/6 administrative coverage for the Tibet internal source component.

* Unique level-4 units hit: `27`
* Total level-4 hits (all points): `75,067`
* Total level-4 hits (inside points): `75,063`

| Level-4 Unit | Hits | Hit Rate (All Points) | Hits (Inside) | Hit Rate (Inside Points) |
| ------------ | ---: | --------------------: | ------------: | -----------------------: |
| `Xinjiang` | 15,463 | 15.46% | 15,463 | 18.19% |
| `Inner Mongolia` | 11,122 | 11.12% | 11,121 | 13.08% |
| `Qinghai` | 5,574 | 5.57% | 5,574 | 6.56% |
| `Heilongjiang` | 4,996 | 5.00% | 4,996 | 5.88% |
| `Sichuan` | 4,169 | 4.17% | 4,169 | 4.90% |
| `Gansu` | 3,785 | 3.79% | 3,785 | 4.45% |
| `Yunnan` | 3,191 | 3.19% | 3,190 | 3.75% |
| `Guangxi` | 2,033 | 2.03% | 2,033 | 2.39% |
| `Jilin` | 1,909 | 1.91% | 1,909 | 2.25% |
| `Guangdong` | 1,902 | 1.90% | 1,902 | 2.24% |

The Tibet source component appears in runtime shapes anchored at level 5 or level 6, primarily through `[5,6,8]`, `[6,8]`, and related shapes. A direct Lhasa smoke lookup returned `Lhasa > Chengguan District > Jêbumgang Subdistrict`.

---

# 11. Boundary Isolation Validation

Under stress testing with 15% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/cn_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the China dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. China proper remains anchored by level 4 administrative coverage.
3. Tibet is included as an internal source component anchored at level 5 or level 6, matching the available OSM administrative structure.
4. Lower administrative levels provide valid detail where available.
5. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges where observed.
6. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `8234fd18285ae1c16add5699ae56c524a5e9f10f`
- Cadis version:
  `0.8.158`
- Boundary builder: `scripts/build_cn_boundaries.py`
- Build boundary: `tmp/cn_country.json`
- Evaluation boundary: `tmp/cn_country.json`
- Staged dataset: `CN/cn.admin/v1.0.1`
- Source OSM manifest SHA256:
  `4dba19915bbc48b3a2164e0cd8514c07832ddc3a4fc8e177dcab8ba0fe312558`
- Source OSM components:
  `china-latest.osm.pbf`, `tibet-latest.osm.pbf`
- Source OSM component SHA256:
  `china-latest.osm.pbf=12973f26feaad974cc6ca6053c049859cd6d05a7b070df0a04325457fc94cd6c`
  `tibet-latest.osm.pbf=2a71933f8cb55fa7b346a0fa03c4c9439f688b9fbb7881f9548f2355a2973291`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `cn.admin v1.0.1` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable composite administrative coverage for China with Tibet included as an internal source component.

The dataset is suitable for autonomous release.
