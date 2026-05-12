# CZ_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `cz.admin`
Version: `v1.0.1`
Country: `CZ`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `cz.admin v1.0.1` under Cadis Runtime after simplifying municipality geometry.

Version `v1.0.1` keeps the Czech Republic administrative model intact and applies France-style simplification to level `8` municipality geometry. This addresses package size without removing semantically important municipality coverage.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `cz.admin` |
| Dataset Version | `v1.0.1` |
| Country | `CZ` |
| Country Name | `Czech Republic` |
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
| Boundary Builder | `scripts/build_cz_boundaries.py` |
| Generated Boundary | `tmp/cz_country.json` |
| Boundary Source | `Natural Earth admin-0` |
| Boundary BBox | `[12.076140991000045, 48.557915752, 18.837433716000106, 51.04001230900009]` |
| OSM Source | `geofabrik:europe/czech-republic` |
| OSM Snapshot Timestamp | `2026-04-26T20:21:04Z` |

The same scoped boundary was used for build and evaluation.

---

# 4. Administrative Model

The Czech Republic engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_region` | 14 | Regions and Prague |
| 6 | `admin_orp` | 227 | ORP administrative units |
| 7 | `admin_pou` | 392 | POU administrative units |
| 8 | `admin_municipality` | 6,234 | Municipalities, simplified in `v1.0.1` |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2 | 1 | Excluded as country-level envelope |
| 3 | 8 | Excluded as broad regional/statistical coverage |
| 5 | 86 | Excluded from the v1 runtime contract |
| 9 | 141 | Excluded as lower municipal subdivision coverage |
| 10 | 12,994 | Excluded as cadastral/detail coverage |

Three Polish border relations remain explicitly excluded by the engine:

| Feature ID | Name | Reason |
| ---------- | ---- | ------ |
| `cz_r6076233` | Wolanow | Poland relation |
| `cz_r6418836` | Wrzosowka | Poland relation |
| `cz_r6419556` | Bielice | Poland relation |

---

# 5. Test Methodology

* Total samples: `100,000`
* Sampling mode: mixed inside/outside stress testing
* Expected inside ratio: `0.9`
* Dataset path: `CZ/cz.admin/v1.0.1`

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `8146.150` QPS |
| Total Runtime | `12.276 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 90,193 / 94 / 9,713 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 90,193 / 94 / 9,713 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 90,193 / 94 / 9,713 / 0 |
| no_nearby | 99.18% | 99.08% | 825 | 825 | 89,085 / 94 / 10,821 / 0 |
| osm_only | 99.18% | 99.08% | 825 | 825 | 89,085 / 94 / 10,821 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 1,108 |
| Total vs OSM-only | 1,108 |

---

# 9. Structural Distribution

| Shape | Count |
| ----- | ----: |
| `[4,6,7,8]` | 88,045 |
| `[]` | 9,979 |
| `[4,6,7]` | 1,089 |
| `[4,6]` | 604 |
| `[4,7,8]` | 181 |
| `[4]` | 84 |
| `[6,7,8]` | 10 |
| `[4,7]` | 4 |

| Source | Count |
| ------ | ----: |
| polygon | 89,179 |
| nearby | 842 |
| admin_tree_id | 216 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 90,021 |
| empty_shape | 9,713 |
| offshore | 266 |

---

# 10. Level-4 Coverage

The run hit all `14` level-4 units, with `90,011` total level-4 hits and `89,967` inside level-4 hits.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No failed samples were observed.
* No boundary-leak failure was observed.
* Expected outside samples were classified as empty or offshore where appropriate.
* Hierarchy and nearby layers did not produce cross-border escalation in this run.

---

# 12. Structural Observations

1. The administrative model remains levels 4, 6, 7, and 8.
2. Level 8 municipality geometry is retained but simplified with tolerance `0.0001`, matching the France size-reduction pattern.
3. Package size is now `9.4 MB` compressed and `12.2 MB` unpacked, down from the prior `20.8 MB` compressed and `23.7 MB` unpacked release.
4. Evaluation behavior remains stable, with 100% overall, policy, and inside pass rates.
5. Nearby fallback remains bounded and accounts for expected edge misses.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `aadd5b21bc0ae0d9f666da7033b75fcd63d31cbb`
- Cadis version:
  `0.8.160`
- Runtime compatibility minimum:
  `0.8.35`
- Source OSM SHA256:
  `a9c678223b596f1980e713bfa9bec4a05d3bfa49cf6a4affe82da88b8ea0151a`
- Boundary builder: `scripts/build_cz_boundaries.py`
- Build boundary: `tmp/cz_country.json`
- Evaluation boundary: `tmp/cz_country.json`
- Staged dataset: `CZ/cz.admin/v1.0.1`

---

# 14. Conclusion

The `cz.admin v1.0.1` dataset passes evaluation and is suitable for release. The package-size issue is resolved through geometry simplification while preserving municipality coverage.
