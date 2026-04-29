# FR_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `fr.admin`
Version: `v1.0.1`
Country: `FR`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity
evaluation of the staged `fr.admin v1.0.1` dataset under Cadis Runtime.

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
| Dataset Version | `v1.0.1` |
| Country | `FR` |
| Country Name | `France` |
| Policy Version | `1.0` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |

---

# 3. Test Methodology

## 3.1 Sampling Strategy

* Total samples: `100,000`
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

Boundary selection rule:

* keep Natural Earth `FR` multipolygon parts intersecting the
  metropolitan France/Corsica bbox `[-6.0, 41.0, 10.0, 52.0]`

Evaluation boundary bbox:

* `[-5.132801886999914, 41.36591217700004, 9.559580925000091, 51.08754088371883]`

Generated scoped boundary polygon count:

* `10`

Major excluded components:

* French overseas territories outside the European France extract, including
  French Guiana, Reunion, Martinique, Guadeloupe, Mayotte, and other overseas
  components represented in the full Natural Earth `FR` country geometry

## 3.2 Observed Distribution

| Category | Count |
| --- | ---: |
| Sample labels: expected inside points | 90,000 |
| Sample labels: expected outside points | 10,000 |
| Structural non-empty outcomes | 90,014 |
| Empty-shape outcomes (`[]`) | 9,986 |
| Offshore outcomes (subset of `[]`) | 94 |

Outside-labeled samples are intentional and do not indicate coverage gaps.

---

# 4. Performance Metrics

| Metric | Value |
| --- | --- |
| Throughput | `4327.550` QPS |
| Total Runtime | `23.108 sec` |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |

This staged run shows no observed policy failures and no inside-coverage misses.

---

# 5. Size Reduction

The `v1.0.1` build applies topology-preserving simplification to level-8
commune geometries while preserving the France dataset contract.

| Artifact | `v1.0.0` | `v1.0.1` | Reduction |
| --- | ---: | ---: | ---: |
| Dataset package | `79,555,239` bytes | `24,273,622` bytes | `69.49%` |
| `geometry.ffsf` | `78,899,060` bytes | `23,613,488` bytes | `70.07%` |

The reduction is concentrated in the geometry layer. Hierarchy and runtime
policy semantics are unchanged.

---

# 6. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| --- | ---: | ---: | ---: | ---: |
| `full_policy` | 100.00% | 100.00% | 0 | 0 |
| `no_hierarchy` | 100.00% | 100.00% | 0 | 0 |
| `no_repair` | 100.00% | 100.00% | 0 | 0 |
| `no_nearby` | 99.75% | 99.72% | 248 | 248 |
| `osm_only` | 99.75% | 99.72% | 248 | 248 |

## Interpretation

* hierarchy supplementation is present but not required for sampled correctness
  in this run
* repair is not active and not needed
* nearby fallback is the only layer that materially rescues failed inside points
* nearby rescue appears bounded and deterministic rather than broad or noisy

---

# 7. Layer Contribution Analysis

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 351 |
| Total vs OSM-only | 351 |

Observed layer usage:

| Usage Type | Count |
| --- | ---: |
| Hierarchy used samples | 0 |
| Repair used samples | 0 |
| Nearby used samples | 257 |
| Polygon used samples | 89,757 |
| Offshore samples | 94 |

Interpretation:

* nearby rescue is the primary France policy contribution in this run
* hierarchy did not contribute any sampled runtime lookup result
* no repair activity was observed

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[4,6,8]` | 88,930 |
| `[]` | 9,986 |
| `[4,8]` | 718 |
| `[4]` | 185 |
| `[4,6]` | 178 |
| `[6,8]` | 2 |
| `[8]` | 1 |

## 8.2 Node Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | 89,757 |
| `nearby` | 257 |
| `admin_tree_id` | 100 |

### Source Mix

| Mix | Count |
| --- | ---: |
| `polygon` | 89,657 |
| `__none__` | 9,986 |
| `nearby` | 257 |
| `admin_tree_id|polygon` | 100 |

## 8.3 Policy Reason Distribution

| Reason | Count |
| --- | ---: |
| `shape_status_map` | 90,014 |
| `empty_shape` | 9,892 |
| `offshore` | 94 |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `13`
* Total level-4 hits (all samples): `90,011`
* Total level-4 hits (inside samples): `89,996`

The staged dataset exposes the intended metropolitan France regional layer
consistently across the sampled inside points.

## 9.1 Level-4 Hit Rates

| Region | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| --- | ---: | ---: | ---: | ---: |
| `Nouvelle-Aquitaine` | 13,515 | 13.51% | 13,514 | 15.02% |
| `Occitanie` | 11,650 | 11.65% | 11,649 | 12.94% |
| `Auvergne-Rhône-Alpes` | 11,386 | 11.39% | 11,384 | 12.65% |
| `Grand Est` | 9,767 | 9.77% | 9,765 | 10.85% |
| `Bourgogne-Franche-Comté` | 7,936 | 7.94% | 7,933 | 8.81% |
| `Centre-Val de Loire` | 6,577 | 6.58% | 6,577 | 7.31% |
| `Hauts-de-France` | 5,625 | 5.62% | 5,623 | 6.25% |
| `Pays de la Loire` | 5,390 | 5.39% | 5,390 | 5.99% |
| `Normandie` | 5,132 | 5.13% | 5,132 | 5.70% |
| `Provence-Alpes-Côte d'Azur` | 4,984 | 4.98% | 4,981 | 5.53% |
| `Bretagne` | 4,640 | 4.64% | 4,639 | 5.15% |
| `Île-de-France` | 2,055 | 2.05% | 2,055 | 2.28% |
| `Corse` | 1,354 | 1.35% | 1,354 | 1.50% |

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced outside-boundary samples:

* no evaluation failures were observed
* empty-shape and offshore outcomes dominated expected outside samples
* no evidence was observed that hierarchy or nearby fallback caused cross-border
  escalation

The full Natural Earth `FR` multipolygon includes overseas territories such as
French Guiana, Reunion, and Martinique, while the source extract is
`geofabrik:europe/France`. The accepted staged evaluation therefore uses the
metropolitan-France/Corsica boundary that matches the dataset source scope.

---

# 11. Structural Observations

1. Geometry integrity is high under the selected metropolitan source scope.
2. Parent linkage incompleteness is low in sampled runtime behavior.
3. Hierarchy supplementation is deterministic but did not appear in sampled
   lookup outputs for this run.
4. Nearby fallback is bounded and contributes the only observed rescue delta.
5. No repair layer is required for this staged corpus.
6. Dataset scope must remain explicit: this build does not represent French
   overseas territories.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `85dafbe5928f34dd185cfbfc1a9a59bfd4cbf232`
- Cadis version:
  `0.8.11`
- OSM snapshot:
  `geofabrik:europe/France`
- OSM replication timestamp:
  `2025-10-18T20:20:55Z`

The dataset package was generated from a clean `cadis_dataset_engine` working
tree.

---

# 13. Conclusion

The staged `fr.admin v1.0.1` dataset demonstrates:

* high geometric integrity for metropolitan France plus Corsica
* full sampled inside-boundary coverage under policy mode
* bounded nearby fallback behavior
* no need for repair-layer activation in this staged run
* strict cross-border isolation in the sampled corpus

The dataset is ready for publish review under the metropolitan-France source
scope.
