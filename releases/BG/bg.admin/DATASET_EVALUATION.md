# BG_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `bg.admin`
Version: `v1.0.1`
Country: `BG`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `bg.admin v1.0.1` under Cadis Runtime after removing dense settlement-level geometry from the runtime contract.

Version `v1.0.1` excludes OSM admin level `8` settlement polygons. The level-8 data is valid, but it made the Bulgaria package disproportionate to the source extract and is not required for the administrative runtime contract.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `bg.admin` |
| Dataset Version | `v1.0.1` |
| Country | `BG` |
| Country Name | `Bulgaria` |
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
| Boundary Builder | `scripts/build_bg_boundaries.py` |
| Generated Boundary | `tmp/bg_country.json` |
| Boundary Source | `Natural Earth admin-0` |
| OSM Source | `geofabrik:europe/bulgaria` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for build and evaluation.

---

# 4. Administrative Model

The reduced Bulgaria engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_province` | 28 | Provinces |
| 5 | `admin_municipality` | 265 | Municipalities |
| 6 | `admin_city_district` | 35 | City districts in major municipalities |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 8 | 1,983 | Excluded from `v1.0.1` for package-size discipline |
| 9 | 137 | Excluded as sparse neighborhood/microdistrict coverage |
| 10 | 7 | Excluded as sparse microdistrict coverage |

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Expected inside ratio: `0.9`
* Dataset path: `BG/bg.admin/v1.0.1`

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `13632.970` QPS |
| Total Runtime | `0.734 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,031 / 0 / 969 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,031 / 0 / 969 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,031 / 0 / 969 / 0 |
| no_nearby | 98.22% | 98.02% | 178 | 178 | 8,822 / 0 / 1,178 / 0 |
| osm_only | 98.22% | 98.02% | 178 | 178 | 8,822 / 0 / 1,178 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 209 |
| Total vs OSM-only | 209 |

---

# 9. Structural Distribution

| Shape | Count |
| ----- | ----: |
| `[4,5]` | 8,798 |
| `[]` | 1,051 |
| `[4,5,6]` | 113 |
| `[4]` | 38 |

| Source | Count |
| ------ | ----: |
| polygon | 8,822 |
| nearby | 127 |
| admin_tree_id | 20 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 8,949 |
| empty_shape | 969 |
| offshore | 82 |

---

# 10. Level-4 Coverage

The run hit all `28` level-4 provinces, with `8,949` total level-4 hits.

---

# 11. Boundary Isolation Validation

No boundary-leak failure was observed in mixed inside/outside stress testing. Empty-shape and offshore outcomes dominated expected outside-labeled samples.

---

# 12. Structural Observations

1. Geometry integrity remains high after reducing the runtime contract to levels 4, 5, and 6.
2. The dominant lookup shape is `[4,5]`, reflecting province and municipality coverage.
3. Level 8 settlement geometry was excluded because 1,983 settlement polygons made the package disproportionate to the source extract.
4. Package size is now `0.09 MB` compressed and `0.23 MB` unpacked.
5. Dataset achieves full inside-boundary coverage under full policy mode.

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
  `754c24a83c44c3418f973d778788cf22b070a6b4a4a89ac5aa16fe2823558bb8`
- Boundary builder: `scripts/build_bg_boundaries.py`
- Build boundary: `tmp/bg_country.json`
- Evaluation boundary: `tmp/bg_country.json`
- Staged dataset: `BG/bg.admin/v1.0.1`

---

# 14. Conclusion

The `bg.admin v1.0.1` dataset passes evaluation and is suitable for release.
