# AM_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `am.admin`
Version: `v1.0.0`
Country: `AM`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `am.admin v1.0.0` dataset under Cadis Runtime.

This report describes the source-data scope, administrative model, lookup behavior, deterministic policy effects, boundary isolation, and reproducibility evidence for release review.

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `am.admin` |
| Dataset Version | `v1.0.0` |
| Country | `AM` |
| Country Name | `Armenia` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.64` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_am_boundaries.py` |
| Generated Boundary | `tmp/am_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Union OSM administrative relation areas at levels 4 and 5 whose representative point is covered by the Natural Earth AM boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[43.449190459749325, 38.84249249950064, 46.626040891389955, 41.29964570499377]` |
| OSM Source | `geofabrik:asia/armenia` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Armenia administrative coverage represented by materialized OSM province and community relations. The country envelope is excluded from the initial runtime contract.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_province` | 11 | Province/Yerevan coverage |
| 5 | `admin_community` | 81 | Community/district coverage |
| 6 | `admin_municipality` | 14 | Municipality coverage where available |
| 7 | `admin_locality` | 151 | Locality coverage where available |
| 8 | `admin_district` | 6 | District coverage where available |
| 10 | `admin_detail` | 7 | Detail coverage where available |

Probe evidence also found level 2 as the country-level envelope; it is intentionally excluded.

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:hy`, and `name:ru`, with bounded multilingual aliases for `en`, `hy`, and `ru`.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Dataset path: `AM/am.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `6578.550` QPS |
| Total Runtime | `1.520 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,028 / 5 / 967 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,028 / 5 / 967 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,028 / 5 / 967 / 0 |
| no_nearby | 100.00% | 100.00% | 0 | 0 | 8,995 / 5 / 1,000 / 0 |
| osm_only | 100.00% | 100.00% | 0 | 0 | 8,995 / 5 / 1,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 33 |
| Total vs OSM-only | 33 |

No explicit repair layer is required for `am.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[4,5]` | 8,138 |
| `[]` | 995 |
| `[4,5,7]` | 392 |
| `[4]` | 366 |
| `[4,5,6]` | 92 |
| `[4,5,8]` | 10 |
| `[5,7]` | 5 |
| `[4,5,10]` | 1 |
| `[4,7]` | 1 |

Empty shapes correspond to:

* `empty_shape`: 967
* `offshore`: 28

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 9,000 |
| admin_tree_id | 37 |
| nearby | 5 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,005 |
| empty_shape | 967 |
| offshore | 28 |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `11`
* Total level-4 hits across all samples: `9,000`
* Total level-4 hits across inside samples: `8,995`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Gegharkunik Province | 1,521 | 15.21% | 1,521 | 16.90% |
| Syunik Province | 1,331 | 13.31% | 1,329 | 14.77% |
| Lori Province | 1,108 | 11.08% | 1,108 | 12.31% |
| Shirak Province | 888 | 8.88% | 887 | 9.86% |
| Tavush Province | 836 | 8.36% | 836 | 9.29% |
| Aragatsotn Province | 820 | 8.20% | 820 | 9.11% |
| Vayots Dzor Province | 709 | 7.09% | 708 | 7.87% |
| Kotayk Province | 696 | 6.96% | 696 | 7.73% |
| Ararat Province | 619 | 6.19% | 618 | 6.87% |
| Armavir Province | 395 | 3.95% | 395 | 4.39% |
| Yerevan | 77 | 0.77% | 77 | 0.86% |

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/am_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Armenia dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,5]`, reflecting strong province and community coverage.
3. Levels 6, 7, 8, and 10 provide valid lower-level detail where available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `bcf29d6aeb79c1d7505812aa6349094a24e2e2fa`
- Cadis version:
  `0.8.64`
- Boundary builder: `scripts/build_am_boundaries.py`
- Build boundary: `tmp/am_country.json`
- Evaluation boundary: `tmp/am_country.json`
- Staged dataset: `AM/am.admin/v1.0.0`
- Source OSM SHA256:
  `5254667ba1a1f1161fe9c8b28ce10a7ec33edd28b8ce07c90ab743b98b74a4ff`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `am.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable administrative coverage for Armenia levels 4, 5, 6, 7, 8, and 10.

The dataset is suitable for human quality review.
