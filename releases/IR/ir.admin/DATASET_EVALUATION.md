# IR_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ir.admin`
Version: `v1.0.1`
Country: `IR`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `ir.admin v1.0.1` under Cadis Runtime after removing detailed locality/admin-unit layers from the runtime contract.

Cadis is a semantic resolver, not a GIS boundary system. Version `v1.0.1` keeps the stable Iran administrative model at levels 4, 5, 6, and 7, while excluding lower sparse/detail levels.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `ir.admin` |
| Dataset Version | `v1.0.1` |
| Country | `IR` |
| Country Name | `Iran` |
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
| Boundary Builder | `scripts/build_ir_boundaries.py` |
| Build Boundary | `tmp/ir_country.json` |
| Evaluation Boundary | `tmp/ir_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 4 whose representative point is covered by the Natural Earth IR boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| OSM Source | `geofabrik:asia/iran` |
| OSM Snapshot Timestamp | `2026-05-10T20:20:31Z` |

The same scoped boundary was used for build and evaluation.

---

# 4. Administrative Model

The reduced Iran engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_region` | 29 | Province anchor coverage |
| 5 | `admin_district` | 453 | District coverage |
| 6 | `admin_municipality` | 1,062 | Municipality/county-like coverage |
| 7 | `admin_locality` | 3,190 | Dominant lower semantic layer |

The previous release also carried these detailed layers:

| Level | Previous Count | Decision |
| ----: | -------------: | -------- |
| 8 | 1,824 | Excluded from `v1.0.1`; detailed locality layer not needed for Cadis semantic resolution |
| 9 | 140 | Excluded from `v1.0.1`; sparse locality tail |
| 10 | 181 | Excluded from `v1.0.1`; sparse detail tail |
| 11 | 2,978 | Excluded from `v1.0.1`; admin-unit layer too fine-grained for the runtime contract |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:ir`, and `name:ru`, with bounded multilingual aliases.

---

# 5. Evaluation Methodology

* Total samples: `100,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `90,000`
* Outside samples: `10,000`
* Lookup mode: Cadis runtime
* Evaluation workers: `1`
* Dataset path: `IR/ir.admin/v1.0.1`

---

# 6. Evaluation Summary

| Metric | Value |
| ------ | ----: |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| HTTP 200 Responses | `100,000` |
| Throughput | `3055.650` QPS |
| Total Runtime | `32.726 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (ok/partial/failed/unknown) |
| -------- | --------: | ---------------: | -----: | ------------: | ----------------------------------- |
| `full_policy` | 100.00% | 100.00% | 0 | 0 | 90,060 / 2 / 9,938 / 0 |
| `no_hierarchy` | 100.00% | 100.00% | 0 | 0 | 90,060 / 2 / 9,938 / 0 |
| `no_repair` | 100.00% | 100.00% | 0 | 0 | 90,060 / 2 / 9,938 / 0 |
| `no_nearby` | 100.00% | 100.00% | 0 | 0 | 89,998 / 2 / 10,000 / 0 |
| `osm_only` | 100.00% | 100.00% | 0 | 0 | 89,998 / 2 / 10,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------: |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 62 |
| Total vs OSM-only | 62 |

No explicit repair layer is required for `ir.admin v1.0.1`.

---

# 9. Structural Distribution

| Shape | Count |
| ----- | ----: |
| `[4,5,6,7]` | 89,622 |
| `[]` | 9,989 |
| `[4,5,6]` | 335 |
| `[4,5,7]` | 30 |
| `[4,6,7]` | 16 |
| `[4]` | 3 |
| `[4,5]` | 2 |
| `[5,6,7]` | 2 |

| Source | Count |
| ------ | ----: |
| polygon | 90,000 |
| admin_tree_id | 44 |
| nearby | 11 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 90,011 |
| empty_shape | 9,938 |
| offshore | 51 |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `29`
* Total level-4 hits (all points): `90,009`
* Total level-4 hits (inside points): `89,998`

Top sampled level-4 units:

| Level-4 Unit | Hits | Hit Rate (All Points) | Hits (Inside) | Hit Rate (Inside Points) |
| ------------ | ---: | --------------------: | ------------: | -----------------------: |
| `Kerman Province` | 9,799 | 9.80% | 9,799 | 10.89% |
| `Sistan and Baluchestan Province` | 9,562 | 9.56% | 9,561 | 10.62% |
| `South Khorasan Province` | 8,243 | 8.24% | 8,243 | 9.16% |
| `Razavi Khorasan` | 6,556 | 6.56% | 6,555 | 7.28% |
| `Fars Province` | 6,480 | 6.48% | 6,479 | 7.20% |
| `Isfahan Province` | 5,828 | 5.83% | 5,828 | 6.48% |
| `Semnan Province` | 5,599 | 5.60% | 5,599 | 6.22% |
| `Hormozgan Province` | 5,487 | 5.49% | 5,486 | 6.10% |

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed.
* Empty-shape and offshore outcomes accounted for expected outside samples.
* No evidence was observed that hierarchy or nearby layers created cross-boundary escalation.
* The same `tmp/ir_country.json` scoped boundary was used for build and evaluation.

---

# 12. Structural Observations

1. The runtime contract now exposes levels 4, 5, 6, and 7 only.
2. Iran remains anchored by level 4 province administrative coverage.
3. Level 7 is retained because it dominates observed in-scope shapes.
4. Levels 8, 9, 10, and 11 were removed because they are sparse or too fine-grained for Cadis semantic resolution.
5. Package size is now `1.5 MB` compressed and `3.5 MB` unpacked, down from the prior `2.0 MB` compressed and `6.3 MB` unpacked release.
6. Evaluation behavior remains stable, with 100% overall, policy, and inside pass rates.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `cc0eac0e7f232b056e3ca493a332ad1413d85efb`
- Cadis version:
  `0.8.160`
- Runtime compatibility minimum:
  `0.8.35`
- Boundary builder: `scripts/build_ir_boundaries.py`
- Build boundary: `tmp/ir_country.json`
- Evaluation boundary: `tmp/ir_country.json`
- Staged dataset: `IR/ir.admin/v1.0.1`
- Source OSM manifest SHA256:
  `6622a2c27229384120115b27df6a22c9fe09a9b9b3e5ad5da48ad04d00b39cf7`

---

# 14. Conclusion

The `ir.admin v1.0.1` dataset passes evaluation and is suitable for release. The runtime model preserves Iran semantic administrative anchors while excluding overly detailed locality/admin-unit geometry.
