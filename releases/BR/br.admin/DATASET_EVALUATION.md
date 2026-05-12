# BR_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `br.admin`
Version: `v1.0.1`
Country: `BR`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `br.admin v1.0.1` under Cadis Runtime after removing district-level geometry from the runtime contract.

Brazil level `9` provides submunicipal district detail, but Cadis is a semantic resolver rather than a GIS boundary system. The stable runtime semantics for Brazil are state and municipality. Version `v1.0.1` keeps levels `4` and `8`, and excludes level `9` to keep the package size proportional to the semantic contract.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `br.admin` |
| Dataset Version | `v1.0.1` |
| Country | `BR` |
| Country Name | `Brazil` |
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
| Boundary Builder | `scripts/build_br_boundaries.py` |
| Generated Boundary | `tmp/br_country.json` |
| Boundary Source | `Natural Earth admin-0 with OSM level-4 state coverage` |
| Boundary BBox | `[-73.9830625, -33.8694284, -28.6289646, 5.2695808]` |
| OSM Source | `geofabrik:south-america/brazil` |
| OSM Snapshot Timestamp | `2026-05-09T20:21:02Z` |

The same scoped boundary was used for build and evaluation.

---

# 4. Administrative Model

The reduced Brazil engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_state` | 27 | States and Federal District |
| 8 | `admin_municipality` | 5,602 | Municipalities |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 9 | 6,721 | Excluded from `v1.0.1`; submunicipal district detail is not required for Cadis semantic resolution |
| 10 | many | Excluded as neighborhood-style detail with substantial parent incompleteness |

---

# 5. Test Methodology

* Total samples: `50,000`
* Sampling mode: mixed inside/outside stress testing
* Expected inside ratio: `0.9`
* Dataset path: `BR/br.admin/v1.0.1`

Sampling is uniform over the scoped boundary area, not population-weighted.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| Throughput | `2575.320` QPS |
| Total Runtime | `19.415 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 43,655 / 1,354 / 4,991 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 43,655 / 1,354 / 4,991 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 43,655 / 1,354 / 4,991 / 0 |
| no_nearby | 99.98% | 99.98% | 10 | 10 | 43,645 / 1,345 / 5,010 / 0 |
| osm_only | 99.98% | 99.98% | 10 | 10 | 43,645 / 1,345 / 5,010 / 0 |

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
| `[4,8]` | 43,647 |
| `[]` | 4,999 |
| `[4]` | 1,354 |

| Source | Count |
| ------ | ----: |
| polygon | 44,990 |
| admin_tree_id | 15 |
| nearby | 11 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 45,001 |
| empty_shape | 4,991 |
| offshore | 8 |

---

# 10. Level-4 Coverage

The run hit all `27` level-4 units, with `45,001` total level-4 hits and `45,000` inside level-4 hits.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* All 45,000 inside samples produced passing outcomes.
* Empty-shape and offshore outcomes were confined to expected outside/offshore behavior.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.

---

# 12. Structural Observations

1. The runtime contract now exposes state and municipality semantics only.
2. Level 9 district geometry was removed because it adds 6,721 submunicipal polygons without being necessary for Cadis semantic resolution.
3. Package size is now `4.4 MB` compressed and `6.5 MB` unpacked, down from the prior `32.2 MB` compressed and `37.7 MB` unpacked release.
4. Evaluation behavior remains stable, with 100% overall, policy, and inside pass rates.
5. Nearby fallback remains minimal and bounded.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `6ec44a6df2706c48e51a12286d79af16fdb9867d`
- Cadis version:
  `0.8.160`
- Runtime compatibility minimum:
  `0.8.35`
- Source OSM SHA256:
  `62cd166273572c84a50dc8f7e9ffef00f5cb240c4a5fe9b07bd43de1a535278a`
- Boundary builder: `scripts/build_br_boundaries.py`
- Build boundary: `tmp/br_country.json`
- Evaluation boundary: `tmp/br_country.json`
- Staged dataset: `BR/br.admin/v1.0.1`

---

# 14. Conclusion

The `br.admin v1.0.1` dataset passes evaluation and is suitable for release. The runtime model is better aligned with Cadis' role as a semantic resolver by excluding district-level detail.
