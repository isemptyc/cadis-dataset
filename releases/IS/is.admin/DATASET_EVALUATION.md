# IS_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `is.admin`
Version: `v1.0.0`
Country: `IS`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates the `is.admin v1.0.0` dataset under Cadis Runtime.

The Iceland dataset models administrative lookup using Natural Earth scoped country
boundaries and OpenStreetMap administrative relations. The runtime policy exposes
Icelandic regions, municipalities, and available urban district/neighborhood
relations while keeping runtime behavior country-agnostic.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `is.admin` |
| Dataset Version | `v1.0.0` |
| Country | `IS` |
| Country Name | `Iceland` |
| Policy Version | `1.0` |
| Cadis Version Evaluated | `0.8.2` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |

---

# 3. Scope and Boundary

Dataset scope is full ISO2 Iceland.

The evaluation uses a tracked boundary builder:

* `scripts/build_is_boundaries.py`

Boundary derivation rule:

* Read Natural Earth admin-0 country data.
* Select rows where `ISO_A2 == IS` or `ADMIN == Iceland`.
* Write the deterministic boundary JSON used for build scoping and evaluation sampling.

Generated boundary bbox:

* `[-24.539906378999945, 63.39671458500004, -13.502919074999909, 66.56415436400005]`

This explicit boundary is required because the generic Natural Earth evaluator
column scan can also match Israel via `POSTAL=IS`. The tracked boundary builder
prevents that cross-country sampling contamination.

---

# 4. Administrative Model

Allowed levels:

* `5`: region
* `6`: municipality
* `9`: district
* `10`: neighborhood

Completion policy:

* `ok`: `[5,6]`, `[5,6,9]`, `[5,6,10]`, `[5,6,9,10]`
* `partial`: sparse shapes without the complete region/municipality path

OSM source structure observed in the build:

| Level | Count |
| --- | ---: |
| `5` | 8 |
| `6` | 62 |
| `9` | 16 |
| `10` | 117 |

---

# 5. Test Methodology

Runtime mass validation used mixed inside/outside stress testing.

| Field | Value |
| --- | ---: |
| Total samples | 10,000 |
| Expected inside samples | 9,000 |
| Expected outside samples | 1,000 |
| Inside ratio | 0.9 |
| Lookup mode | `runtime` |
| Workers | 1 |

The run used one worker because the current parallel evaluator path undercounts
processed futures. Single-worker runtime mode records all 10,000 samples.

---

# 6. Performance and Pass Rates

| Metric | Value |
| --- | ---: |
| Throughput | 14,952.98 QPS |
| Total runtime | 0.669 sec |
| Overall pass rate | 100.00% |
| Policy pass rate | 100.00% |
| Inside coverage pass rate | 100.00% |
| Failed samples | 0 |
| Inside failed samples | 0 |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (`ok`/`partial`/`failed`/`unknown`) |
| --- | ---: | ---: | ---: | ---: | --- |
| `full_policy` | 100.00% | 100.00% | 0 | 0 | 9,012 / 29 / 959 / 0 |
| `no_hierarchy` | 100.00% | 100.00% | 0 | 0 | 9,012 / 29 / 959 / 0 |
| `no_repair` | 100.00% | 100.00% | 0 | 0 | 9,012 / 29 / 959 / 0 |
| `no_nearby` | 98.86% | 98.73% | 114 | 114 | 8,859 / 29 / 1,112 / 0 |
| `osm_only` | 98.86% | 98.73% | 114 | 114 | 8,859 / 29 / 1,112 / 0 |

---

# 8. Layer Contribution

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 153 |
| Total vs OSM-only | 153 |

Nearby fallback resolves bounded coastal adjacency cases. No repair layer is
required for this dataset.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[5,6]` | 8,956 |
| `[]` | 993 |
| `[5]` | 29 |
| `[5,6,9]` | 13 |
| `[5,6,9,10]` | 5 |
| `[5,6,10]` | 4 |

Empty shapes correspond to expected outside/offshore outcomes:

| Reason | Count |
| --- | ---: |
| `empty_shape` | 959 |
| `offshore` | 34 |

## 9.2 Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | 8,888 |
| `nearby` | 119 |
| `admin_tree_id` | 25 |

| Source Mix | Count |
| --- | ---: |
| `polygon` | 8,863 |
| `__none__` | 993 |
| `nearby` | 119 |
| `admin_tree_id|polygon` | 25 |

---

# 10. Boundary Isolation Validation

The evaluation intentionally included 1,000 expected outside samples.

No boundary leak or cross-country escalation was observed. All inside-labeled
samples passed coverage validation, and outside/offshore samples remained
contained by the runtime policy.

---

# 11. Structural Observations

1. Iceland's stable nationwide administrative path is region plus municipality (`[5,6]`).
2. Urban district/neighborhood levels are available only in limited areas and are optional enrichments.
3. The repair layer is not needed.
4. Nearby fallback materially improves coastal coverage.
5. Explicit boundary JSON is required for reliable evaluation because the generic NE ISO-like scan matches Israel's `POSTAL=IS`.

---

# 11. Reproducibility

Dataset package was generated from:

* cadis-dataset-engine commit:
  `18370af01e95ff030e13437e5274f85adf393196`
* Cadis version:
  `0.8.2`

---

# 12. Conclusion

The `is.admin v1.0.0` dataset is suitable for release.

It achieves full policy and inside-boundary pass rates under the scoped Iceland
boundary, preserves deterministic runtime artifacts, and models Iceland's normal
administrative output as region plus municipality with optional lower-level
urban detail.
