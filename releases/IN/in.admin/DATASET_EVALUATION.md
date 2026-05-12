# IN_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `in.admin`
Version: `v1.0.1`
Country: `IN`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `in.admin v1.0.1` under Cadis Runtime after removing village/locality-level geometry from the runtime contract.

Cadis is a semantic resolver, not a GIS boundary system. For India, OSM admin level `9` introduces a very large and complex village/locality layer whose completeness and real-world alignment are not appropriate assumptions for Cadis runtime semantics. Version `v1.0.1` therefore keeps state, district, and subdistrict semantics while excluding level `9`.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `in.admin` |
| Dataset Version | `v1.0.1` |
| Country | `IN` |
| Country Name | `India` |
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
| Scope Label | `admin-level-4 coverage union` |
| Boundary Builder | `scripts/build_in_boundaries.py` |
| Generated Boundary | `tmp/in_country.json` |
| Boundary Source | `OSM admin-level-4 coverage union after Natural Earth IN admin-0 filtering` |
| Boundary BBox | `[68.1756585, 6.757237, 96.0124397, 35.6728459]` |
| OSM Source | `geofabrik:asia/india` |
| OSM Snapshot Timestamp | `2026-05-09T20:21:02Z` |

The same scoped boundary was used for build and evaluation.

---

# 4. Administrative Model

The reduced India engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_state_union_territory` | 34 | States and union territories |
| 5 | `admin_district` | 752 | Districts |
| 6 | `admin_subdistrict` | 6,266 | Subdistrict-level administrative units |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 9 | 55,000 | Excluded from `v1.0.1`; too large and too locality-specific for Cadis semantic resolution |

---

# 5. Test Methodology

* Total samples: `50,000`
* Sampling mode: mixed inside/outside stress testing
* Expected inside ratio: `0.9`
* Dataset path: `IN/in.admin/v1.0.1`

Sampling is uniform over the declared scope, not population-weighted.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `6357.310` QPS |
| Total Runtime | `7.865 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 43,617 / 1,402 / 4,981 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 43,617 / 1,402 / 4,981 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 43,617 / 1,402 / 4,981 / 0 |
| no_nearby | 100.00% | 100.00% | 1 | 1 | 43,601 / 1,399 / 5,000 / 0 |
| osm_only | 100.00% | 100.00% | 1 | 1 | 43,601 / 1,399 / 5,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 19 |
| Total vs OSM-only | 19 |

---

# 9. Structural Distribution

| Shape | Count |
| ----- | ----: |
| `[4,5,6]` | 43,603 |
| `[]` | 4,995 |
| `[4,5]` | 1,329 |
| `[4]` | 40 |
| `[4,6]` | 33 |

| Source | Count |
| ------ | ----: |
| polygon | 45,000 |
| nearby | 5 |
| admin_tree_id | 1 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 45,005 |
| empty_shape | 4,981 |
| offshore | 14 |

---

# 10. Level-4 Coverage

The run hit all `34` level-4 units, with `45,005` total level-4 hits and `45,000` inside level-4 hits.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-scope samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The evaluation used the same scoped boundary that the build used.

---

# 12. Structural Observations

1. The runtime contract now exposes state/union-territory, district, and subdistrict semantics only.
2. Level 9 was removed because 55,000 village/locality features are too detailed and quality-variable for Cadis semantic resolution.
3. Package size is now `2.7 MB` compressed and `5.4 MB` unpacked, down from the prior `33.3 MB` compressed and `57.4 MB` unpacked release.
4. Evaluation behavior remains stable, with 100% overall, policy, and inside pass rates.
5. Nearby fallback remains minimal and bounded.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `23518e0af4af950f147bdce89b5902b02c2c7411`
- Cadis version:
  `0.8.160`
- Runtime compatibility minimum:
  `0.8.35`
- Source OSM SHA256:
  `8e08957fe6ace40ca08056cc1742d744262b722f542cb09b31e16f8dd7f6f65b`
- Boundary builder: `scripts/build_in_boundaries.py`
- Build boundary: `tmp/in_country.json`
- Evaluation boundary: `tmp/in_country.json`
- Staged dataset: `IN/in.admin/v1.0.1`

---

# 14. Conclusion

The `in.admin v1.0.1` dataset passes evaluation and is suitable for release. The runtime model is better aligned with Cadis' role as a semantic resolver by excluding village/locality-level geometry.
