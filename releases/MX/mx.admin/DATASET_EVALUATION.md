# MX_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `mx.admin`
Version: `v1.0.0`
Country: `MX`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `mx.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the selected Mexico dataset scope
* Quantifies structural completeness under randomized lookup stress testing
* Documents deterministic policy effects
* Validates boundary isolation against the declared evaluation boundary
* Provides reproducible integrity metrics for release review

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value        |
| ----------------------- | ------------ |
| Dataset ID              | `mx.admin`   |
| Dataset Version         | `v1.0.0`     |
| Country                 | `MX`         |
| Country Name            | `Mexico`     |
| Policy Version          | `1.0`        |
| Hierarchy Required      | `True`       |
| Repair Required         | `False`      |
| Runtime Policy Detected | `True`       |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value |
| ------------------------ | ----- |
| Scope Label              | `full MX Natural Earth admin-0 boundary` |
| Boundary Builder Script  | `build_mx_boundaries.py` |
| Generated Boundary JSON  | `mx_country.json` |
| Source Boundary          | `ne_10m_admin_0_countries.dbf` |
| Selection Rule           | Natural Earth admin-0 country row selected by `ISO2=MX` and country name `Mexico` |
| Boundary BBox            | `[-118.36880422899992, 14.546279476000095, -86.70059160099993, 32.71283640500009]` |
| Major Excluded Components | None intentionally excluded |

The same generated boundary JSON was used for build and evaluation.

The v1.0.0 engine intentionally exposes OSM administrative levels:

* `4`: state / federal entity
* `6`: municipality

The staged hierarchy contains `2,507` nodes:

| Level | Count |
| ----- | ----: |
| `4`   | 32    |
| `6`   | 2,475 |

All `2,475` level-6 nodes have a parent in the staged hierarchy.

---

# 4. Source Snapshot

| Field                         | Value |
| ----------------------------- | ----- |
| OSM Source                    | `geofabrik:north-america/mexico` |
| OSM Replication Timestamp UTC | `2026-05-09T20:21:02Z` |
| OSM File SHA256               | `c49184f607c6ee8b1629fafd1aa59f6fc638ef3cd6aae260fab22b27d641cf87` |
| Engine Commit                 | `7110e654df72d3f48ee86260b02ceb2f1142f6fe` |
| Cadis Runtime Compatibility   | `>=0.8.17, <1.0.0` |

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Lookup mode: Cadis runtime
* Evaluation boundary: `mx_country.json`

The test intentionally injects approximately 10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over land area and is not population-weighted.

---

# 6. Performance Metrics

| Metric                    | Value          |
| ------------------------- | -------------- |
| Throughput                | `8471.280` QPS |
| Total Runtime             | `1.180 sec`    |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |
| Failed Samples            | `0`            |

The evaluation observed no inside-boundary lookup failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (ok/partial/failed/unknown) |
| ------------ | --------- | ---------------- | ------ | ------------- | ----------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,001 / 3 / 996 / 0                 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,001 / 3 / 996 / 0                 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,001 / 3 / 996 / 0                 |
| no_nearby    | 99.77%    | 99.74%           | 23     | 23            | 8,975 / 3 / 1,022 / 0               |
| osm_only     | 99.77%    | 99.74%           | 23     | 23            | 8,975 / 3 / 1,022 / 0               |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 26              |
| Total vs OSM-only | 26              |

## Interpretation

* OSM-only success rate was already high at `99.77%`.
* Hierarchy and repair layers did not rescue sampled lookups in this run.
* Nearby fallback recovered coastal or near-boundary samples that would otherwise be empty.
* No repair layer is required for this dataset.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape   | Count |
| ------- | ----: |
| `[4,6]` | 8,994 |
| `[]`    | 1,003 |
| `[4]`   | 3     |

Empty shapes correspond to expected outside or offshore outcomes:

| Reason        | Count |
| ------------- | ----: |
| `empty_shape` | 996   |
| `offshore`    | 7     |

## 9.2 Node Source Distribution

| Source          | Count |
| --------------- | ----: |
| `polygon`       | 8,978 |
| `nearby`        | 19    |
| `admin_tree_id` | 9     |

## 9.3 Source Mix Distribution

| Source Mix              | Count |
| ----------------------- | ----: |
| `polygon`               | 8,969 |
| `__none__`              | 1,003 |
| `nearby`                | 19    |
| `admin_tree_id|polygon` | 9     |

## 9.4 Policy Reason Distribution

| Reason             | Count |
| ------------------ | ----: |
| `shape_status_map` | 8,997 |
| `empty_shape`      | 996   |
| `offshore`         | 7     |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `32`
* Total level-4 hits: `8,997`
* Total level-4 hits from inside samples: `8,996`

All 32 Mexico level-4 units were hit in the 10,000-point randomized test.

## 10.1 Top Level-4 Hit Rates

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ---------------------- | --------------------: | ------------------------- |
| Chihuahua | 1,192 | 11.92% | 1,192 | 13.24% |
| Sonora | 862 | 8.62% | 862 | 9.58% |
| Coahuila | 682 | 6.82% | 682 | 7.58% |
| Durango | 559 | 5.59% | 559 | 6.21% |
| Oaxaca | 452 | 4.52% | 452 | 5.02% |
| Jalisco | 391 | 3.91% | 391 | 4.34% |
| Baja California | 384 | 3.84% | 384 | 4.27% |
| Tamaulipas | 350 | 3.50% | 350 | 3.89% |
| Zacatecas | 347 | 3.47% | 347 | 3.86% |
| Baja California Sur | 325 | 3.25% | 324 | 3.60% |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No inside-boundary lookup failure was observed.
* No policy failure was observed.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.

This confirms strict boundary containment within the declared MX dataset scope for this sampled run.

---

# 12. Structural Observations

1. The `4 -> 6` model is structurally strong for Mexico in the source PBF.
2. The staged hierarchy contains all 32 level-4 units and 2,475 level-6 municipalities.
3. Every level-6 node has a level-4 parent in the staged hierarchy.
4. The repair layer is not required.
5. Nearby fallback is useful but limited, rescuing 26 samples versus OSM-only mode.
6. Boundary evaluation used the same generated Natural Earth MX boundary used for the build.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

* Engine commit: `7110e654df72d3f48ee86260b02ceb2f1142f6fe`
* Cadis version: `0.8.17`

The dataset was generated after committing the Mexico engine.
Cadis support/version changes were present locally during evaluation and are intentionally not yet committed.

---

# 14. Conclusion

The `mx.admin v1.0.0` staged dataset demonstrates:

* Full inside-boundary lookup coverage under policy mode
* Complete level-4 coverage in sampled evaluation
* Strong level-6 parentage under the selected Mexico model
* No repair-layer requirement
* Bounded nearby fallback contribution
* Strict boundary behavior against the declared MX evaluation boundary
