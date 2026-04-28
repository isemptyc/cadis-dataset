# AU_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `au.admin`
Version: `v1.0.0`
Country: `AU`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `au.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source OSM administrative model selected for Australia
* Documents the dataset scope used for build and evaluation
* Quantifies runtime lookup behavior under mixed inside/outside sampling
* Validates policy-layer containment and nearby fallback behavior
* Records reproducible integrity metrics for the release candidate

OSM data is not incorrect.
Observed sparse shapes reflect administrative coverage differences and parent-link incompleteness, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value      |
| ----------------------- | ---------- |
| Dataset ID              | `au.admin` |
| Dataset Version         | `v1.0.0`   |
| Country                 | `AU`       |
| Policy Version          | `1.0`      |
| Hierarchy Required      | `True`     |
| Repair Required         | `False`    |
| Runtime Policy Detected | `True`     |
| Cadis Version           | `0.6.9`    |

---

# 3. Dataset Scope

The Australia release uses a tracked scoped-boundary builder:

* Boundary source: Natural Earth admin-0 AU polygon plus OSM level-4 administrative relation geometry
* Deterministic selection rule: union OSM administrative relation areas at level 4 whose representative point is covered by the Natural Earth AU boundary

Included level-4 units:

* `Australian Capital Territory`
* `Jervis Bay Territory`
* `New South Wales`
* `Northern Territory`
* `Queensland`
* `South Australia`
* `Tasmania`
* `Victoria`
* `Western Australia`

Major external territory components recorded as excluded or not represented as first-class level-4 units in this initial scope:

* `Cocos (Keeling) Islands`
* `Christmas Island`
* `Heard Island and McDonald Islands`
* `Norfolk Island`

Scoped boundary bbox:

```text
[112.8656697, -55.1730541, 159.3390311, -9.0880125]
```

---

# 4. Test Methodology

## 4.1 Sampling Strategy

* Total samples: `200,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `180,000`
* Outside samples: `20,000`
* Lookup mode: Cadis runtime
* Evaluation workers: `1`

Single-worker evaluation was used because the current concurrent evaluator path under-counts batch results. The runtime batch path processed all 200,000 samples.

---

# 5. Performance Metrics

| Metric                    | Value          |
| ------------------------- | -------------- |
| Throughput                | `2171.530` QPS |
| Total Runtime             | `92.101 sec`   |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |
| HTTP/Runtime 200 Count    | `200,000`      |

No policy, coverage, or inside-boundary failures were observed.

---

# 6. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| ------------ | --------- | ---------------- | ------ | ------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             |
| no_nearby    | 99.98%    | 99.97%           | 46     | 46            |
| osm_only     | 99.98%    | 99.97%           | 46     | 46            |

---

# 7. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 94              |
| Total vs OSM-only | 94              |

## Interpretation

* Geometry and polygon coverage are strong for the sampled scope.
* Nearby fallback resolved a small number of coastal or boundary-adjacent samples.
* No explicit repair layer is required for this release candidate.
* Hierarchy artifacts are present and required by policy, but this sample did not require hierarchy supplementation to convert failures into successful outcomes.

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape       | Count |
| ----------- | ----- |
| `[4,6,9]`   | 169,873 |
| `[]`        | 19,992  |
| `[4,6]`     | 5,153   |
| `[4]`       | 2,733   |
| `[4,9]`     | 2,108   |
| `[4,7]`     | 51      |
| `[6,9]`     | 30      |
| `[4,6,7,9]` | 30      |
| `[9]`       | 20      |
| `[4,7,9]`   | 10      |

Empty shapes correspond to expected outside/offshore samples:

* `empty_shape`: 19,952
* `offshore`: 40

## 8.2 Node Source Distribution

| Source          | Count |
| --------------- | ----- |
| polygon         | 179,954 |
| nearby          | 54      |
| admin_tree_id   | 24      |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `9`
* Total level-4 hits: `179,958`
* Total inside level-4 hits: `179,950`

| Level-4 Unit                 | Hits  | Hit Rate | Inside Hits | Inside Hit Rate |
| ---------------------------- | ----- | -------- | ----------- | --------------- |
| Western Australia            | 58,266 | 29.13%   | 58,264     | 32.37%          |
| Queensland                   | 39,645 | 19.82%   | 39,641     | 22.02%          |
| Northern Territory           | 29,958 | 14.98%   | 29,956     | 16.64%          |
| South Australia              | 24,412 | 12.21%   | 24,412     | 13.56%          |
| New South Wales              | 19,008 | 9.50%    | 19,008     | 10.56%          |
| Victoria                     | 5,961  | 2.98%    | 5,961      | 3.31%           |
| Tasmania                     | 2,645  | 1.32%    | 2,645      | 1.47%           |
| Australian Capital Territory | 62     | 0.03%    | 62         | 0.03%           |
| Jervis Bay Territory         | 1      | 0.00%    | 1          | 0.00%           |

The 200,000-sample run hit `Jervis Bay Territory` once, which is expected for uniform land-area sampling because it is very small.

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed.
* Empty-shape and offshore outcomes accounted for expected outside samples.
* No evidence was observed that hierarchy or nearby layers created cross-boundary escalation.

This confirms strict containment within the declared AU scoped boundary.

---

# 11. Structural Observations

1. Australia’s useful OSM administrative model is level 4 states/territories, level 6 local government areas, level 7 ACT/district-like structures, and level 9 suburbs/localities.
2. The dominant successful shape is `[4,6,9]`.
3. Sparse valid shapes such as `[4]`, `[4,6]`, and `[4,9]` occur in real coverage and are handled by policy.
4. The repair layer is not required.
5. Nearby fallback is bounded and low-volume.
6. The scoped boundary builder is required to make the release scope explicit and reproducible.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit: `e69710d0e7af42e2c766da7cd09e8bacc66f0988`
* Cadis version prepared locally: `0.6.9`
* OSM source: `geofabrik:oceania/Australia`
* OSM file: `australia-260331.osm.pbf`
* OSM SHA256: `359d2933335054174d01d56d69c2ccfdb6ab4f86527dd991fca3c14b51d2ba50`
* OSM replication timestamp: `2026-03-31T20:21:06Z`

The engine repository was clean and committed before staged dataset generation.

---

# 13. Conclusion

The `au.admin v1.0.0` release candidate demonstrates:

* Full inside-boundary coverage under policy mode
* Strict outside-boundary isolation
* Strong polygon-based resolution for the declared Australia scope
* No need for explicit repair rules
* Low-volume nearby fallback for edge cases
* Reproducible scoped-boundary generation

Dataset quality is acceptable for the next SOP step: review, then run the official publish command only after operator approval.
