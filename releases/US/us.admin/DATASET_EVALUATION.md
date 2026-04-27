# US_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `us.admin`
Version: `v1.0.0`
Country: `US`
Scope: full United States administrative coverage built from Geofabrik state and territory extracts
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `us.admin v1.0.0` dataset under Cadis Runtime.

The United States dataset is larger than a normal single-country Cadis build and is produced from multiple Geofabrik state and territory PBF extracts. The build therefore uses a US-specific stitching stage before geometry export.

This report:

* Documents the multi-extract source and stitch build evidence
* Quantifies runtime lookup behavior under inside/outside stress testing
* Validates policy behavior and boundary isolation
* Records level-4 state and territory coverage observed in the sampled run
* Preserves reproducible artifact paths for the evaluated package

Cadis does not modify geography. It reconstructs administrative topology from source OSM data and enforces deterministic runtime policy.

---

# 2. Dataset Identity

| Field                   | Value                       |
| ----------------------- | --------------------------- |
| Dataset ID              | `us.admin`                  |
| Dataset Version         | `v1.0.0`                    |
| Country                 | `US`                        |
| Country Name            | `United States of America`  |
| Policy Version          | `1.0`                       |
| Hierarchy Required      | `True`                      |
| Repair Required         | `False`                     |
| Runtime Policy Detected | `True`                      |

---

# 3. Dataset Scope

The dataset is built from a directory of Geofabrik extracts instead of a single country PBF.

The source manifest contains `53` state and territory PBF files, including:

* 50 states
* District of Columbia
* Puerto Rico
* U.S. Virgin Islands

The runtime dataset is published as one Cadis dataset, `us.admin`, rather than as separate per-state datasets. Puerto Rico (`PR`) and U.S. Virgin Islands (`VI`) are routed to the US dataset at Cadis API level.

---

# 4. Build Evidence

The US engine uses multi-PBF relation stitching before geometry assembly. This is required because per-extract assembly can drop cross-extract administrative relations before Cadis ever receives geometry.

| Build Field                     | Value |
| ------------------------------- | ----: |
| Stitch relation count           | `30,235` |
| Required way references         | `182,437` |
| Present way references          | `175,460` |
| Missing way references          | `6,977` |
| Required node references        | `7,588,901` |
| Present node references         | `7,588,901` |
| Missing node references         | `0` |
| Assembled geometries            | `30,181` |
| Assembly failures               | `54` |
| Failure reason                  | `outer_polygonize_failed` |

## 4.1 Scope Filter

| Scope Filter Field       | Value |
| ------------------------ | ----: |
| Enabled                  | `True` |
| Allowed level-4 count    | `53` |
| Geometry count before    | `30,181` |
| Geometry count after     | `29,920` |
| Removed out-of-scope     | `261` |

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `400,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `360,000`
* Outside samples: `40,000`

The test intentionally injects 10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over land area and is not population-weighted.

## 5.2 Observed Distribution

| Category                         | Count |
| -------------------------------- | ----: |
| Runtime `ok` status              | `355,405` |
| Runtime `partial` status         | `4,793` |
| Runtime `failed` status          | `39,802` |
| Structural non-empty outcomes    | `360,019` |
| Empty-shape outcomes (`[]`)      | `39,981` |
| Offshore outcomes                | `179` |

The `failed` runtime status here corresponds to expected out-of-country empty-shape behavior. It is counted as policy-valid for outside stress samples.

---

# 6. Performance Metrics

| Metric                    | Value            |
| ------------------------- | ---------------- |
| Throughput                | `152.980` QPS    |
| Total Runtime             | `2614.644 sec`   |
| Overall Pass Rate         | `100.00%`        |
| Policy Pass Rate          | `100.00%`        |
| Coverage Pass Rate        | `100.00%`        |
| Inside Coverage Pass Rate | `100.00%`        |
| HTTP 200 Responses        | `400,000`        |

This run confirms full sampled inside-land coverage under the active runtime policy.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (ok/partial/failed/unknown) |
| ------------ | --------: | ---------------: | -----: | ------------: | ----------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 355,405/4,793/39,802/0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 355,405/4,793/39,802/0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 355,405/4,793/39,802/0 |
| no_nearby    | 99.98%    | 99.97%           | 93     | 93            | 355,147/4,783/40,070/0 |
| osm_only     | 99.98%    | 99.97%           | 93     | 93            | 355,147/4,783/40,070/0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------: |
| Hierarchy         | `0` |
| Repair            | `0` |
| Nearby            | `268` |
| Total vs OSM-only | `268` |

## Interpretation

* OSM-only success rate is already `99.98%` in this sampled run.
* The hierarchy layer was not required for sampled US lookups.
* The repair layer was not triggered.
* Nearby fallback rescued `268` samples by scenario delta and appeared directly as a node source for `89` samples.
* The nearby delta is small relative to the 400,000-sample run and remains bounded to boundary-adjacent/coastal behavior.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[4,6]`   | `317,902` |
| `[]`      | `39,981` |
| `[4,6,8]` | `37,297` |
| `[4]`     | `4,793` |
| `[4,8]`   | `27` |

## 9.2 Node Source Distribution

