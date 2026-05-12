# AU_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `au.admin`
Version: `v1.0.1`
Country: `AU`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `au.admin v1.0.1` under Cadis Runtime after removing the level 9 suburb/locality layer from the runtime contract.

Cadis is a semantic resolver, not a GIS boundary system. Version `v1.0.1` keeps Australia states/territories, local government areas, and sparse district-like administrative structure at levels 4, 6, and 7, while excluding level 9 suburb/locality geometry.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `au.admin` |
| Dataset Version | `v1.0.1` |
| Country | `AU` |
| Country Name | `Australia` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.160` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |

---

# 3. Dataset Scope

The Australia release uses the tracked scoped-boundary builder:

* Boundary builder: `scripts/build_au_boundaries.py`
* Boundary source: Natural Earth admin-0 AU polygon plus OSM level-4 administrative relation geometry
* Deterministic selection rule: union OSM administrative relation areas at level 4 whose representative point is covered by the Natural Earth AU boundary
* Build boundary: `tmp/au_country.json`
* Evaluation boundary: `tmp/au_country.json`
* OSM source: `geofabrik:oceania/Australia`
* OSM snapshot timestamp: `2026-03-31T20:21:06Z`

Included level-4 units:

* `Australian Capital Territory`
* `Jervis Bay Territory`
* `New South Wales`
* `Northern Territory`
* `Queensland`
* `South Australia`
* `Tasmania`
* `Victoria`
* `Western Australia`

Major external territory components recorded as excluded or not represented as first-class level-4 units in this scope:

* `Cocos (Keeling) Islands`
* `Christmas Island`
* `Heard Island and McDonald Islands`
* `Norfolk Island`

---

# 4. Administrative Model

The reduced Australia engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_state_territory` | 9 | State and territory anchors |
| 6 | `admin_local_government_area` | 599 | Local government area coverage |
| 7 | `admin_district` | 23 | Sparse district-like administrative coverage |

The previous release also included level 9 suburbs/localities:

| Level | Previous Count | Decision |
| ----: | -------------: | -------- |
| 9 | 15,641 | Excluded from `v1.0.1`; locality layer too fine-grained for Cadis semantic resolution and disproportionate to package size |

The engine uses canonical names from `name:en`, `name`, and `official_name`, with bounded multilingual aliases for `en`.

---

# 5. Evaluation Methodology

* Total samples: `100,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `90,000`
* Outside samples: `10,000`
* Lookup mode: Cadis runtime
* Evaluation workers: `1`
* Dataset path: `AU/au.admin/v1.0.1`

---

# 6. Evaluation Summary

| Metric | Value |
| ------ | ----: |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| HTTP 200 Responses | `100,000` |
| Throughput | `7223.340` QPS |
| Total Runtime | `13.844 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (ok/partial/failed/unknown) |
| -------- | --------: | ---------------: | -----: | ------------: | ----------------------------------- |
| `full_policy` | 100.00% | 100.00% | 0 | 0 | 87,580 / 2,441 / 9,979 / 0 |
| `no_hierarchy` | 100.00% | 100.00% | 0 | 0 | 87,580 / 2,441 / 9,979 / 0 |
| `no_repair` | 100.00% | 100.00% | 0 | 0 | 87,580 / 2,441 / 9,979 / 0 |
| `no_nearby` | 99.97% | 99.96% | 34 | 34 | 87,553 / 2,413 / 10,034 / 0 |
| `osm_only` | 99.97% | 99.96% | 34 | 34 | 87,553 / 2,413 / 10,034 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------: |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 55 |
| Total vs OSM-only | 55 |

No explicit repair layer is required for `au.admin v1.0.1`.

---

# 9. Structural Distribution

| Shape | Count |
| ----- | ----: |
| `[4,6]` | 87,533 |
| `[]` | 9,997 |
| `[4]` | 2,428 |
| `[4,7]` | 29 |
| `[4,6,7]` | 13 |

| Source | Count |
| ------ | ----: |
| polygon | 89,966 |
| nearby | 37 |
| admin_tree_id | 24 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 90,003 |
| empty_shape | 9,979 |
| offshore | 18 |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `8`
* Total level-4 hits (all points): `90,003`
* Total level-4 hits (inside points): `90,000`

| Level-4 Unit | Hits | Hit Rate (All Points) | Hits (Inside) | Hit Rate (Inside Points) |
| ------------ | ---: | --------------------: | ------------: | -----------------------: |
| `Western Australia` | 29,027 | 29.03% | 29,027 | 32.25% |
| `Queensland` | 19,893 | 19.89% | 19,893 | 22.10% |
| `Northern Territory` | 14,977 | 14.98% | 14,976 | 16.64% |
| `South Australia` | 12,182 | 12.18% | 12,181 | 13.53% |
| `New South Wales` | 9,546 | 9.55% | 9,545 | 10.61% |
| `Victoria` | 2,993 | 2.99% | 2,993 | 3.33% |
| `Tasmania` | 1,355 | 1.35% | 1,355 | 1.51% |
| `Australian Capital Territory` | 30 | 0.03% | 30 | 0.03% |

The sampled run did not hit `Jervis Bay Territory`, which is expected for uniform land-area sampling because it is very small.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed.
* Empty-shape and offshore outcomes accounted for expected outside samples.
* No evidence was observed that hierarchy or nearby layers created cross-boundary escalation.
* The same `tmp/au_country.json` scoped boundary was used for build and evaluation.

---

# 12. Structural Observations

1. The runtime contract now exposes levels 4, 6, and 7 only.
2. Australia remains anchored by level 4 state and territory administrative coverage.
3. Level 6 local government areas provide the main semantic subnational layer.
4. Level 7 remains because it is small and represents sparse administrative district-like coverage.
5. Level 9 suburb/locality geometry was removed because it is too fine-grained for Cadis semantic resolution and made package size disproportionate.
6. Package size is now `0.4 MB` compressed and `0.7 MB` unpacked, down from the prior `23.7 MB` compressed and `30.2 MB` unpacked release.
7. Evaluation behavior remains stable, with 100% overall, policy, and inside pass rates.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `b4a7a36cab502c307d8be05f3f7b256a7c38ac97`
- Cadis version:
  `0.8.160`
- Runtime compatibility minimum:
  `0.8.35`
- Boundary builder: `scripts/build_au_boundaries.py`
- Build boundary: `tmp/au_country.json`
- Evaluation boundary: `tmp/au_country.json`
- Staged dataset: `AU/au.admin/v1.0.1`
- Source OSM manifest SHA256:
  `359d2933335054174d01d56d69c2ccfdb6ab4f86527dd991fca3c14b51d2ba50`

---

# 14. Conclusion

The `au.admin v1.0.1` dataset passes evaluation and is suitable for release. The runtime model preserves Australia semantic administrative anchors while excluding overly detailed suburb/locality geometry.
