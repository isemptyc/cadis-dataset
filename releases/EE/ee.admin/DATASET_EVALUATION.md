# EE_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ee.admin`
Version: `v1.0.1`
Country: `EE`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `ee.admin v1.0.1` under Cadis Runtime after removing dense settlement-level geometry from the runtime contract.

Version `v1.0.1` excludes OSM admin level `9` settlement polygons. The level-9 data is valid, but it made the Estonia package disproportionate to the source extract and is not required for the administrative runtime contract.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `ee.admin` |
| Dataset Version | `v1.0.1` |
| Country | `EE` |
| Country Name | `Estonia` |
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
| Boundary Builder | `scripts/build_ee_boundaries.py` |
| Generated Boundary | `tmp/ee_country.json` |
| Boundary Source | `Natural Earth admin-0` |
| OSM Source | `geofabrik:europe/estonia` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for build and evaluation.

---

# 4. Administrative Model

The reduced Estonia engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 6 | `admin_county` | 15 | Counties |
| 7 | `admin_municipality` | 78 | Municipalities and cities |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 8 | 2 | Excluded as sparse city subarea coverage |
| 9 | 4,709 | Excluded from `v1.0.1` for package-size discipline |
| 10 | 83 | Excluded as sparse neighborhood coverage |

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Expected inside ratio: `0.9`
* Dataset path: `EE/ee.admin/v1.0.1`

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `10930.230` QPS |
| Total Runtime | `0.915 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,028 / 0 / 972 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,028 / 0 / 972 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,028 / 0 / 972 / 0 |
| no_nearby | 99.93% | 99.92% | 7 | 7 | 8,993 / 0 / 1,007 / 0 |
| osm_only | 99.93% | 99.92% | 7 | 7 | 8,993 / 0 / 1,007 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 35 |
| Total vs OSM-only | 35 |

---

# 9. Structural Distribution

| Shape | Count |
| ----- | ----: |
| `[6,7]` | 5,502 |
| `[6]` | 3,499 |
| `[]` | 999 |

| Source | Count |
| ------ | ----: |
| polygon | 8,993 |
| nearby | 8 |
| admin_tree_id | 2 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,001 |
| empty_shape | 972 |
| offshore | 27 |

---

# 10. Level-4 Coverage

No level-4 city hits are reported because the Estonia runtime contract does not expose level 4.

---

# 11. Boundary Isolation Validation

No boundary-leak failure was observed in mixed inside/outside stress testing. Empty-shape and offshore outcomes dominated expected outside-labeled samples.

---

# 12. Structural Observations

1. Geometry integrity remains high after reducing the runtime contract to levels 6 and 7.
2. The dominant lookup shape is `[6,7]`, reflecting county and municipality coverage.
3. Level 9 settlement geometry was excluded because 4,709 settlement polygons made the package disproportionate to the source extract.
4. Package size is now `0.12 MB` compressed and `0.22 MB` unpacked.
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
  `b13ea07caea9b77986d0bb0db28f2f650fe09143a8b4ea721c5139b3998bf95c`
- Boundary builder: `scripts/build_ee_boundaries.py`
- Build boundary: `tmp/ee_country.json`
- Evaluation boundary: `tmp/ee_country.json`
- Staged dataset: `EE/ee.admin/v1.0.1`

---

# 14. Conclusion

The `ee.admin v1.0.1` dataset passes evaluation and is suitable for release.
