# IE_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ie.admin`
Version: `v1.0.0`
Country: `IE`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ie.admin v1.0.0` dataset under Cadis Runtime.

This release keeps `IE` as a standalone Cadis-routable dataset. It is built from the Geofabrik Ireland and Northern Ireland source extract, scoped to the Natural Earth Ireland admin-0 boundary so Northern Ireland remains routed through `gb.admin`.

Cadis does not modify geography. It enforces structural determinism over the administrative coverage available in the source data.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `ie.admin` |
| Dataset Version | `v1.0.0` |
| Country | `IE` |
| Country Name | `Ireland` |
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
| Scope Label | `full ISO2 country scope` |
| Boundary Builder | `scripts/build_ie_boundaries.py` |
| Generated Boundary | `tmp/ie_country.json` |
| Boundary Source | `Natural Earth admin-0` |
| Boundary Selection Rule | `Natural Earth IE admin-0 country boundary, used to clip the Ireland portion out of the Ireland and Northern Ireland source extract.` |
| Boundary BBox | `[-10.478179490999935, 51.44570547100005, -5.993519660999937, 55.386379299000055]` |
| OSM Source | `geofabrik:europe/ireland-and-northern-ireland` |
| OSM Snapshot Timestamp | `2026-05-11T20:20:52Z` |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 5 | `admin_province` | 4 | Province-level coverage |
| 6 | `admin_county` | 26 | County-level coverage |
| 7 | `admin_city` | 20 | City/local authority coverage where available |
| 8 | `admin_municipal_district` | 98 | Municipal district coverage |
| 9 | `admin_civil_parish` | 3,582 | Civil parish coverage |

The engine uses canonical names from `name:en`, `name`, `name:ga`, and `official_name`, with bounded multilingual aliases for English and Irish. Level 10 townlands are intentionally excluded from this release to keep the runtime dataset aligned with Cadis country-package size expectations.

---

# 5. Test Methodology

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `8,500`
* Outside samples: `1,500`
* Dataset path: `IE/ie.admin/v1.0.0`

The test intentionally injects out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| HTTP 200 Responses | `10,000` |
| Throughput | `7134.980` QPS |
| Total Runtime | `1.402 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 3,935 / 4,615 / 1,450 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 3,935 / 4,615 / 1,450 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 3,935 / 4,615 / 1,450 / 0 |
| no_nearby | 99.66% | 99.60% | 34 | 34 | 3,871 / 4,598 / 1,531 / 0 |
| osm_only | 99.66% | 99.60% | 34 | 34 | 3,871 / 4,598 / 1,531 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 81 |
| Total vs OSM-only | 81 |

No explicit repair layer is required for `ie.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[5,6,8,9]` | 4,305 |
| `[5,6,7,8,9]` | 3,896 |
| `[]` | 1,489 |
| `[5,6,7,9]` | 218 |
| `[5,6,7,8]` | 30 |
| `[6,8,9]` | 15 |
| `[5]` | 11 |
| `[5,6,7]` | 9 |
| Other valid administrative shapes | 27 |

Empty shapes correspond to expected outside/offshore samples:

* `empty_shape`: 1,450
* `offshore`: 39

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 8,469 |
| nearby | 42 |
| admin_tree_id | 21 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 8,511 |
| empty_shape | 1,450 |
| offshore | 39 |

---

# 10. Boundary Isolation Validation

Under stress testing with 15% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ie_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the IE dataset scope.

---

# 11. Runtime Smoke

A direct Dublin smoke lookup returned `Leinster > County Dublin > Dublin > North City Ward 1986`, with English and Irish names present on the province, county, and city nodes.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. Ireland is published as a standalone `IE` dataset package.
3. The Ireland/Northern Ireland source extract is clipped to the Natural Earth IE boundary, leaving Northern Ireland routing to `gb.admin`.
4. Level 9 civil parish coverage is included; level 10 townlands are intentionally omitted for package-size discipline.
5. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges where observed.
6. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `629081b95fb6c37816e2a9796106a8a5c4abe208`
- Cadis version:
  `0.8.160`
- Boundary builder: `scripts/build_ie_boundaries.py`
- Build boundary: `tmp/ie_country.json`
- Evaluation boundary: `tmp/ie_country.json`
- Staged dataset: `IE/ie.admin/v1.0.0`
- Source OSM manifest SHA256:
  `fed9ae5866fdde0f6582939b2f10a3393509416d281bba7e6336fc6fa7f07d6e`
- Source OSM component:
  `ireland-and-northern-ireland-260511.osm.pbf`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ie.admin v1.0.0` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and practical administrative coverage for Ireland.

The dataset is suitable for autonomous release.
