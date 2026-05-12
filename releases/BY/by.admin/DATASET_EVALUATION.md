# BY_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `by.admin`
Version: `v1.0.1`
Country: `BY`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `by.admin v1.0.1` under Cadis Runtime after removing detailed locality/admin-unit geometry from the runtime contract.

Belarus level `10` contributed a very large detailed administrative layer, and levels `9` and `11` were sparse/local detail. Version `v1.0.1` keeps country, region, municipality, and locality semantics while excluding those fine-grained layers.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `by.admin` |
| Dataset Version | `v1.0.1` |
| Country | `BY` |
| Country Name | `Belarus` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.160` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_by_boundaries.py` |
| Generated Boundary | `tmp/by_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 2 whose representative point is covered by the Natural Earth BY boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[23.182506720056995, 51.264755016263095, 32.76064845302717, 56.16965743904249]` |
| OSM Source | `geofabrik:europe/belarus` |
| OSM Snapshot Timestamp | `2026-05-11T20:20:52Z` |

The same scoped boundary was used for build and evaluation.

---

# 4. Administrative Model

The reduced Belarus engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 2 | `admin_country` | 1 | Country coverage anchor |
| 4 | `admin_region` | 7 | Regions and Minsk |
| 6 | `admin_municipality` | 128 | Municipality/district coverage |
| 8 | `admin_locality` | 1,287 | Locality-level coverage retained where available |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 9 | 25 | Excluded from `v1.0.1`; sparse local detail |
| 10 | 22,342 | Excluded from `v1.0.1`; detailed layer too large for Cadis semantic resolution |
| 11 | 0 | Excluded from `v1.0.1`; no scoped coverage in the release report |

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Expected inside ratio: `0.9`
* Dataset path: `BY/by.admin/v1.0.1`

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `4345.310` QPS |
| Total Runtime | `2.301 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,022 / 0 / 978 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,022 / 0 / 978 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,022 / 0 / 978 / 0 |
| no_nearby | 100.00% | 100.00% | 0 | 0 | 9,000 / 0 / 1,000 / 0 |
| osm_only | 100.00% | 100.00% | 0 | 0 | 9,000 / 0 / 1,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 22 |
| Total vs OSM-only | 22 |

---

# 9. Structural Distribution

| Shape | Count |
| ----- | ----: |
| `[2,4,6,8]` | 8,919 |
| `[]` | 998 |
| `[2,4,6]` | 51 |
| `[2,4]` | 18 |
| `[2,6,8]` | 13 |
| `[2,4,8]` | 1 |

| Source | Count |
| ------ | ----: |
| polygon | 9,000 |
| admin_tree_id | 10 |
| nearby | 2 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,002 |
| empty_shape | 978 |
| offshore | 20 |

---

# 10. Level-4 Coverage

The run hit all `7` level-4 units, with `8,989` total level-4 hits and `8,987` inside level-4 hits.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/by_country.json` scoped boundary was used for build and evaluation.

---

# 12. Structural Observations

1. The runtime contract now exposes levels 2, 4, 6, and 8.
2. Level 10 was removed because 22,342 detailed features made metadata and hierarchy disproportionate to Cadis semantic needs.
3. Sparse levels 9 and 11 were also removed from the runtime contract.
4. Package size is now `0.9 MB` compressed and `1.7 MB` unpacked, down from the prior `3.6 MB` compressed and `16.1 MB` unpacked release.
5. Evaluation behavior remains stable, with 100% overall, policy, and inside pass rates.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `c139f250231e8d968581a65a6379558c943b5e85`
- Cadis version:
  `0.8.160`
- Runtime compatibility minimum:
  `0.8.35`
- Source OSM SHA256:
  `c049ac0361f674b923aa9c00ce0b8012a93d81bcfdbc936328d2abc88f00fa2c`
- Boundary builder: `scripts/build_by_boundaries.py`
- Build boundary: `tmp/by_country.json`
- Evaluation boundary: `tmp/by_country.json`
- Staged dataset: `BY/by.admin/v1.0.1`

---

# 14. Conclusion

The `by.admin v1.0.1` dataset passes evaluation and is suitable for release. The runtime model keeps useful Belarus administrative semantics while excluding overly detailed locality/admin-unit geometry.
