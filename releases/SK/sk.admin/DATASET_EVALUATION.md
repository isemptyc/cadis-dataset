# SK_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `sk.admin`
Version: `v1.0.0`
Country: `SK`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `sk.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Slovakia dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `sk.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `SK`              |
| Country Name            | `Slovakia`        |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.36`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_sk_boundaries.py`       |
| Generated Boundary       | `tmp/sk_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth SK admin-0 country boundary.` |
| Boundary BBox            | `[16.84448042800011, 47.75000640900008, 22.539636678000136, 49.60177968400002]` |
| OSM Source               | `geofabrik:europe/slovakia`            |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Slovakia engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_region`       | 8             | Regions |
| 6     | `admin_district`     | 79            | Districts |
| 8     | `admin_municipality` | 2,856         | Municipalities and cities |
| 9     | `admin_borough`      | 39            | Bratislava and Košice borough-level areas |
| 10    | `admin_locality`     | 3,517         | Locality/detail coverage |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 3     | 4           | Excluded as broad statistical coverage |
| 5     | 2           | Excluded as sparse Bratislava/Košice city envelopes |
| 7     | 0           | Excluded because no scoped features were found |

The engine uses canonical names from `name:sk`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `cs`, `de`, `en`, `hu`, and `sk`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `SK/sk.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over land area, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value          |
| ------------------------- | -------------- |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |
| Failed Samples            | `0`            |
| Throughput                | `7734.240` QPS |
| Total Runtime             | `1.293 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,019 / 19 / 962 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,019 / 19 / 962 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,019 / 19 / 962 / 0 |
| no_nearby    | 99.48%    | 99.42%           | 52     | 52            | 8,929 / 19 / 1,052 / 0 |
| osm_only     | 99.48%    | 99.42%           | 52     | 52            | 8,929 / 19 / 1,052 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 90              |
| Total vs OSM-only | 90              |

## Interpretation

* OSM-only success rate: `99.48%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves boundary-adjacent and geometry-edge samples that otherwise miss polygon containment.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `sk.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape          | Count |
| -------------- | ----: |
| `[4,6,8,10]`   | 8,810 |
| `[]`           | 998   |
| `[4,6,9,10]`  | 122   |
| `[4,6]`        | 25    |
| `[4,8,10]`    | 13    |
| `[6,8,10]`    | 12    |
| `[4,6,8]`     | 7     |
| `[8]`          | 5     |

Additional low-count shapes observed in the full report were `[8,10]`, `[4]`, `[4,9,10]`, and `[4,6,8,9,10]`.

Empty shapes correspond to:

* `empty_shape`: 962
* `offshore`: 36

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,948 |
| nearby        | 54    |
| admin_tree_id | 1     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,947 |
| none                   | 998   |
| nearby                 | 54    |
| admin_tree_id\|polygon | 1     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,002 |
| empty_shape      | 962   |
| offshore         | 36    |

---

# 10. Level-4 Coverage

* Unique level-4 region units hit: `8`
* Total level-4 hits across all samples: `8,983`
* Total level-4 hits across inside samples: `8,980`

| Level-4 Unit          | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| --------------------- | ---: | ----------------------: | --------------------: | -------------------------: |
| Banskobystrický kraj  | 1,733 | 17.33%                  | 1,731                 | 19.23%                     |
| Prešovský kraj        | 1,633 | 16.33%                  | 1,633                 | 18.14%                     |
| Žilinský kraj         | 1,255 | 12.55%                  | 1,254                 | 13.93%                     |
| Košický kraj          | 1,188 | 11.88%                  | 1,188                 | 13.20%                     |
| Nitriansky kraj       | 1,099 | 10.99%                  | 1,099                 | 12.21%                     |
| Trenčiansky kraj      | 905   | 9.05%                   | 905                   | 10.06%                     |
| Trnavský kraj         | 786   | 7.86%                   | 786                   | 8.73%                      |
| Bratislavský kraj     | 384   | 3.84%                   | 384                   | 4.27%                      |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/sk_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Slovakia dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,6,8,10]`, reflecting region, district, municipality, and locality coverage.
3. Level 9 adds borough detail for Bratislava and Košice where available.
4. Nearby fallback is bounded and accounts for a small rescue effect around polygon edges.
5. Levels 3 and 5 are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `dcf02dd3f42eeb373ba70d0d0b9675b699a98ba5`
- Cadis version:
  `0.8.36`
- Boundary builder: `scripts/build_sk_boundaries.py`
- Build boundary: `tmp/sk_country.json`
- Evaluation boundary: `tmp/sk_country.json`
- Staged dataset: `SK/sk.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `sk.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Slovakia levels 4, 6, 8, 9, and 10

The dataset is suitable for human quality review.
