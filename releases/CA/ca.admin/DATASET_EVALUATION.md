# CA_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ca.admin`
Version: `v1.0.0`
Country: `CA`
Scope: full Canada administrative coverage
Policy Version: `1.0`

---

# 1. Purpose

This document provides a behavioral and boundary-integrity evaluation of the `ca.admin v1.0.0` dataset under Cadis Runtime.

The evaluation quantifies runtime lookup behavior under mixed inside/outside stress testing, validates policy behavior, records level-4 province and territory coverage, and preserves reproducible artifact paths for the evaluated package.

Cadis does not modify geography. It reconstructs administrative topology from source data and enforces deterministic runtime policy.

---

# 2. Dataset Identity

| Field                   | Value      |
| ----------------------- | ---------- |
| Dataset ID              | `ca.admin` |
| Dataset Version         | `v1.0.0`   |
| Country                 | `CA`       |
| Country Name            | `Canada`   |
| Runtime Policy Detected | `True`     |
| Policy Version          | `1.0`      |
| Hierarchy Required      | `True`     |
| Repair Required         | `False`    |
| Allowed Shapes Count    | `31`       |
| Shape Status Count      | `31`       |
| Nearby Fallback Enabled | `True`     |

---

# 3. Test Methodology

## 3.1 Sampling Strategy

