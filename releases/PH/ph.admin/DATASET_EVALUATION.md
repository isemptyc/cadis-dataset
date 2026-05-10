# PH_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ph.admin`
Version: `v1.0.0`
Country: `PH`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ph.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the selected Philippines dataset scope
* Quantifies structural completeness under randomized lookup stress testing
* Documents deterministic policy effects
* Validates boundary isolation against the declared evaluation boundary
* Provides reproducible integrity metrics for release review

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value         |
| ----------------------- | ------------- |
| Dataset ID              | `ph.admin`    |
| Dataset Version         | `v1.0.0`      |
| Country                 | `PH`          |
| Country Name            | `Philippines` |
| Policy Version          | `1.0`         |
| Hierarchy Required      | `True`        |
| Repair Required         | `False`       |
| Runtime Policy Detected | `True`        |

## 2.1 Scope

| Field                    | Value |
| ------------------------ | ----- |
| Dataset scope            | `OSM admin-level-4 coverage union` |
| Boundary builder         | `build_ph_boundaries.py` |
| Build boundary JSON      | `ph_country.json` |
| Evaluation boundary JSON | `ph_country.json` |
| Selection rule           | Union all extracted Philippines `admin_level=4` province polygons after Natural Earth PH admin-0 filtering |
| Scoped boundary bbox     | `[117.9666667, 4.4545677, 126.7386324, 19.708072]` |
| Province polygon count   | `70` |

The build and evaluation used the same scoped boundary. Natural Earth components not covered by extracted OSM level-4 province polygons are outside this `v1.0.0` evaluation scope.

---

# 3. Test Methodology

## 3.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside points: `9,000`
* Outside points: `1,000`
* Lookup mode: `runtime`
* Dataset dir: `PH/ph.admin/v1.0.0`

The test intentionally injects outside-boundary samples to validate rejection behavior, offshore classification, and policy-layer containment.

---

# 4. Performance Metrics

| Metric                    | Value          |
| ------------------------- | -------------- |
| Throughput                | `12328.310` QPS |
| Total Runtime             | `0.811 sec`    |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |

No policy, coverage, or inside-boundary failures were observed.

---

# 5. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (`ok`/`partial`/`failed`) |
| ------------ | --------- | ---------------- | ------ | ------------- | -------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 253 / 8,761 / 986                |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 253 / 8,761 / 986                |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 253 / 8,761 / 986                |
| no_nearby    | 100.00%   | 100.00%          | 0      | 0             | 241 / 8,759 / 1,000              |
| osm_only     | 100.00%   | 100.00%          | 0      | 0             | 241 / 8,759 / 1,000              |

---

# 6. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 14              |
| Total vs OSM-only | 14              |

Direct observed node sources:

| Source  | Count |
| ------- | ----- |
| polygon | 9,000 |
| nearby  | 2     |

Hierarchy is still required by policy because the dataset ships a deterministic hierarchy layer, but this sampled run did not need hierarchy supplementation to pass.

---

# 7. Structural Distribution

## 7.1 Build-Time Hierarchy

| Level | Label                   | Count |
| ----- | ----------------------- | ----: |
| 4     | province                | 70    |
| 6     | city/municipality       | 688   |
| 10    | barangay                | 2,553 |

Build-time hierarchy edges: `3,170`
Unresolved hierarchy edges: `0`

## 7.2 Lookup Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[4,6]`   | 4,986 |
| `[4]`     | 3,771 |
| `[]`      | 998   |
| `[4,6,10]`| 241   |
| `[4,10]`  | 4     |

Empty shapes correspond to expected outside-boundary or offshore outcomes:

| Reason             | Count |
| ------------------ | ----: |
| `shape_status_map` | 9,002 |
| `empty_shape`      | 986   |
| `offshore`         | 12    |

---

# 8. Level-4 Coverage

* Unique level-4 provinces hit: `70`
* Total level-4 hits: `9,002`
* Total level-4 hits from inside samples: `9,000`

Top sampled level-4 units:

| Level-4 Unit          | Hits | Hit Rate (All Samples) | Hits (Inside Samples) |
| --------------------- | ---: | ----------------------: | --------------------: |
| Tawi-Tawi             | 443  | 4.43%                   | 443 |
| Cagayan               | 424  | 4.24%                   | 424 |
| Quezon                | 375  | 3.75%                   | 375 |
| Sulu                  | 372  | 3.72%                   | 372 |
| Zamboanga del Norte   | 333  | 3.33%                   | 333 |
| Bohol                 | 267  | 2.67%                   | 266 |
| Masbate               | 266  | 2.66%                   | 266 |
| Bukidnon              | 249  | 2.49%                   | 249 |
| Davao Oriental        | 242  | 2.42%                   | 242 |
| Occidental Mindoro    | 239  | 2.39%                   | 239 |

Hit distribution reflects uniform area sampling, not population-weighted coverage.

---

# 9. Boundary Isolation Validation

Under stress testing with 10% forced outside-boundary samples:

* No inside-boundary coverage failure was observed.
* No policy failure was observed.
* Empty-shape and offshore outcomes were limited to expected outside-boundary behavior.
* No evidence was observed that hierarchy or nearby layers created cross-boundary escalation in this run.

---

# 10. Structural Observations

1. The selected PH engine uses levels `4`, `6`, and `10`: province, city/municipality, and barangay.
2. OSM levels `5` and `7` were excluded from runtime policy because probe output showed they primarily contain district-style structures rather than the core postal/administrative stack.
3. Level-10 barangay coverage exists but is sparse relative to level-6 and level-4 coverage in area-weighted random sampling.
4. The dataset produced 100% inside-boundary pass rate under the scoped boundary.
5. Nearby fallback was minimal and bounded.
6. No repair layer was required.

---

# 11. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `54fffee8b041afa0d85734ff6fa2904bb45a2a29`
- Cadis version:
  `0.8.18`
- OSM source:
  `geofabrik:asia/philippines`
- OSM replication timestamp:
  `2026-05-09T20:21:02Z`
- OSM file SHA256:
  `c1010249014a3d9b92dce18e7e1205a9b28124ccb40e2555f167a6e06d97d1ee`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.
Cadis support/version changes and release-script changes were local at evaluation time.

---

# 12. Conclusion

The `ph.admin v1.0.0` staged dataset demonstrates:

* 100% overall, policy, and inside-boundary pass rates in the 10,000-sample runtime evaluation
* Deterministic runtime policy for the selected `4/6/10` administrative stack
* Full sampled coverage for the declared scoped boundary
* Strict boundary behavior under mixed inside/outside stress testing
* Minimal bounded nearby fallback
* No need for a repair layer in this evaluation run
