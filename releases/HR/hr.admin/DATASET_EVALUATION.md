# HR_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `hr.admin`
Version: `v1.0.1`
Country: `HR`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `hr.admin v1.0.1` under Cadis Runtime after removing dense settlement-level geometry from the runtime contract.

Version `v1.0.1` excludes OSM admin level `8` settlement polygons. The level-8 data is valid, but it made the Croatia package disproportionate to the source extract and is not required for the administrative runtime contract.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `hr.admin` |
| Dataset Version | `v1.0.1` |
| Country | `HR` |
| Country Name | `Croatia` |
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
| Boundary Builder | `scripts/build_hr_boundaries.py` |
| Generated Boundary | `tmp/hr_country.json` |
| Boundary Source | `Natural Earth admin-0` |
| OSM Source | `geofabrik:europe/croatia` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for build and evaluation.

---

# 4. Administrative Model

The reduced Croatia engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_county` | 21 | Counties and the City of Zagreb |
| 7 | `admin_city_or_municipality` | 555 | Cities and municipalities |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 8 | 6,754 | Excluded from `v1.0.1` for package-size discipline |
| 9 | 260 | Excluded as sparse city district/local board coverage |
| 10 | 218 | Excluded as sparse local board coverage |

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Expected inside ratio: `0.9`
* Dataset path: `HR/hr.admin/v1.0.1`

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `4243.690` QPS |
| Total Runtime | `2.356 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,026 / 0 / 974 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,026 / 0 / 974 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,026 / 0 / 974 / 0 |
| no_nearby | 99.85% | 99.83% | 15 | 15 | 8,985 / 0 / 1,015 / 0 |
| osm_only | 99.85% | 99.83% | 15 | 15 | 8,985 / 0 / 1,015 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 41 |
| Total vs OSM-only | 41 |

---

# 9. Structural Distribution

| Shape | Count |
| ----- | ----: |
| `[4,7]` | 5,774 |
| `[4]` | 3,231 |
| `[]` | 995 |

| Source | Count |
| ------ | ----: |
| polygon | 8,985 |
| admin_tree_id | 21 |
| nearby | 20 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,005 |
| empty_shape | 974 |
| offshore | 21 |

---

# 10. Level-4 Coverage

The run hit all `21` level-4 counties, with `9,005` total level-4 hits.

---

# 11. Boundary Isolation Validation

No boundary-leak failure was observed in mixed inside/outside stress testing. Empty-shape and offshore outcomes dominated expected outside-labeled samples.

---

# 12. Structural Observations

1. Geometry integrity remains high after reducing the runtime contract to levels 4 and 7.
2. The dominant lookup shape is `[4,7]`, reflecting county and city/municipality coverage.
3. Level 8 settlement geometry was excluded because 6,754 settlement polygons made the package disproportionate to the source extract.
4. Package size is now `0.29 MB` compressed and `0.54 MB` unpacked.
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
  `7669481715bd16d1d0ad3989bd8a39b0decbef177ad3b9532116dfc475d42360`
- Boundary builder: `scripts/build_hr_boundaries.py`
- Build boundary: `tmp/hr_country.json`
- Evaluation boundary: `tmp/hr_country.json`
- Staged dataset: `HR/hr.admin/v1.0.1`

---

# 14. Conclusion

The `hr.admin v1.0.1` dataset passes evaluation and is suitable for release.