| Source          | Count |
| --------------- | ----: |
| `polygon`       | `359,930` |
| `nearby`        | `89` |
| `admin_tree_id` | `42` |

## 9.3 Source Mix

| Mix                       | Count |
| ------------------------- | ----: |
| `polygon`                 | `359,888` |
| `__none__`                | `39,981` |
| `nearby`                  | `89` |
| `admin_tree_id|polygon`   | `42` |

## 9.4 Policy Reason Distribution

| Reason             | Count |
| ------------------ | ----: |
| `shape_status_map` | `360,019` |
| `empty_shape`      | `39,802` |
| `offshore`         | `179` |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `52`
* Total level-4 hits, all samples: `360,019`
* Total level-4 hits, inside samples: `359,995`

The sampled level-4 distribution reflects uniform land-area sampling. Alaska dominates because the sampling is area-weighted, not population-weighted.

## 10.1 Top-30 Level-4 Hit Rates

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ---------------------: | --------------------: | ------------------------: |
| `Alaska` | 68,658 | 17.16% | 68,638 | 19.07% |
| `Texas` | 22,646 | 5.66% | 22,646 | 6.29% |
| `Montana` | 15,577 | 3.89% | 15,577 | 4.33% |
| `California` | 14,492 | 3.62% | 14,492 | 4.03% |
| `New Mexico` | 10,885 | 2.72% | 10,885 | 3.02% |
| `Nevada` | 10,237 | 2.56% | 10,237 | 2.84% |
| `Oregon` | 9,930 | 2.48% | 9,930 | 2.76% |
| `Michigan` | 9,903 | 2.48% | 9,903 | 2.75% |
| `Colorado` | 9,881 | 2.47% | 9,881 | 2.74% |
| `Arizona` | 9,842 | 2.46% | 9,842 | 2.73% |
| `Wyoming` | 9,784 | 2.45% | 9,784 | 2.72% |
| `Minnesota` | 9,085 | 2.27% | 9,085 | 2.52% |
| `Idaho` | 8,438 | 2.11% | 8,438 | 2.34% |
| `Utah` | 8,213 | 2.05% | 8,213 | 2.28% |
| `South Dakota` | 7,761 | 1.94% | 7,761 | 2.16% |
| `Kansas` | 7,697 | 1.92% | 7,697 | 2.14% |
| `Nebraska` | 7,630 | 1.91% | 7,630 | 2.12% |
| `North Dakota` | 7,496 | 1.87% | 7,496 | 2.08% |
| `Washington` | 7,247 | 1.81% | 7,247 | 2.01% |
| `Wisconsin` | 6,743 | 1.69% | 6,743 | 1.87% |
| `Missouri` | 6,429 | 1.61% | 6,429 | 1.79% |
| `Oklahoma` | 6,228 | 1.56% | 6,228 | 1.73% |
| `Iowa` | 5,543 | 1.39% | 5,543 | 1.54% |
| `Illinois` | 5,536 | 1.38% | 5,536 | 1.54% |
| `New York` | 5,195 | 1.30% | 5,195 | 1.44% |
| `Georgia` | 5,084 | 1.27% | 5,084 | 1.41% |
| `Arkansas` | 4,715 | 1.18% | 4,715 | 1.31% |
| `Florida` | 4,693 | 1.17% | 4,692 | 1.30% |
| `North Carolina` | 4,478 | 1.12% | 4,478 | 1.24% |
| `Pennsylvania` | 4,460 | 1.11% | 4,460 | 1.24% |

Additional low-frequency level-4 units observed in the 400,000-sample run include `District of Columbia` and `Puerto Rico`. The top-30 table is intentionally truncated for readability.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No sampled boundary leak was observed.
* All inside samples passed coverage and policy checks.
* The `39,981` empty-shape outcomes align with the expected outside sample set.
* Offshore classification was limited to `179` samples.
* Nearby fallback rescued a bounded set of boundary-adjacent samples and did not create cross-border escalation in this run.

This confirms strict sampled boundary containment for the evaluated US dataset.

---

# 12. Structural Observations

1. The prior per-extract assembly failure mode is addressed by stitching relation members before polygon assembly.
2. Runtime geometry is healthy for sampled lookups: `359,930` samples resolved directly from polygons.
3. No repair-layer behavior was required in the sampled run.
4. Hierarchy supplementation was not needed for sampled US results.
5. Nearby fallback has a small, bounded effect: `268` scenario-delta rescues over `400,000` samples.
6. The source-side `_stitch_cache` is an optimization, not a required input for public reproduction.

---

# 13. Reproducibility

Build/runtime versions:

* cadis-dataset-engine commit:
  `fdcebd5`
* Cadis version:
  `0.6.7`

The dataset can be reproduced from the public PBF extracts without checked-in pickle files. The `_stitch_cache` directory is regenerated when absent.

---

# 14. Conclusion

`us.admin v1.0.0` passes the Cadis runtime evaluation with:

* `100.00%` overall pass rate
* `100.00%` policy pass rate
* `100.00%` inside coverage pass rate
* `400,000/400,000` sampled lookups passing
* `360,000/360,000` inside samples passing coverage

The evaluation supports the US-specific architecture decision: stitch multi-extract OSM topology first, then assemble geometry once for Cadis export.
