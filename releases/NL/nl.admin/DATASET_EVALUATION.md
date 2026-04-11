# NL_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `nl.admin`
Version: `v1.0.0`
Country: `NL`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity
evaluation of the staged `nl.admin v1.0.0` dataset under Cadis Runtime.

This report:

* describes the observed Netherlands runtime structure
* quantifies policy-layer effects
* validates boundary isolation under stress testing
* records reproducible staging metrics before publish

This is a staged evaluation report, not a published release note.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `nl.admin` |
| Dataset Version | `v1.0.0` |
| Country | `NL` |
| Country Name | `Netherlands` |
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

For Netherlands evaluation, the sampling boundary used the Europe-only polygon
derived from Natural Earth. This avoids sampling Caribbean Netherlands while the
OSM source extract is `geofabrik:europe/Netherlands`.

Evaluation boundary bbox:

* `[3.3494146740001156, 50.74754954000004, 7.198505900000043, 53.55809153900003]`

## 3.2 Observed Distribution

| Category | Count |
| --- | ---: |
| Sample labels: expected inside points | 9,000 |
| Sample labels: expected outside points | 1,000 |
| Structural non-empty outcomes | 9,015 |
| Empty-shape outcomes (`[]`) | 985 |
| Offshore outcomes (subset of `[]`) | 32 |

Outside-labeled samples are intentional and do not indicate coverage gaps.

---

# 4. Performance Metrics

| Metric | Value |
| --- | --- |
| Throughput | `1004.350` QPS |
| Total Runtime | `9.957 sec` |
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
| `no_nearby` | 99.37% | 99.30% | 63 | 63 |
| `osm_only` | 99.37% | 99.30% | 63 | 63 |

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
| Nearby | 98 |
| Total vs OSM-only | 98 |

Observed layer usage:

| Usage Type | Count |
| --- | ---: |
| Hierarchy used samples | 20 |
| Repair used samples | 0 |
| Nearby used samples | 66 |
| Polygon used samples | 8,949 |
| Offshore samples | 32 |

Interpretation:

* nearby rescue is the primary Netherlands policy contribution in this run
* hierarchy appears as a low-frequency supplement but did not change pass/fail
  outcomes in the sampled corpus
* no repair activity was observed

---

# 7. Structural Distribution

## 7.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[4,8]` | 8,899 |
| `[]` | 985 |
| `[4]` | 116 |

## 7.2 Node Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | 8,949 |
| `nearby` | 66 |
| `admin_tree_name` | 20 |

### Source Mix

| Mix | Count |
| --- | ---: |
| `polygon` | 8,929 |
| `__none__` | 985 |
| `nearby` | 66 |
| `admin_tree_name|polygon` | 20 |

## 7.3 Policy Reason Distribution

| Reason | Count |
| --- | ---: |
| `shape_status_map` | 9,015 |
| `empty_shape` | 953 |
| `offshore` | 32 |

---

# 8. Level-4 Coverage

* Unique level-4 units hit: `12`
* Total level-4 hits (all samples): `9,015`
* Total level-4 hits (inside samples): `9,000`

The staged dataset exposes the intended province layer consistently across the
sampled inside points.

## 8.1 Level-4 Hit Rates

| Province | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| --- | ---: | ---: | ---: | ---: |
| `Gelderland` | 1,220 | 12.20% | 1,220 | 13.56% |
| `Noord-Brabant` | 1,211 | 12.11% | 1,211 | 13.46% |
| `Fryslân` | 1,031 | 10.31% | 1,025 | 11.39% |
| `Noord-Holland` | 853 | 8.53% | 852 | 9.47% |
| `Overijssel` | 822 | 8.22% | 822 | 9.13% |
| `Zuid-Holland` | 734 | 7.34% | 734 | 8.16% |
| `Drenthe` | 650 | 6.50% | 650 | 7.22% |
| `Groningen` | 563 | 5.63% | 562 | 6.24% |
| `Flevoland` | 540 | 5.40% | 540 | 6.00% |
| `Limburg` | 520 | 5.20% | 517 | 5.74% |
| `Zeeland` | 494 | 4.94% | 490 | 5.44% |
| `Utrecht` | 377 | 3.77% | 377 | 4.19% |

---

# 9. Boundary Isolation Validation

Under stress testing with 10% forced outside-boundary samples:

* no evaluation failures were observed
* empty-shape and offshore outcomes dominated expected outside samples
* no evidence was observed that hierarchy or nearby fallback caused cross-border
  escalation

This confirms strict staged containment for the Europe-only Netherlands source
boundary used in evaluation.

---

# 10. Structural Observations

1. The runtime spine is stable at province plus municipality (`[4,8]`).
2. Nearby fallback is useful but bounded; it rescues a small minority of points.
3. Hierarchy support exists and is low-frequency in the sampled run.
4. No repair layer is needed for the current Netherlands dataset.
5. The staged dataset achieves full inside-land coverage in this evaluation.
6. The engine excludes the Belgian enclave municipality `Baarle-Hertog`, which
   otherwise survives the raw source filter.

---

# 11. Reproducibility

This staged dataset evaluation used:

* cadis-dataset-engine commit:
  `8c1d57c2ca19495ee4bccdcf3da8feab7b597ac0`
* local Cadis version:
  `0.4.9`

This was evaluated from a staged local build. The Cadis support changes are
still local and not yet published.

---

# 12. Conclusion

`nl.admin v1.0.0` passes staged evaluation and is promotion-ready from the
observed runtime perspective.

The current evidence supports moving to the publish step, followed by Cadis
release finalization, provided you are satisfied with the staged report and want
to promote the dataset.
