# AT_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `at.admin`
Version: `v1.0.0`
Country: `AT`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates the `at.admin v1.0.0` dataset under Cadis Runtime.

The Austria dataset models administrative lookup using a Natural Earth scoped
country boundary and OpenStreetMap administrative relations. The runtime policy
exposes states, districts/statutory cities, and municipalities while keeping
runtime behavior country-agnostic.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `at.admin` |
| Dataset Version | `v1.0.0` |
| Country | `AT` |
| Country Name | `Austria` |
| Policy Version | `1.0` |
| Cadis Version Evaluated | `0.8.5` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |

---

# 3. Scope and Boundary

Dataset scope is full ISO2 Austria.

Boundary derivation rule:

* Read Natural Earth admin-0 country data.
* Select Austria by `ISO_A2 == AT` or country name.
* Write the deterministic boundary JSON used for build scoping and evaluation sampling.

Generated boundary bbox:

* `[9.52115482500011, 46.37864308700007, 17.148337850000075, 49.00977447500007]`

The same generated boundary was used for both build scoping and evaluation
sampling.

---

# 4. Administrative Model

Allowed levels:

* `4`: state
* `6`: district / statutory city
* `8`: municipality

Completion policy:

* `ok`: `[4,6]`, `[4,6,8]`, `[4,8]`
* `partial`: `[4]`, `[6]`, `[6,8]`, `[8]`

Source probe observations:

| Level | Count |
| --- | ---: |
| 2 | 1 |
| 4 | 9 |
| 6 | 93 |
| 7 | 1 |
| 8 | 2,073 |
| 9 | 79 |
| 10 | 3,754 |

The release model intentionally excludes levels `9` and `10` because they are
submunicipal/cadastral-heavy and not stable country-wide administrative output
levels for the initial Cadis contract.

The build produced `2,174` administrative nodes and `2,165` dataset-scoped
parent edges after excluding one known cross-border extract leak:

* `at_r1155951`: Mauren, Liechtenstein

---

# 5. Test Methodology

Mass lookup validation used mixed inside/outside stress testing.

| Metric | Value |
| --- | ---: |
| Total samples | 10,000 |
| Expected inside samples | 9,000 |
| Expected outside samples | 1,000 |
| Workers | 1 |
| Max sampling attempts | 2,000,000 |

The test intentionally injects approximately 10% out-of-country samples to
validate boundary rejection, offshore classification, and cross-border
isolation.

---

# 6. Performance and Pass Rates

| Metric | Value |
| --- | ---: |
| Runtime | 0.863 sec |
| Throughput | 11,590.180 QPS |
| Overall pass rate | 100.00% |
| Policy pass rate | 100.00% |
| Inside coverage pass rate | 100.00% |
| Failed samples | 0 |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| --- | ---: | ---: | ---: | ---: |
| `full_policy` | 100.00% | 100.00% | 0 | 0 |
| `no_hierarchy` | 100.00% | 100.00% | 0 | 0 |
| `no_repair` | 100.00% | 100.00% | 0 | 0 |
| `no_nearby` | 99.13% | 99.03% | 87 | 87 |
| `osm_only` | 99.13% | 99.03% | 87 | 87 |

Layer effect summary:

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 106 |
| Total vs OSM-only | 106 |

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[4,6,8]` | 8,799 |
| `[]` | 998 |
| `[4,6]` | 113 |
| `[4]` | 70 |
| `[4,8]` | 19 |
| `[6,8]` | 1 |

Empty shapes correspond to expected outside or offshore outcomes:

* `empty_shape`: 979
* `offshore`: 19

## 8.2 Node Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | 8,915 |
| `nearby` | 87 |
| `admin_tree_id` | 26 |

## 8.3 Policy Reason Distribution

| Reason | Count |
| --- | ---: |
| `shape_status_map` | 9,002 |
| `empty_shape` | 979 |
| `offshore` | 19 |

---

# 9. Level-4 Coverage

All 9 Austrian states were hit during uniform boundary sampling.

| Level-4 Unit | Hits | Hit Rate | Hits Inside | Inside Hit Rate |
| --- | ---: | ---: | ---: | ---: |
| `Niederösterreich` | 2,029 | 20.29% | 2,029 | 22.54% |
| `Steiermark` | 1,792 | 17.92% | 1,792 | 19.91% |
| `Tirol` | 1,323 | 13.23% | 1,322 | 14.69% |
| `Oberösterreich` | 1,295 | 12.95% | 1,294 | 14.38% |
| `Kärnten` | 1,063 | 10.63% | 1,063 | 11.81% |
| `Salzburg` | 789 | 7.89% | 789 | 8.77% |
| `Burgenland` | 375 | 3.75% | 375 | 4.17% |
| `Vorarlberg` | 277 | 2.77% | 277 | 3.08% |
| `Wien` | 58 | 0.58% | 58 | 0.64% |

Hit distribution reflects uniform land-area sampling, not population-weighted
coverage.

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No policy or coverage failure was observed.
* Expected outside samples resolved as empty-shape or offshore outcomes.
* Nearby fallback rescued 87 inside-boundary samples that would otherwise have
  failed under OSM-only geometry.
* No evidence was observed that hierarchy or nearby layers created cross-border
  escalation in this run.

---

# 11. Build Note

This evaluation package was staged by the official Docker-backed SOP command.

Build image:

* `ghcr.io/isemptyc/cadis-dataset-engine@sha256:e5b134214a51368391576e5bbe6b06b403e3acf19d9d91124063caba72f1ae76`

---

# 12. Reproducibility

Inputs:

* OSM source: `geofabrik:europe/Austria`
* OSM file: `austria-260426.osm.pbf`
* OSM replication timestamp: `2026-04-26T20:21:04Z`
* OSM SHA256: `bcab8519aa625b8b490e8dd41481682307d53d063b5457550024d29b1f267512`
* cadis-dataset-engine commit: `2c27022da62fec9ec92e8220ed91ec8296500d8c`
* Cadis version: `0.8.5`

---

# 13. Conclusion

The `at.admin v1.0.0` dataset is structurally sound for the initial Austria
Cadis contract:

* Full inside-boundary pass rate under policy mode.
* Stable level model using states, districts/statutory cities, and municipalities.
* No semantic repair layer required.
* Nearby fallback is bounded and materially improves edge coverage.
* Cross-border extract leakage was identified and explicitly excluded.

