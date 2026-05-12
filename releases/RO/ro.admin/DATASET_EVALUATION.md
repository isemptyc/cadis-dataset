# RO_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ro.admin`
Version: `v1.0.1`
Country: `RO`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `ro.admin v1.0.1` under Cadis Runtime after removing sparse locality-level geometry from the runtime contract.

Romania level `9` is sparse local detail. It is valid OSM data, but Cadis' stable semantic contract for Romania is county plus municipality/commune. Version `v1.0.1` keeps levels `4` and `8`, and excludes level `9`.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `ro.admin` |
| Dataset Version | `v1.0.1` |
| Country | `RO` |
| Country Name | `Romania` |
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
| Boundary Builder | `scripts/build_ro_boundaries.py` |
| Generated Boundary | `tmp/ro_country.json` |
| Boundary Source | `Natural Earth admin-0` |
| Boundary BBox | `[20.24282596900008, 43.6500499480001, 29.699554884000065, 48.27483225600007]` |
| OSM Source | `geofabrik:europe/romania` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for build and evaluation.

---

# 4. Administrative Model

The reduced Romania engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_county` | 42 | Counties and Bucharest |
| 8 | `admin_municipality` | 3,165 | Cities, towns, communes, and municipalities |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 7 | 7 | Excluded as metropolitan association coverage |
| 9 | 19 | Excluded from `v1.0.1`; sparse locality detail is outside the Cadis semantic contract |
| 10 | 4 | Excluded as sparse neighborhood/border detail |

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Expected inside ratio: `0.9`
* Dataset path: `RO/ro.admin/v1.0.1`

Sampling is uniform over land area, not population-weighted.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `12113.030` QPS |
| Total Runtime | `0.826 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 9,018 / 1 / 981 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 9,018 / 1 / 981 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 9,018 / 1 / 981 / 0 |
| no_nearby | 99.73% | 99.70% | 27 | 27 | 8,972 / 1 / 1,027 / 0 |
| osm_only | 99.73% | 99.70% | 27 | 27 | 8,972 / 1 / 1,027 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 46 |
| Total vs OSM-only | 46 |

---

# 9. Structural Distribution

| Shape | Count |
| ----- | ----: |
| `[4,8]` | 8,949 |
| `[]` | 1,000 |
| `[4]` | 50 |
| `[8]` | 1 |

| Source | Count |
| ------ | ----: |
| polygon | 8,973 |
| nearby | 27 |
| admin_tree_id | 15 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 9,000 |
| empty_shape | 981 |
| offshore | 19 |

---

# 10. Level-4 Coverage

The run hit all `42` level-4 county units, with `8,999` total level-4 hits and `8,997` inside level-4 hits.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ro_country.json` scoped boundary was used for build and evaluation.

---

# 12. Structural Observations

1. The runtime contract now exposes county and municipality/commune semantics only.
2. Level 9 was removed because sparse locality detail is not needed for Cadis semantic resolution.
3. Package size is now `6.0 MB` compressed and `7.2 MB` unpacked, down slightly from the prior `6.3 MB` compressed and `7.6 MB` unpacked release.
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
  `337bedf448b7a7b0100e652fabff3b6a3668aff059e17cd5152176ce7f87859d`
- Boundary builder: `scripts/build_ro_boundaries.py`
- Build boundary: `tmp/ro_country.json`
- Evaluation boundary: `tmp/ro_country.json`
- Staged dataset: `RO/ro.admin/v1.0.1`

---

# 14. Conclusion

The `ro.admin v1.0.1` dataset passes evaluation and is suitable for release. The runtime model is better aligned with Cadis' role as a semantic resolver by excluding sparse locality-level detail.
