# CH_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ch.admin`
Version: `v1.0.0`
Country: `CH`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates the `ch.admin v1.0.0` dataset under Cadis Runtime.

The Switzerland dataset models administrative lookup using a Natural Earth scoped
country boundary and OpenStreetMap administrative relations. The runtime policy
exposes cantons, districts, and municipalities while keeping runtime behavior
country-agnostic.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `ch.admin` |
| Dataset Version | `v1.0.0` |
| Country | `CH` |
| Country Name | `Switzerland` |
| Policy Version | `1.0` |
| Cadis Version Evaluated | `0.8.4` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |

---

# 3. Scope and Boundary

Dataset scope is full ISO2 Switzerland.

Boundary derivation rule:

* Read Natural Earth admin-0 country data.
* Select Switzerland by `ISO_A2 == CH` or country name.
* Write the deterministic boundary JSON used for build scoping and evaluation sampling.

Generated boundary bbox:

* `[5.954809204000128, 45.82071848599999, 10.466626831000013, 47.801166077000076]`

The same generated boundary was used for both build scoping and evaluation
sampling.

---

# 4. Administrative Model

Allowed levels:

* `4`: canton
* `6`: district
* `8`: municipality

Completion policy:

* `ok`: `[4,6]`, `[4,6,8]`, `[4,8]`
* `partial`: `[4]`, `[6]`, `[6,8]`, `[8]`

OSM source structure observed in the build:

| Level | Count |
| --- | ---: |
| 4 | 26 |
| 6 | 131 |
| 8 | 2,108 |

The build produced `2,265` administrative nodes and `2,035` dataset-scoped
parent edges. Some municipalities legitimately appear without an intermediate
district path; the `[4,8]` shape is therefore accepted as complete.

---

# 5. Test Methodology

Runtime mass validation used:

* Samples: `10,000`
* Inside ratio: `0.9`

---

# 6. Performance Metrics

| Metric | Value |
| --- | ---: |
| Throughput | `11649.690` QPS |
| Total Runtime | `0.858 sec` |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |

No failed samples were recorded.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| --- | ---: | ---: | ---: | ---: |
| full_policy | 100.00% | 100.00% | 0 | 0 |
| no_hierarchy | 100.00% | 100.00% | 0 | 0 |
| no_repair | 100.00% | 100.00% | 0 | 0 |
| no_nearby | 98.43% | 98.26% | 157 | 157 |
| osm_only | 98.43% | 98.26% | 157 | 157 |

Layer contribution:

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 194 |
| Total vs OSM-only | 194 |

Nearby fallback is the only policy layer that materially changes sampled
outcomes. No repair layer is required.

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[4,6,8]` | 7,725 |
| `[4,8]` | 1,188 |
| `[]` | 996 |
| `[4]` | 70 |
| `[8]` | 12 |
| `[4,6]` | 6 |
| `[6,8]` | 3 |

## 8.2 Source Distribution

| Source | Count |
| --- | ---: |
| polygon | 8,844 |
| nearby | 160 |
| admin_tree_id | 48 |

Policy reason distribution:

| Reason | Count |
| --- | ---: |
| shape_status_map | 9,004 |
| empty_shape | 962 |
| offshore | 34 |

---

# 9. Level-4 Coverage

All 26 Swiss cantons were hit during uniform boundary sampling.

Top sampled level-4 units:

| Level-4 Unit | Hits | Inside Hits |
| --- | ---: | ---: |
| Graubünden/Grischun/Grigioni | 1,516 | 1,514 |
| Bern/Berne | 1,306 | 1,306 |
| Valais/Wallis | 1,138 | 1,136 |
| Vaud | 675 | 675 |
| Ticino | 660 | 659 |
| St. Gallen | 413 | 412 |
| Zürich | 412 | 412 |
| Fribourg/Freiburg | 370 | 370 |
| Luzern | 336 | 336 |
| Aargau | 306 | 305 |

Hit distribution reflects uniform land-area sampling, not population weighting.

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed.
* Empty-shape and offshore outcomes accounted for expected outside samples.
* The same full-country boundary was used for build and evaluation.
* No evidence was observed that nearby fallback created cross-border escalation.

---

# 11. Reproducibility

Dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit:
  `051357c09b9d68bc7df13b81244b02083750f173`
* Cadis version:
  `0.8.4`
* OSM source:
  `geofabrik:europe/Switzerland`
* OSM replication timestamp:
  `2026-04-26T20:21:04Z`
* OSM SHA256:
  `432bff8faff3e8322d8df18389eaa0fbfb3b841ff93d876bdbe0a5e5e03f0526`

---

# 12. Conclusion

The `ch.admin v1.0.0` staged dataset is acceptable for publish-gate review.

It demonstrates:

* Full inside-boundary coverage under policy mode
* Full policy pass rate
* Complete level-4 canton sampling coverage
* No repair-layer requirement
* Bounded nearby fallback for edge/coastal-border cases
* Strict full-country boundary scoping using a tracked builder script
