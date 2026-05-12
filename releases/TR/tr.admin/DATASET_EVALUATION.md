# TR_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `tr.admin`
Version: `v1.0.1`
Country: `TR`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `tr.admin v1.0.1` under Cadis Runtime after removing neighborhood/local-unit geometry from the runtime contract.

Turkey level `8` is a meaningful local layer, but it is closer to arrondissement/neighborhood-style detail than to Cadis' core country administrative semantics. Version `v1.0.1` keeps province and district resolution while excluding level `8`.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `tr.admin` |
| Dataset Version | `v1.0.1` |
| Country | `TR` |
| Country Name | `Turkey` |
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
| Scope Label | `full ISO2 country administrative land coverage` |
| Boundary Builder | `scripts/build_tr_boundaries.py` |
| Build Boundary | `tmp/tr_v101/tr_country.json` |
| Evaluation Boundary | `tmp/tr_v101/tr_admin_coverage_boundary.json` |
| Boundary Source | `Natural Earth admin-0 plus scoped OSM levels 4 and 6 for evaluation coverage` |
| OSM Source | `geofabrik:asia/turkey` |
| OSM Snapshot Timestamp | `2026-04-26T20:21:04Z` |

The evaluation boundary is the union of scoped OSM administrative polygons at levels 4 and 6, buffered inward by a tiny deterministic tolerance to avoid coastal and precision slivers.

---

# 4. Administrative Model

The reduced Turkey engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_province` | 78 | Provinces |
| 6 | `admin_district` | 970 | Districts |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 8 | 13,329 | Excluded from `v1.0.1`; local-unit/neighborhood detail is outside the Cadis semantic contract |
| 5, 7, 9, 10 | sparse | Excluded from the runtime contract |

---

# 5. Test Methodology

* Total samples: `50,000`
* Sampling mode: mixed inside/outside stress testing
* Expected inside ratio: `0.9`
* Dataset path: `TR/tr.admin/v1.0.1`

Sampling is uniform over the administrative coverage boundary, not population-weighted.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `9077.240` QPS |
| Total Runtime | `5.508 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 39,560 / 5,497 / 4,943 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 39,560 / 5,497 / 4,943 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 39,560 / 5,497 / 4,943 / 0 |
| no_nearby | 100.00% | 100.00% | 0 | 0 | 39,506 / 5,494 / 5,000 / 0 |
| osm_only | 100.00% | 100.00% | 0 | 0 | 39,506 / 5,494 / 5,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 57 |
| Total vs OSM-only | 57 |

---

# 9. Structural Distribution

| Shape | Count |
| ----- | ----: |
| `[4,6]` | 39,508 |
| `[]` | 4,995 |
| `[4]` | 3,273 |
| `[6]` | 2,224 |

| Source | Count |
| ------ | ----: |
| polygon | 45,000 |
| admin_tree_id | 69 |
| nearby | 5 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 45,005 |
| empty_shape | 4,943 |
| offshore | 52 |

---

# 10. Level-4 Coverage

The run hit all `78` level-4 provinces, with `42,781` total level-4 hits and `42,776` inside level-4 hits.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Expected outside samples resolved to empty-shape or offshore outcomes.
* Nearby fallback did not create cross-border escalation.
* The reduced evaluation boundary matches the `v1.0.1` levels 4 and 6 runtime contract.

---

# 12. Structural Observations

1. The runtime contract now exposes province and district semantics only.
2. Level 8 was removed because 13,329 local-unit/neighborhood polygons are too fine-grained for Cadis semantic resolution.
3. Package size is now `0.4 MB` compressed and `0.8 MB` unpacked, down from the prior `11.7 MB` compressed and `17.4 MB` unpacked release.
4. Evaluation behavior remains stable, with 100% overall, policy, and inside pass rates.
5. Nearby fallback remains minimal and bounded.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `78b7894d1a8b40ba0aba480bc9a2cfa1e442c192`
- Cadis version:
  `0.8.160`
- Runtime compatibility minimum:
  `0.8.35`
- Source OSM SHA256:
  `977e1b9e6991e4d104acb65627cacf1cb1171a28122b532f93a584473ebc3c46`
- Boundary builder: `scripts/build_tr_boundaries.py`
- Build boundary: `tmp/tr_v101/tr_country.json`
- Evaluation boundary: `tmp/tr_v101/tr_admin_coverage_boundary.json`
- Staged dataset: `TR/tr.admin/v1.0.1`

---

# 14. Conclusion

The `tr.admin v1.0.1` dataset passes evaluation and is suitable for release. The runtime model is better aligned with Cadis' role as a semantic resolver by excluding local-unit/neighborhood detail.
