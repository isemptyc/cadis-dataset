# JP_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `jp.admin`  
Version: `v1.0.2`  
Country: `JP`  
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `jp.admin v1.0.2` dataset under Cadis Runtime.

This report:

* Describes runtime evaluation conditions
* Quantifies structural outcomes under stress sampling
* Documents deterministic policy effects
* Validates boundary isolation with forced outside samples
* Provides reproducible metrics for the JP release candidate

OSM data is not assumed incorrect.
Observed anomalies in this report are interpreted as structural coverage outcomes under policy constraints.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `jp.admin` |
| Dataset Version | `v1.0.2` |
| Country | `JP` |
| Country Name | `Japan` |
| Policy Version | `1.0` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |

---

# 3. Test Methodology

## 3.1 Sampling Strategy

* Total samples: `30,000`
* Sampling mode: mixed inside/outside stress testing
* Inside ratio: `0.9`
* Expected outside ratio: `0.1`

The test intentionally injects ~10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification behavior
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over the country boundary envelope (not population-weighted).

---

# 4. Observed Distribution

| Category | Count |
| --- | ---: |
| Sample labels: expected inside points | 27,000 |
| Sample labels: expected outside points | 3,000 |
| Structural non-empty outcomes | 27,020 |
| Empty-shape outcomes (`[]`) | 2,980 |
| Offshore outcomes (subset of `[]`) | 1 |

Outside-labeled samples are intentional and do not indicate coverage defects by themselves.
`expected inside/outside` is from Natural Earth sampling labels; structural outcomes are from runtime lookup results.

---

# 5. Performance Metrics

| Metric | Value |
| --- | --- |
| Throughput | `472.080` QPS |
| Total Runtime | `63.548 sec` |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |

No policy or coverage failures were observed in this 30k run.

---

# 6. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed |
| --- | ---: | ---: | ---: |
| full_policy | 100.00% | 100.00% | 0 |
| no_hierarchy | 100.00% | 100.00% | 0 |
| no_repair | 100.00% | 100.00% | 0 |
| no_nearby | 100.00% | 100.00% | 0 |
| osm_only | 100.00% | 100.00% | 0 |

---

# 7. Layer Contribution Analysis

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 1 |
| Total vs OSM-only | 1 |

## Interpretation

* JP runtime policy for this release requires hierarchy support, but no pass/fail rescue delta was observed in this run.
* Nearby fallback provided a minimal, bounded contribution (`1` sample delta).
* Full policy and OSM-only outcomes are effectively equivalent at this sample size for pass/fail outcomes.

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[3,4,7]` | 27,012 |
| `[]` | 2,980 |
| `[3,4]` | 8 |

Empty-shape outcomes correspond to:

* `empty_shape`: 2,979
* `offshore`: 1

## 8.2 Node Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | 27,020 |
| `admin_tree_name` | 20,877 |

### Source Mix

| Mix | Count |
| --- | ---: |
| `admin_tree_name|polygon` | 20,877 |
| `polygon` | 6,143 |
| `none` | 2,980 |

---

# 9. Policy Reason Distribution

| Reason | Count |
| --- | ---: |
| `shape_status_map` | 27,020 |
| `empty_shape` | 2,979 |
| `offshore` | 1 |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `47`
* Total level-4 hits (all samples): `27,020`
* Total level-4 hits (inside samples): `27,000`

Hit distribution reflects area-based random sampling.

## 10.1 Top-10 Level-4 Hit Rates

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| --- | ---: | ---: | ---: | ---: |
| 北海道 | 6,143 | 20.48% | 6,138 | 22.73% |
| 岩手県 | 1,088 | 3.63% | 1,088 | 4.03% |
| 福島県 | 975 | 3.25% | 974 | 3.61% |
| 長野県 | 933 | 3.11% | 933 | 3.46% |
| 新潟県 | 913 | 3.04% | 910 | 3.37% |
| 秋田県 | 905 | 3.02% | 905 | 3.35% |
| 岐阜県 | 759 | 2.53% | 759 | 2.81% |
| 青森県 | 694 | 2.31% | 694 | 2.57% |
| 山形県 | 677 | 2.26% | 677 | 2.51% |
| 広島県 | 612 | 2.04% | 612 | 2.27% |

Additional verification (name variant check):

* `沖縄県` was hit (`126` all-sample hits, `125` inside-sample hits).

---

# 11. OSM Semantic Interpretation (Cadis Perspective)

Japan administrative polygons in OSM frequently include maritime extensions along coastal regions.

Under the 30k stress test:

* Structural non-empty outcomes (`27,020`) slightly exceed expected inside-land labels (`27,000`).
* Only `1` sample was classified as `offshore`.
* Several sea-surface samples were structurally contained within level-4 polygons.

This indicates that:

* OSM administrative boundaries in Japan extend into territorial sea areas.
* Cadis strictly respects original OSM polygon geometry.
* Cadis does not clip or reinterpret maritime extensions as non-land.

Therefore:

The effective “land coverage” under Cadis equals OSM administrative polygon coverage, not Natural Earth land mask.

This is a dataset-semantic property, not a runtime policy side effect.

---

# 12. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed.
* Outside-labeled samples predominantly resolved to empty-shape outcomes.
* No evidence was observed that optional policy effects caused cross-border escalation.

This run supports strict boundary containment within the JP dataset.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `5ff06d4d5aafb8b5bf958cbcc45ffb42013db285`
- cadis version:
  `v0.4.6`

The dataset package was generated from a clean working tree.
No local modifications were present at release time.

---

# 14. Conclusion

The `jp.admin v1.0.2` dataset demonstrates:

* Stable performance under 30k stress sampling
* Full policy and coverage pass (`100%`)
* Strong hierarchy usage footprint without pass/fail dependency in this run
* Minimal nearby fallback contribution
* Broad level-4 coverage across 47 units (including Okinawa)
* Strict boundary containment behavior in this run

This JP release candidate is structurally consistent with its declared runtime policy and is suitable for promotion to formal release evaluation review.
