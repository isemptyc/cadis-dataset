# DK_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `dk.admin`  
Version: `v1.0.0`  
Country: `DK`  
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `dk.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the condition of the source OSM data
* Quantifies structural completeness and sparse outcomes
* Documents deterministic policy effects
* Validates Denmark boundary isolation under stress testing
* Records reproducible runtime metrics for release review

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `dk.admin` |
| Dataset Version | `v1.0.0` |
| Country | `DK` |
| Country Name | `Denmark` |
| Policy Version | `1.0` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |

---

# 3. Test Methodology

## 3.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Boundary source: `tmp/denmark_country_from_ne.json`

The test intentionally injects approximately 10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over area, not population-weighted.

## 3.2 Observed Distribution

| Category | Count |
| --- | ---: |
| Sample labels: expected inside points | 9,000 |
| Sample labels: expected outside points | 1,000 |
| Structural non-empty outcomes | 9,006 |
| Empty-shape outcomes (`[]`) | 994 |
| Offshore outcomes (subset of `[]`) | 38 |

Outside-labeled samples are intentional and do not indicate Denmark coverage gaps.

---

# 4. Performance Metrics

| Metric | Value |
| --- | --- |
| Throughput | `1380.360` QPS |
| Total Runtime | `7.245 sec` |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |

This run shows no policy or coverage failures in the staged Denmark dataset.

---

# 5. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed |
| --- | ---: | ---: | ---: |
| `full_policy` | `100.00%` | `100.00%` | 0 |
| `no_hierarchy` | `100.00%` | `100.00%` | 0 |
| `no_repair` | `100.00%` | `100.00%` | 0 |
| `no_nearby` | `98.29%` | `98.10%` | 171 |
| `osm_only` | `98.29%` | `98.10%` | 171 |

## Interpretation

* Nearby fallback is the only layer that materially changes pass/fail outcomes in this run.
* Hierarchy improves completeness of some results but does not change pass rate.
* Repair is correctly inactive for this dataset.

---

# 6. Layer Contribution Analysis

| Layer | Value |
| --- | ---: |
| Hierarchy used samples | 47 |
| Repair used samples | 0 |
| Nearby used samples | 176 |
| Polygon used samples | 8,830 |
| Rescued by hierarchy | 0 |
| Rescued by repair | 0 |
| Rescued by nearby | 214 |
| Total rescued vs OSM-only | 214 |

## Interpretation

* `nearby used samples = 176` is direct runtime source usage.
* `rescued by nearby = 214` is the scenario-delta metric versus `osm_only`.
* These values are related but not identical measures.
* Denmark does not currently require a repair layer.

---

# 7. Structural Distribution

## 7.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[4,7]` | 8,956 |
| `[]` | 994 |
| `[4]` | 50 |

Empty shapes correspond to:

* `empty_shape`: 956
* `offshore`: 38

## 7.2 Node Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | 8,830 |
| `nearby` | 176 |
| `admin_tree_name` | 47 |

### Source Mix

| Mix | Count |
| --- | ---: |
| `polygon` | 8,783 |
| `admin_tree_name|polygon` | 47 |
| `nearby` | 176 |
| `__none__` | 994 |

## 7.3 Policy Reason Distribution

| Reason | Count |
| --- | ---: |
| `shape_status_map` | 9,006 |
| `empty_shape` | 956 |
| `offshore` | 38 |

---

# 8. Level-4 Coverage

* Unique level-4 units hit: `5`
* Total level-4 hits (all samples): `9,006`
* Total level-4 hits (inside samples): `9,000`

## 8.1 Level-4 Hit Rates

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| --- | ---: | ---: | ---: | ---: |
| `Region Midtjylland` | 2,799 | 27.99% | 2,797 | 31.08% |
| `Region Syddanmark` | 2,483 | 24.83% | 2,482 | 27.58% |
| `Region Nordjylland` | 1,737 | 17.37% | 1,736 | 19.29% |
| `Region Sjælland` | 1,469 | 14.69% | 1,467 | 16.30% |
| `Region Hovedstaden` | 518 | 5.18% | 518 | 5.76% |

This distribution is consistent with uniform area sampling across Denmark.

---

# 9. Administrative Spine Review

The Denmark engine currently exposes:

* Level `4`: region
* Level `7`: municipality

Observed build-stage hierarchy:

* `5` level-4 regions
* `98` level-7 municipalities

This matches the expected modern Denmark administrative spine and showed no foreign-leakage symptoms in the evaluated staged release.

---

# 10. A/B Check for Old-Era Country-Scope Note

An explicit A/B check was run before the official SOP evaluation:

* `A`: no export-time country-scope metadata
* `B`: export-time country-scope metadata from Natural Earth

Observed result:

* Runtime outcomes were identical in the seeded comparison corpus.
* `country_scope_flag` changed from `null` to `true` in metadata, but no behavioral difference appeared in runtime validation.

Conclusion for current Denmark onboarding:

* the old-era note was relevant enough to test
* current Denmark release behavior does not presently depend on export-time country-scope metadata
* keeping optional support is still useful for future diagnostics

---

# 11. Reproducibility

The staged Denmark dataset and the evaluation results are reproducible from the following inputs:

* `cadis_dataset_engine` commit:
  `cbe74efa03045504265b6a83205294c4b691936b`
* Cadis local version used for evaluation:
  `0.4.4`

Clean-tree status at generation time:

* `cadis_dataset_engine` was committed and clean for the official SOP staging run.
* `cadis` support for `DK` and the `0.4.4` version bump were present locally for evaluation, but the `cadis` repo had not yet been finalized as a release commit at the time this report was written.

Therefore:

* engine-side dataset generation is pinned to a stable commit
* runtime-side reproducibility currently depends on preserving the same local `cadis` support state until the `cadis` release commit is finalized

---

# 12. Conclusion

Release-readiness assessment for the staged Denmark dataset:

* Official SOP staging build: `PASS`
* 10,000-point runtime evaluation: `PASS`
* Boundary isolation under mixed inside/outside stress: `PASS`
* Deterministic nearby support: `PASS`
* Repair-layer dependency: `NONE`

Current recommendation:

* `dk.admin v1.0.0` is suitable to continue through the remaining release flow, subject to normal operator review before publish.
