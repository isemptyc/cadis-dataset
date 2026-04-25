# FR_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `fr.admin`
Version: `v1.0.0`
Country: `FR`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity
evaluation of the staged `fr.admin v1.0.0` dataset under Cadis Runtime.

This report:

* describes the observed France runtime structure
* quantifies policy-layer effects
* validates boundary isolation under stress testing
* records reproducible staging metrics before publish

This is a staged evaluation report, not a published release note.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `fr.admin` |
| Dataset Version | `v1.0.0` |
| Country | `FR` |
| Country Name | `France` |
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

The run intentionally injects about 10% outside-boundary points to validate:

* boundary rejection behavior
* offshore handling
* nearby fallback containment
* cross-border isolation

For France evaluation, the sampling boundary used a metropolitan-France plus
Corsica polygon derived from Natural Earth. This avoids sampling overseas
territories while the source extract is `geofabrik:europe/France`.

Evaluation boundary bbox:

* `[-5.132801886999914, 41.36591217700004, 9.559580925000091, 51.08754088371883]`

## 3.2 Observed Distribution

| Category | Count |
| --- | ---: |
| Sample labels: expected inside points | 9,000 |
| Sample labels: expected outside points | 1,000 |
| Structural non-empty outcomes | 9,002 |
| Empty-shape outcomes (`[]`) | 998 |
| Offshore outcomes (subset of `[]`) | 4 |

Outside-labeled samples are intentional and do not indicate coverage gaps.

---

# 4. Performance Metrics

| Metric | Value |
| --- | --- |
| Throughput | `169.780` QPS |
| Total Runtime | `58.898 sec` |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |

This staged run shows no observed policy failures and no inside-coverage misses.

---

# 5. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| --- | ---: | ---: | ---: | ---: |
| `full_policy` | 100.00% | 100.00% | 0 | 0 |
| `no_hierarchy` | 100.00% | 100.00% | 0 | 0 |
| `no_repair` | 100.00% | 100.00% | 0 | 0 |
| `no_nearby` | 99.76% | 99.73% | 24 | 24 |
| `osm_only` | 99.76% | 99.73% | 24 | 24 |

## Interpretation

* hierarchy supplementation is present but not required for sampled correctness
  in this run
* repair is not active and not needed
* nearby fallback is the only layer that materially rescues failed inside points
* nearby rescue appears bounded and deterministic rather than broad or noisy

---

# 6. Layer Contribution Analysis

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 29 |
| Total vs OSM-only | 29 |

Observed layer usage:

| Usage Type | Count |
| --- | ---: |
| Hierarchy used samples | 8 |
| Repair used samples | 0 |
| Nearby used samples | 25 |
| Polygon used samples | 8,977 |
| Offshore samples | 4 |

Interpretation:

* nearby rescue is the primary France policy contribution in this run
* hierarchy appears as a low-frequency supplement but did not change pass/fail
  outcomes in the sampled corpus
* no repair activity was observed

---

# 7. Structural Distribution

## 7.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[4,6,8]` | 8,909 |
| `[]` | 998 |
| `[4,8]` | 64 |
| `[4]` | 18 |
| `[4,6]` | 11 |

## 7.2 Node Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | 8,977 |
| `nearby` | 25 |
| `admin_tree_name` | 8 |

### Source Mix

| Mix | Count |
| --- | ---: |
| `polygon` | 8,969 |
| `__none__` | 998 |
| `nearby` | 25 |
| `admin_tree_name|polygon` | 8 |

## 7.3 Policy Reason Distribution

| Reason | Count |
| --- | ---: |
| `shape_status_map` | 9,002 |
| `empty_shape` | 994 |
| `offshore` | 4 |

---

# 8. Level-4 Coverage

* Unique level-4 units hit: `13`
* Total level-4 hits (all samples): `9,002`
* Total level-4 hits (inside samples): `9,000`

The staged dataset exposes the intended metropolitan France regional layer
consistently across the sampled inside points.

## 8.1 Level-4 Hit Rates

| Region | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| --- | ---: | ---: | ---: | ---: |
| `Nouvelle-Aquitaine` | 1,356 | 13.56% | 1,356 | 15.07% |
| `Auvergne-Rhône-Alpes` | 1,151 | 11.51% | 1,151 | 12.79% |
| `Occitanie` | 1,146 | 11.46% | 1,146 | 12.73% |
| `Grand Est` | 986 | 9.86% | 985 | 10.94% |
| `Bourgogne-Franche-Comté` | 753 | 7.53% | 753 | 8.37% |
| `Centre-Val de Loire` | 667 | 6.67% | 667 | 7.41% |
| `Hauts-de-France` | 568 | 5.68% | 567 | 6.30% |
| `Pays de la Loire` | 530 | 5.30% | 530 | 5.89% |
| `Provence-Alpes-Côte d'Azur` | 524 | 5.24% | 524 | 5.82% |
| `Normandie` | 508 | 5.08% | 508 | 5.64% |
| `Bretagne` | 476 | 4.76% | 476 | 5.29% |
| `Île-de-France` | 201 | 2.01% | 201 | 2.23% |
| `Corse` | 136 | 1.36% | 136 | 1.51% |

---

# 9. Boundary Isolation Validation

Under stress testing with 10% forced outside-boundary samples:

* no evaluation failures were observed
* empty-shape and offshore outcomes dominated expected outside samples
* no evidence was observed that hierarchy or nearby fallback caused cross-border
  escalation

The first evaluation using the full Natural Earth `FR` multipolygon failed
coverage because that boundary includes overseas territories such as French
Guiana, Réunion, and Martinique, while the source extract is
`geofabrik:europe/France`. The accepted staged evaluation therefore uses the
metropolitan-France/Corsica boundary that matches the dataset source scope.

---

# 10. Structural Observations

1. Geometry integrity is high under the selected metropolitan source scope.
2. Parent linkage incompleteness is low in sampled runtime behavior.
3. Hierarchy supplementation is deterministic and low frequency.
4. Nearby fallback is bounded and contributes the only observed rescue delta.
5. No repair layer is required for this staged corpus.
6. Dataset scope must remain explicit: this build does not represent French
   overseas territories.

---

# 11. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `23ff8a83aa3a27aa2908dbdc1089f7de62692ca9`
- Cadis version:
  `0.5.3`

The dataset package was generated from a clean `cadis_dataset_engine` working
tree.

---

# 12. Conclusion

The staged `fr.admin v1.0.0` dataset demonstrates:

* high geometric integrity for metropolitan France plus Corsica
* full sampled inside-boundary coverage under policy mode
* deterministic hierarchy supplementation
* bounded nearby fallback behavior
* no need for repair-layer activation in this staged run
* strict cross-border isolation in the sampled corpus

The dataset is ready for publish review under the metropolitan-France source
scope.
