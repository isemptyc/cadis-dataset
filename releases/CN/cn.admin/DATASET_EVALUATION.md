# CN_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `cn.admin`
Version: `v1.0.3`
Country: `CN`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates `cn.admin v1.0.3` under Cadis Runtime after removing detailed locality/admin-unit geometry from the runtime contract.

This release keeps `CN` as the public Cadis-routable dataset and includes Tibet as an internal source component. Tibet remains anchored by level 5 and level 6 administrative coverage because the Natural Earth world classifier resolves Tibet coordinates to `CN`.

Cadis is a semantic resolver, not a GIS boundary system. Version `v1.0.3` therefore keeps the stable China/Tibet administrative anchors at levels 4, 5, and 6, while excluding lower locality/detail levels.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `cn.admin` |
| Dataset Version | `v1.0.3` |
| Country | `CN` |
| Country Name | `China` |
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
| Boundary Builder | `scripts/build_cn_boundaries.py` |
| Generated Boundary | `tmp/cn_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | `Use OSM administrative relation areas at level 4 whose representative point is covered by the Natural Earth CN boundary, plus Tibet internal sub-engine level 5 and level 6 administrative anchors, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox | `[73.50254522441283, 20.12183040413475, 134.77345103091676, 53.55881051212936]` |
| OSM Source | `geofabrik:asia/china+asia/tibet` |
| OSM Snapshot Timestamp | `2026-05-11T20:20:52Z` |

The same scoped boundary was used for build and evaluation.

---

# 4. Administrative Model

The reduced China engine exposes these OSM administrative levels:

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_region` | 27 | China primary anchor coverage |
| 5 | `admin_district` | 322 | Includes Tibet internal source anchors |
| 6 | `admin_municipality` | 2,736 | Includes Tibet internal source anchors where level 5 coverage is absent |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 7 | 140 | Excluded from `v1.0.3`; sparse locality detail |
| 8 | 18,056 | Excluded from `v1.0.3`; locality layer too detailed for Cadis semantic resolution |
| 9 | 304 | Excluded from `v1.0.3`; sparse locality detail |
| 10 | 7,789 | Excluded from `v1.0.3`; detailed layer too fine-grained for Cadis semantic resolution |
| 11 | 137 | Excluded from `v1.0.3`; sparse admin-unit detail |
| 12 | 4 | Excluded from `v1.0.3`; tiny admin-unit detail |
| 14 | 0 | No scoped rows in this snapshot |

The engine uses canonical names from `name:en`, `name`, `official_name`, `name:zh`, `name:bo`, and `name:ru`, with bounded multilingual aliases for `en`, `zh`, `bo`, and `ru`.

---

# 5. Test Methodology

* Total samples: `100,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `85,000`
* Outside samples: `15,000`
* Dataset path: `CN/cn.admin/v1.0.3`

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| HTTP 200 Responses | `100,000` |
| Throughput | `1086.850` QPS |
| Total Runtime | `92.009 sec` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 85,039 / 0 / 14,961 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 85,039 / 0 / 14,961 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 85,039 / 0 / 14,961 / 0 |
| no_nearby | 100.00% | 100.00% | 1 | 1 | 84,999 / 0 / 15,001 / 0 |
| osm_only | 100.00% | 100.00% | 1 | 1 | 84,999 / 0 / 15,001 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 40 |
| Total vs OSM-only | 40 |

---

# 9. Structural Distribution

| Shape | Count |
| ----- | ----: |
| `[4,5,6]` | 73,394 |
| `[]` | 14,996 |
| `[5,6]` | 8,499 |
| `[6]` | 1,429 |
| `[4,6]` | 1,285 |
| `[4]` | 334 |
| `[4,5]` | 63 |

| Source | Count |
| ------ | ----: |
| polygon | 84,999 |
| admin_tree_id | 15 |
| nearby | 5 |

| Reason | Count |
| ------ | ----: |
| shape_status_map | 85,004 |
| empty_shape | 14,961 |
| offshore | 35 |

---

# 10. Level-4 Coverage

China proper is anchored by level 4 administrative coverage, while the Tibet source component is anchored at levels 5 and 6.

* Unique level-4 units hit: `27`
* Total level-4 hits (all points): `75,076`
* Total level-4 hits (inside points): `75,072`

---

# 11. Boundary Isolation Validation

Under stress testing with 15% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/cn_country.json` scoped boundary was used for build and evaluation.

---

# 12. Structural Observations

1. The runtime contract now exposes levels 4, 5, and 6 only.
2. China proper remains anchored by level 4 administrative coverage.
3. Tibet remains included as an internal source component anchored at level 5 or level 6.
4. Lower locality/detail levels were removed because they are too fine-grained for Cadis semantic resolution and made package metadata/hierarchy disproportionate.
5. Package size is now `5.4 MB` compressed and `6.9 MB` unpacked, down from the prior `14.2 MB` compressed and `26.7 MB` unpacked release.
6. Evaluation behavior remains stable, with 100% overall, policy, and inside pass rates.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `b0e83736c7df45ecd2ebdc732c209e2a3ce7d472`
- Cadis version:
  `0.8.160`
- Runtime compatibility minimum:
  `0.8.35`
- Boundary builder: `scripts/build_cn_boundaries.py`
- Build boundary: `tmp/cn_country.json`
- Evaluation boundary: `tmp/cn_country.json`
- Staged dataset: `CN/cn.admin/v1.0.3`
- Source OSM manifest SHA256:
  `a0e099ef0a2354b890463b615e240ccc51d7950cddd19c45b6d211966d053bdf`

---

# 14. Conclusion

The `cn.admin v1.0.3` dataset passes evaluation and is suitable for release. The runtime model preserves China and Tibet semantic anchors while excluding overly detailed locality/admin-unit geometry.
