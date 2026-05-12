# GB_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `gb.admin`
Version: `v1.0.2`
Country: `GB`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `gb.admin v1.0.2` dataset under Cadis Runtime.

This release keeps `GB` as the public Cadis-routable dataset and treats Northern Ireland as an internal source component. The package is built from a Great Britain source extract plus the Northern Ireland portion of the Ireland and Northern Ireland source extract.

Cadis does not modify geography. It enforces structural determinism over the administrative coverage available in the source data.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `gb.admin` |
| Dataset Version | `v1.0.2` |
| Country | `GB` |
| Country Name | `United Kingdom` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.158` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `full ISO2 country scope` |
| Boundary Builder | `scripts/build_gb_boundaries.py` |
| Generated Boundary | `tmp/gb_country.json` |
| Boundary Source | `Natural Earth admin-0` |
| Boundary Selection Rule | `Full Natural Earth GB admin-0 country boundary, used to combine the Great Britain source extract with the Northern Ireland portion of the Ireland and Northern Ireland source extract.` |
| Boundary BBox | `[-13.69131425699993, 49.90961334800005, 1.7711694670000497, 60.84788646000004]` |
| OSM Source | `geofabrik:europe/great-britain+europe/ireland-and-northern-ireland` |
| OSM Snapshot Timestamp | `2026-05-11T20:20:52Z` |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count | Notes |
| ----: | ------------- | ------------: | ----- |
| 4 | `admin_country_region` | 4 | England, Scotland, Wales, Northern Ireland |
| 5 | `admin_county` | 16 | Coverage where available |
| 6 | `admin_district` | 137 | Coverage where relation-stable OSM areas are available |
| 7 | `admin_district` | 10 | Includes relation-stable Northern Ireland districts such as Belfast City District |
| 8 | `admin_locality` | 230 | Coverage where available |

The engine uses canonical names from `name:en`, `name`, and `official_name`, with bounded multilingual aliases for English, Welsh, and Scottish Gaelic.

---

# 5. Test Methodology

* Total samples: `100,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `85,000`
* Outside samples: `15,000`
* Dataset path: `GB/gb.admin/v1.0.2`

The test intentionally injects out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |
| HTTP 200 Responses | `100,000` |
| Throughput | `11343.560` QPS |
| Total Runtime | `8.816 sec` |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| -------- | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy | 100.00% | 100.00% | 0 | 0 | 81,423 / 3,922 / 14,655 / 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 | 81,423 / 3,922 / 14,655 / 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 | 81,423 / 3,922 / 14,655 / 0 |
| no_nearby | 99.97% | 99.96% | 33 | 33 | 81,248 / 3,907 / 14,845 / 0 |
| osm_only | 99.97% | 99.96% | 33 | 33 | 81,248 / 3,907 / 14,845 / 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| ----- | --------------- |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 190 |
| Total vs OSM-only | 190 |

No explicit repair layer is required for `gb.admin v1.0.2`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[4,6]` | 46,900 |
| `[]` | 14,795 |
| `[4,6,8]` | 13,090 |
| `[4,5,6,8]` | 7,516 |
| `[4,5,6]` | 7,342 |
| `[4,7]` | 6,435 |
| `[4,5,8]` | 2,869 |
| `[4]` | 1,003 |
| Other valid administrative shapes | 50 |

Empty shapes correspond to expected outside/offshore samples:

* `empty_shape`: 14,655
* `offshore`: 140

## 9.2 Node Source Distribution

| Source | Count |
| ------ | ----: |
| polygon | 85,155 |
| nearby | 50 |

## 9.3 Policy Reason Distribution

| Reason | Count |
| ------ | ----: |
| shape_status_map | 85,205 |
| empty_shape | 14,655 |
| offshore | 140 |

---

# 10. Level-4 Coverage

The dataset is anchored by level 4 administrative coverage for the United Kingdom constituent countries.

* Unique level-4 units hit: `4`
* Total level-4 hits (all points): `85,192`
* Total level-4 hits (inside points): `84,987`

| Level-4 Unit | Hits | Hit Rate (All Points) | Hits (Inside) | Hit Rate (Inside Points) |
| ------------ | ---: | --------------------: | ------------: | -----------------------: |
| `England` | 43,065 | 43.06% | 43,002 | 50.59% |
| `Scotland` | 28,586 | 28.59% | 28,474 | 33.50% |
| `Wales` | 6,830 | 6.83% | 6,811 | 8.01% |
| `Northern Ireland` | 6,711 | 6.71% | 6,700 | 7.88% |

A direct Belfast smoke lookup returned `Northern Ireland > Belfast City District`.

---

# 11. Boundary Isolation Validation

Under stress testing with 15% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/gb_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the GB dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. Northern Ireland is included as an internal source component of `gb.admin`.
3. Relation-stable level 7 districts are included so Northern Ireland locations can resolve below level 4 when source coverage is available.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges where observed.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `c3215ce8f173f17755573bb6d067aaed4fc3e1fc`
- Cadis version:
  `0.8.158`
- Boundary builder: `scripts/build_gb_boundaries.py`
- Build boundary: `tmp/gb_country.json`
- Evaluation boundary: `tmp/gb_country.json`
- Staged dataset: `GB/gb.admin/v1.0.2`
- Source OSM manifest SHA256:
  `ebb86f3dc2ce08596e75512d13a3f672d3519b14fec083093d198494a4d60543`
- Source OSM components:
  `great-britain-latest.osm.pbf`, `ireland-and-northern-ireland-latest.osm.pbf`
- Source OSM component SHA256:
  `great-britain-latest.osm.pbf=94f02e1b81936b70296e097981c821c473eb872c776808bfb32df5abb3ad260a`
  `ireland-and-northern-ireland-latest.osm.pbf=fed9ae5866fdde0f6582939b2f10a3393509416d281bba7e6336fc6fa7f07d6e`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `gb.admin v1.0.2` dataset demonstrates full inside-boundary coverage, strict boundary isolation, high geometric integrity, no required repair layer, bounded nearby fallback behavior, and stable composite administrative coverage for the United Kingdom with Northern Ireland included as an internal source component.

The dataset is suitable for autonomous release.
