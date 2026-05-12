# SK_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `sk.admin`
Version: `v1.0.1`
Country: `SK`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `sk.admin v1.0.1` under Cadis Runtime after removing dense locality/detail geometry from the runtime contract.

Version `v1.0.1` excludes OSM admin level `10` locality/detail polygons. The level-10 data is valid, but it made the Slovakia package disproportionate to the source extract and is not required for the administrative runtime contract. Level `9` borough coverage is retained because it is small and semantically useful for Bratislava and Košice.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `sk.admin` |
| Dataset Version | `v1.0.1` |
| Country | `SK` |
| Country Name | `Slovakia` |
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
| Scope Label | `full ISO2 country` |
| Boundary Builder | `scripts/build_sk_boundaries.py` |
| Generated Boundary | `tmp/sk_country.json` |
| Boundary Source | `Natural Earth admin-0` |
| OSM Source | `geofabrik:europe/slovakia` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for build and evaluation.

---

# 4. Administrative Model

The reduced Slovakia engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_region` | 8 | Regions |
| 6 | `admin_district` | 79 | Districts |
| 8 | `admin_municipality` | 2,856 | Municipalities and cities |
| 9 | `admin_borough` | 39 | Bratislava and Košice borough-level areas |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 3 | 4 | Excluded as broad statistical coverage |
| 5 | 2 | Excluded as sparse Bratislava/Košice city envelopes |
| 7 | 0 | Excluded because no scoped features were found |
| 10 | 3,517 | Excluded from `v1.0.1` for package-size discipline |

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Expected inside ratio: `0.9`
* Dataset path: `SK/sk.admin/v1.0.1`

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `9553.430` QPS |
| Total Runtime | `1.047 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,032 / 6 / 962 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,032 / 6 / 962 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,032 / 6 / 962 / 0 |
| no_nearby | 99.48% | 99.42% | 52 | 52 | 8,942 / 6 / 1,052 / 0 |
| osm_only | 99.48% | 99.42% | 52 | 52 | 8,942 / 6 / 1,052 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 90 |
| Total vs OSM-only | 90 |

---

# 9. Structural Distribution

| Shape | Count |
| ----- | ----: |
| `[4,6,8]` | 8,828 |
| `[]` | 998 |
| `[4,6,9]` | 122 |
| `[4,6]` | 25 |
| `[4,8]` | 15 |
| `[8]` | 5 |
| `[4]` | 4 |
| `[6,8]` | 1 |

| Source | Count |
| ------ | ----: |
| polygon | 8,948 |
| nearby | 54 |
| admin_tree_id | 14 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,002 |
| empty_shape | 962 |
| offshore | 36 |

---

# 10. Level-4 Coverage

The run hit all `8` level-4 regions, with `8,996` total level-4 hits.

---

# 11. Boundary Isolation Validation

No boundary-leak failure was observed in mixed inside/outside stress testing. Empty-shape and offshore outcomes dominated expected outside-labeled samples.

---

# 12. Structural Observations

1. Geometry integrity remains high after reducing the runtime contract to levels 4, 6, 8, and 9.
2. The dominant lookup shape is `[4,6,8]`, reflecting region, district, and municipality coverage.
3. Level 10 locality/detail geometry was excluded because 3,517 polygons made the package disproportionate to the source extract.
4. Level 9 borough geometry is retained because it contains only 39 scoped nodes and provides useful Bratislava/Košice detail.
5. Package size is now `4.80 MB` compressed and `6.05 MB` unpacked.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `598b60ffe8553d6d0d658aa516bba7b02ca82e6c`
- Cadis version:
  `0.8.160`
- Runtime compatibility minimum:
  `0.8.35`
- Source OSM SHA256:
  `4caff7a85d0b21e364e74a4404dcccdc53866a88842e78cd4c80b4eead44860e`
- Boundary builder: `scripts/build_sk_boundaries.py`
- Build boundary: `tmp/sk_country.json`
- Evaluation boundary: `tmp/sk_country.json`
- Staged dataset: `SK/sk.admin/v1.0.1`

---

# 14. Conclusion

The `sk.admin v1.0.1` dataset passes evaluation and is suitable for release.
