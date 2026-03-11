# GB_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `gb.admin`
Version: `v1.0.0`
Country: `GB`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `gb.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the condition of the packaged dataset
* Quantifies structural completeness under runtime policy
* Documents deterministic policy effects
* Validates boundary isolation under stress testing
* Provides reproducible integrity metrics

OSM data is not incorrect.
Observed anomalies reflect structural incompleteness or expected outside-boundary behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value        |
| ----------------------- | ------------ |
| Dataset ID              | `gb.admin`   |
| Dataset Version         | `v1.0.0`     |
| Country                 | `GB`         |
| Policy Version          | `1.0`        |
| Hierarchy Required      | `True`       |
| Repair Required         | `False`      |
| Runtime Policy Detected | `True`       |

---

# 3. Test Methodology

## 3.1 Sampling Strategy

* Total samples: `30,000`
* Sampling mode: mixed inside/outside stress testing
* Inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Lookup mode: `runtime`

The test intentionally injects ~10% out-of-country points to validate:

* Boundary rejection behavior
* Empty-shape handling
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over land area, not population-weighted.

This GB run used an explicit GB boundary JSON derived from Natural Earth `ISO_A2=GB` to avoid the broader `POSTAL=GB` match in the raw shapefile sampler path.

---

## 3.2 Observed Distribution

| Category                               | Count  |
| -------------------------------------- | ------ |
| Sample labels: expected inside points  | 27,000 |
| Sample labels: expected outside points | 3,000  |
| Structural non-empty outcomes          | 27,053 |
| Empty-shape outcomes (`[]`)            | 2,947  |
| Offshore outcomes (subset of `[]`)     | 39     |

Outside-labeled samples are intentional and do not indicate dataset coverage gaps.
`expected inside/outside` comes from the sampling geometry labels; structural outcomes come from Cadis runtime evaluation.

---

# 4. Performance Metrics

| Metric                    | Value         |
| ------------------------- | ------------- |
| Throughput                | `709.970` QPS |
| Total Runtime             | `42.255 sec`  |
| Overall Pass Rate         | `100.00%`     |
| Inside Coverage Pass Rate | `100.00%`     |
| Policy Pass Rate          | `100.00%`     |

This run confirms no policy failures and no inside-boundary coverage failures.

---

# 5. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed |
| ------------ | --------- | ---------------- | ------ |
| full_policy  | 100.00%   | 100.00%          | 0      |
| no_hierarchy | 100.00%   | 100.00%          | 0      |
| no_repair    | 100.00%   | 100.00%          | 0      |
| no_nearby    | 99.97%    | 99.96%           | 10     |
| osm_only     | 99.97%    | 99.96%           | 10     |

---

# 6. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 52              |
| Total vs OSM-only | 52              |

## Interpretation

* OSM-only success rate: `99.97%`
* Hierarchy supplementation was not needed in this sampled run.
* No geometric repair operations were triggered.
* Nearby fallback resolved a small but measurable set of edge cases.
* Structural coverage is already near-complete from polygon-derived results alone.

Observed failures under `no_nearby` and `osm_only` mode are minimal and bounded.

---

# 7. Structural Distribution

## 7.1 Shape Distribution

| Shape     | Count |
| --------- | ----- |
| [4,6]     | 15,037 |
| [4,6,8]   | 4,287  |
| []        | 2,947  |
| [4,5,6,8] | 2,519  |
| [4,5,6]   | 2,403  |
| [4]       | 1,848  |
| [4,5,8]   | 952    |
| [4,8]     | 5      |
| [4,5]     | 2      |

Empty shapes correspond to:

* `empty_shape`: 2,908
* `offshore`: 39

---

## 7.2 Node Source Distribution

| Source  | Count |
| ------- | ----- |
| polygon | 27,040 |
| nearby  | 13     |

### Source Mix

| Mix     | Count |
| ------- | ----- |
| polygon | 27,040 |
| none    | 2,947  |
| nearby  | 13     |

---

# 8. Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----- |
| shape_status_map | 27,053 |
| empty_shape      | 2,908  |
| offshore         | 39     |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `4`
* Total level-4 hits (all samples): `27,053`
* Total level-4 hits (inside samples): `27,000`

Hit distribution reflects uniform land-area sampling rather than population density.

## 9.1 Level-4 Hit Rates

| Level-4 Unit      | Hits  | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ----------------- | ----- | ---------------------- | --------------------- | ------------------------- |
| England           | 13,997 | 46.66%                 | 13,983                | 51.79%                    |
| Scotland          | 9,182  | 30.61%                 | 9,146                 | 33.87%                    |
| Wales             | 2,244  | 7.48%                  | 2,241                 | 8.30%                     |
| Northern Ireland  | 1,630  | 5.43%                  | 1,630                 | 6.04%                     |

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Expected outside-labeled samples resolved to empty shapes or offshore outcomes as intended.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* Nearby fallback remained tightly bounded at `13` direct uses and `52` scenario-rescued samples.

This confirms strict boundary containment within the GB dataset in this runtime evaluation.

---

# 11. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. Polygon-derived results account for nearly all successful classifications.
3. Hierarchy supplementation was not required in this sampled run.
4. Nearby fallback usage is low and well-bounded.
5. Dataset achieves full inside-boundary coverage under policy mode.
6. Boundary rejection behavior is strict and leak-free in this evaluation.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `d50652ade1d6c05a77b04daa3f935ac686132a55`
- cadis commit:
  `56cec3a0579de88950d3a25d22f7b4f5741fe91c`

The dataset package was generated from a clean working tree.
No local modifications were present at release time.

---

# 13. Conclusion

The `gb.admin v1.0.0` dataset demonstrates:

* High geometric integrity
* Near-complete structural coverage from polygon data alone
* No observed need for hierarchy supplementation in this sampled run
* Low nearby fallback usage
* Full inside-boundary coverage under policy
* Strict cross-border isolation

OSM data is not incorrect.
It is occasionally incomplete.

Cadis does not modify geography.
It enforces structural determinism and boundary integrity.
