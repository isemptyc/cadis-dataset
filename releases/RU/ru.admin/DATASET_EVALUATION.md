# RU_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ru.admin`
Version: `v1.0.0`
Country: `RU`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ru.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect. Observed sparse lower-level outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography. It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `ru.admin` |
| Dataset Version | `v1.0.0` |
| Country | `RU` |
| Country Name | `Russia` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.159` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_ru_boundaries.py` |
| Generated Boundary | `tmp/ru_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 4 whose ISO3166-2 tag starts with RU- or whose representative point is covered by the Natural Earth RU boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[-179.998, 41.18755897461323, 179.998, 82.05662304041535]` |
| OSM Source | `geofabrik:europe/russia` |
| OSM Snapshot Timestamp | `2026-05-11T20:20:52Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Russia administrative coverage represented by materialized OSM level 4 administrative relations, with lower-level detail where OSM administrative coverage is available. The ISO3166-2 inclusion rule is necessary because Natural Earth admin-0 RU does not cover all valid RU level-4 representative points, including Saint Petersburg.

---

# 4. Administrative Model

| Level | Runtime Label | Probe Count | Notes |
| ----: | ------------- | ----------: | ----- |
| 4 | `admin_federal_subject` | 85 | Primary parent level in the runtime contract |
| 5 | `admin_federal_city_district` | 43 | Mostly federal-city district structure |
| 6 | `admin_municipality` | 2,291 | Common municipal layer |
| 8 | `admin_settlement` | 11,414 | Common settlement layer |
| 9 | `admin_locality` | 428 | Sparse but materialized |
| 10 | `admin_detail` | 3,926 | Lower-detail administrative layer |

Probe evidence found level 7 and level 11 as sparse; they are intentionally excluded from the initial runtime contract.

The engine uses canonical names from `name:ru`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `en`, `ja`, `ru`, and `uk`.

---

# 5. Test Methodology

* Total samples: `200,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `180,000`
* Outside samples: `20,000`
* Dataset path: `RU/ru.admin/v1.0.0`

The test intentionally injects out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over the scoped boundary area, not population-weighted.

---

# 6. Performance Metrics

| Metric | Value |
| ------ | ----- |
| Throughput | `1424.140` QPS |
| Total Runtime | `140.436 sec` |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |

This run confirms no policy or coverage failures in the sampled corpus.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Status (ok/partial/failed/unknown) |
| -------- | --------- | ---------------- | -----: | ---------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 180365/33/19602/0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 180365/33/19602/0 |
| no_repair | 100.00% | 100.00% | 0 | 180365/33/19602/0 |
| no_nearby | 99.99% | 99.99% | 17 | 179950/33/20017/0 |
| osm_only | 99.99% | 99.99% | 17 | 179950/33/20017/0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------: |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 415 |
| Total vs OSM-only | 415 |

No repair layer is required. Nearby fallback only affected a small number of offshore-adjacent samples in this run.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[4,6]` | 113,625 |
| `[4,6,8]` | 54,079 |
| `[]` | 19,995 |
| `[4]` | 6,395 |
| `[4,5,6]` | 4,300 |
| `[4,5]` | 727 |
| `[4,5,6,8]` | 464 |
| `[4,6,9]` | 297 |
| `[4,5,8]` | 37 |
| `[6]` | 26 |
| `[4,6,10]` | 25 |
| `[4,8]` | 9 |
| `[6,8]` | 7 |
| `[4,5,6,9]` | 6 |
| `[4,6,9,10]` | 3 |
| `[4,6,8,10]` | 3 |
| `[4,6,8,9]` | 2 |

Empty shapes correspond to expected outside or offshore outcomes:

* `empty_shape`: 19,602
* `offshore`: 393

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 179,983 |
| nearby | 22 |

### Source Mix

| Mix | Count |
| --- | ----: |
| polygon | 179,983 |
| none | 19,995 |
| nearby | 22 |

---

# 10. Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 180,005 |
| empty_shape | 19,602 |
| offshore | 393 |

---

# 11. Level-4 Coverage

* Unique level-4 units hit: `85`
* Total level-4 hits: `179,972`
* Total level-4 hits from inside samples: `179,967`

Hit distribution reflects uniform area sampling and is dominated by Russia's largest federal subjects.

## 11.1 Top-10 Level-4 Hit Rates

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ---------------------: | --------------------: | ------------------------: |
| `Республика Саха (Якутия)` | 36,655 | 18.33% | 36,655 | 20.36% |
| `Красноярский край` | 31,208 | 15.60% | 31,207 | 17.34% |
| `Чукотский автономный округ` | 9,458 | 4.73% | 9,457 | 5.25% |
| `Ямало-Ненецкий автономный округ` | 9,458 | 4.73% | 9,458 | 5.25% |
| `Архангельская область` | 7,840 | 3.92% | 7,840 | 4.36% |
| `Хабаровский край` | 6,689 | 3.34% | 6,689 | 3.72% |
| `Иркутская область` | 6,453 | 3.23% | 6,453 | 3.58% |
| `Камчатский край` | 5,183 | 2.59% | 5,181 | 2.88% |
| `Ханты-Мансийский автономный округ — Югра` | 5,076 | 2.54% | 5,076 | 2.82% |
| `Магаданская область` | 4,863 | 2.43% | 4,863 | 2.70% |

---

# 12. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation in this run.

This confirms strict boundary containment within the scoped RU dataset in the sampled corpus.

---

# 13. Structural Observations

1. Geometry integrity is high in the sampled corpus.
2. The level-4 federal subject layer is the stable parent level for v1.0.0.
3. Level 7 and level 11 are too sparse for the initial runtime contract.
4. No explicit repair layer is required by this sampled run.
5. Nearby fallback is minimal and bounded.
6. Dataset achieves full inside-boundary coverage under policy mode in this sampled run.

---

# 14. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `8c733b3`
- Cadis version:
  `v0.8.159`
- Boundary builder:
  `scripts/build_ru_boundaries.py`

---

# 15. Conclusion

The `ru.admin v1.0.0` dataset draft demonstrates:

* High sampled geometry integrity
* Full sampled inside-boundary coverage under policy mode
* Strict sampled boundary containment
* No repair-layer requirement in the current model
* A stable initial administrative contract centered on level 4 federal subjects