* Total samples: `400,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `360,000`
* Outside samples: `40,000`
* Inside ratio: `0.9`
* Outside ratio: `0.1`
* Lookup mode: `runtime`

The test intentionally injects 10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over land area and is not population-weighted.

## 3.2 Boundary BBox

| Coordinate | Value |
| ---------- | ----: |
| Min Lon    | `-141.00556393099993` |
| Min Lat    | `41.66908559200006` |
| Max Lon    | `-52.61660722599993` |
| Max Lat    | `83.11652252800008` |

---

# 4. Observed Runtime Distribution

| Category                         | Count |
| -------------------------------- | ----: |
| Total samples                    | `400,000` |
| Runtime `ok` status              | `360,909` |
| Runtime `partial` status         | `0` |
| Runtime `failed` status          | `39,091` |
| Structural non-empty outcomes    | `360,813` |
| Empty-shape outcomes (`[]`)      | `39,187` |
| Offshore outcomes                | `96` |
| HTTP 200 responses               | `400,000` |

The `failed` runtime status corresponds to expected out-of-country empty-shape behavior for outside stress samples. It is policy-valid in this evaluation.

---

# 5. Performance Metrics

| Metric                    | Value            |
| ------------------------- | ---------------- |
| Throughput                | `1209.960` QPS   |
| Total Runtime             | `330.591 sec`    |
| Overall Pass Rate         | `100.00%`        |
| Policy Pass Rate          | `100.00%`        |
| Coverage Pass Rate        | `100.00%`        |
| Inside Coverage Pass Rate | `100.00%`        |
| Passed Samples            | `400,000`        |
| Failed Samples            | `0`              |
| Inside Failed             | `0`              |
| Outside Policy Failed     | `0`              |

This run confirms full sampled inside-land coverage under the active runtime policy.

---

# 6. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (ok/partial/failed/unknown) |
| ------------ | --------: | ---------------: | -----: | ------------: | ----------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 360,909/0/39,091/0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 360,909/0/39,091/0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 360,909/0/39,091/0 |
| no_nearby    | 99.98%    | 99.98%           | 72     | 72            | 360,736/0/39,264/0 |
| osm_only     | 99.98%    | 99.98%           | 72     | 72            | 360,736/0/39,264/0 |

---

# 7. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------: |
| Hierarchy         | `0` |
| Repair            | `0` |
| Nearby            | `173` |
| Total vs OSM-only | `173` |

## Interpretation

* OSM-only success rate is already `99.98%` in this sampled run.
* The hierarchy layer was not required for sampled Canada lookups.
* The repair layer was not triggered.
* Nearby fallback rescued `173` samples by scenario delta and appeared directly as a node source for `77` samples.
* Nearby fallback remains bounded to edge/coastal behavior.

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape           | Count |
| --------------- | ----: |
| `[4,5]`         | `150,182` |
| `[4,6]`         | `67,048` |
| `[4,5,6,8]`     | `62,952` |
| `[]`            | `39,187` |
| `[4,5,6]`       | `29,635` |
| `[4,6,8]`       | `21,205` |
| `[4]`           | `18,965` |
| `[4,5,6,7]`     | `9,672` |
| `[4,5,8]`       | `664` |
| `[4,5,6,7,8]`   | `286` |
| `[4,8]`         | `187` |
| `[5,6,8]`       | `9` |
| `[6]`           | `4` |
| `[4,5,7]`       | `1` |
| `[5,6]`         | `1` |
| `[6,8]`         | `1` |
| `[8]`           | `1` |

## 8.2 Node Source Distribution

| Source          | Count |
| --------------- | ----: |
| `polygon`       | `360,736` |
| `nearby`        | `77` |
| `admin_tree_id` | `5` |

## 8.3 Source Mix

| Mix                       | Count |
| ------------------------- | ----: |
| `polygon`                 | `360,731` |
| `__none__`                | `39,187` |
| `nearby`                  | `77` |
| `admin_tree_id|polygon`   | `5` |

## 8.4 Policy Reason Distribution

| Reason             | Count |
| ------------------ | ----: |
| `shape_status_map` | `360,813` |
| `empty_shape`      | `39,091` |
| `offshore`         | `96` |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `13`
* Total level-4 hits, all samples: `360,797`
* Total level-4 hits, inside samples: `359,983`

The sampled level-4 distribution reflects uniform land-area sampling. Northern provinces and territories dominate because the sampling is area-weighted, not population-weighted.

## 9.1 Level-4 Hit Rates

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ---------------------: | --------------------: | ------------------------: |
| `Nunavut` | 106,266 | 26.57% | 105,623 | 29.34% |
| `Northwest Territories` | 60,933 | 15.23% | 60,849 | 16.90% |
| `Quebec` | 43,368 | 10.84% | 43,336 | 12.04% |
| `Ontario` | 29,532 | 7.38% | 29,532 | 8.20% |
| `British Columbia` | 28,268 | 7.07% | 28,259 | 7.85% |
| `Alberta` | 20,399 | 5.10% | 20,399 | 5.67% |
| `Manitoba` | 20,015 | 5.00% | 20,014 | 5.56% |
| `Saskatchewan` | 19,607 | 4.90% | 19,607 | 5.45% |
| `Yukon` | 19,093 | 4.77% | 19,092 | 5.30% |
| `Newfoundland and Labrador` | 11,138 | 2.78% | 11,110 | 3.09% |
| `New Brunswick` | 1,123 | 0.28% | 1,117 | 0.31% |
| `Nova Scotia` | 987 | 0.25% | 978 | 0.27% |
| `Prince Edward Island` | 68 | 0.02% | 67 | 0.02% |

All 13 Canada province/territory level-4 units were observed in the 400,000-sample run.

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No sampled boundary leak was observed.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* Outside policy failures: `0`
* Inside coverage failures: `0`

This confirms strict sampled boundary containment within the `CA` dataset.

---

# 11. Structural Observations

1. Geometry coverage is strong under the active runtime policy.
2. No repair layer activation was observed.
3. Hierarchy supplementation was not required for sampled lookups.
4. Nearby fallback is small and bounded: `77` direct nearby-source samples, `173` scenario-delta rescues.
5. Canada exhibits irregular administrative depth, reflected in diverse valid shapes such as `[4,5]`, `[4,6]`, `[4,5,6,8]`, `[4,5,6]`, and `[4,5,6,7]`.
6. This report treats the irregular shape distribution as dataset truth rather than forcing a perfectly regular hierarchy.

---

# 12. Reproducibility

Build/runtime versions:

* cadis-dataset-engine commit:
  `ff4ab52f75d6319f9008d5067386dd0ada67eb57`
* Cadis version:
  `0.6.8`

The dataset can be reproduced from the public PBF extracts without checked-in pickle files. The `_stitch_cache` directory is regenerated when absent.

---

# 13. Evaluation Verdict

`ca.admin v1.0.0` passes the 400,000-sample Cadis Runtime evaluation.

| Verdict Field                 | Result |
| ----------------------------- | ------ |
| Overall evaluation            | `PASS` |
| Runtime policy detected       | `PASS` |
| Policy pass rate              | `100.00%` |
| Inside coverage pass rate     | `100.00%` |
| Boundary isolation            | `PASS` |
| Repair dependency             | `None observed` |
| Hierarchy rescue dependency   | `None observed` |
| Nearby dependency             | `Small and bounded` |

The dataset is suitable for release candidate review from the perspective of sampled runtime behavior and boundary containment.
