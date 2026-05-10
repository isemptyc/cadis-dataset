# IN_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `in.admin`
Version: `v1.0.0`
Country: `IN`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `in.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the selected India dataset scope
* Quantifies structural completeness under randomized lookup stress testing
* Documents deterministic policy effects
* Validates boundary isolation against the declared evaluation boundary
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse shapes reflect structural incompleteness or intentionally partial administrative coverage, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value                  |
| ----------------------- | ---------------------- |
| Dataset ID              | `in.admin`             |
| Dataset Version         | `v1.0.0`               |
| Country                 | `IN`                   |
| Country Name            | `India`                |
| Policy Version          | `1.0`                  |
| Name Schema             | `multilingual_v1`      |
| Hierarchy Required      | `True`                 |
| Repair Required         | `False`                |
| Runtime Policy Detected | `True`                 |
| Minimum Cadis Version   | `0.8.16`               |

---

# 3. Dataset Scope

The `in.admin v1.0.0` release scope is:

`OSM admin-level-4 coverage union, derived from India state/union-territory polygons after Natural Earth IN admin-0 filtering`

Boundary source of truth:

* Tracked builder script: `build_in_boundaries.py`
* Generated build/evaluation boundary: `in_country.json`
* Natural Earth source: `ne_10m_admin_0_countries.dbf`
* OSM source: `india-260509.osm.pbf`
* Selection rule: union all extracted India `admin_level=4` state and union-territory polygons after NE `IN` admin-0 filtering
* Admin-level-4 polygon count: `34`
* Scoped boundary bbox: `[68.1756585, 6.757237, 96.0124397, 35.6728459]`

The same scoped boundary was used for build and evaluation.
Natural Earth components not covered by OSM admin-level-4 state/union-territory polygons are outside this v1.0.0 dataset evaluation scope.

---

# 4. Test Methodology

## 4.1 Sampling Strategy

* Total samples: `50,000`
* Sampling mode: mixed inside/outside stress testing
* Inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Evaluation boundary: `in_country.json`

The test intentionally injects ~10% out-of-scope points to validate:

* Boundary rejection behavior
* Offshore classification
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over the declared scope, not population-weighted.

---

# 5. Performance Metrics

| Metric                    | Value          |
| ------------------------- | -------------- |
| Throughput                | `2310.790` QPS |
| Total Runtime             | `21.638 sec`   |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |
| Failed Samples            | `0`            |

The 50,000-point run included `45,000` inside-scope samples and `5,000` out-of-scope samples.

The staged dataset passed all randomized runtime checks in this evaluation run.

---

# 6. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (`ok`/`partial`/`failed`) |
| ------------ | --------- | ---------------- | ------ | ------------- | -------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 43,617 / 1,402 / 4,981           |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 43,617 / 1,402 / 4,981           |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 43,617 / 1,402 / 4,981           |
| no_nearby    | 99.998%   | 99.998%          | 1      | 1             | 43,601 / 1,399 / 5,000           |
| osm_only     | 99.998%   | 99.998%          | 1      | 1             | 43,601 / 1,399 / 5,000           |

---

# 7. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 19              |
| Total vs OSM-only | 19              |

## Interpretation

* OSM-only success rate was already high for the declared scope.
* Dataset-scoped hierarchy is present, but the observed randomized sample did not require hierarchy supplementation.
* No repair layer was required.
* Nearby fallback rescued a small number of boundary-adjacent samples.
* The policy mode achieved full inside-boundary coverage for the declared v1.0.0 scope.

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape       | Count |
| ----------- | ----- |
| `[4,5,6]`   | 39,809 |
| `[]`        | 4,995  |
| `[4,5,6,9]` | 3,794  |
| `[4,5]`     | 1,313  |
| `[4]`       | 40     |
| `[4,6]`     | 26     |
| `[4,5,9]`   | 16     |
| `[4,6,9]`   | 7      |

Empty shapes correspond to expected out-of-scope or offshore samples:

* `empty_shape`: 4,981
* `offshore`: 14

## 8.2 Node Source Distribution

| Source  | Count |
| ------- | ----- |
| polygon         | 45,000 |
| nearby          | 5      |
| admin_tree_id   | 1      |

### Source Mix

| Mix     | Count |
| ------- | ----- |
| polygon               | 44,999 |
| none                  | 4,995  |
| nearby                | 5      |
| admin_tree_id/polygon | 1      |

---

# 9. Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----- |
| shape_status_map | 45,005 |
| empty_shape      | 4,981  |
| offshore         | 14     |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `34`
* Total level-4 hits: `45,005`
* Total level-4 hits for inside samples: `45,000`

Hit distribution reflects uniform area sampling under the declared boundary.

## 10.1 Top-10 Level-4 Hit Rates

| Level-4 Unit     | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ---------------- | ---- | ---------------------- | --------------------- | ------------------------- |
| Rajasthan        | 5,153 | 10.31%                 | 5,153                 | 11.45%                    |
| Madhya Pradesh   | 4,416 | 8.83%                  | 4,416                 | 9.81%                     |
| Maharashtra      | 4,297 | 8.59%                  | 4,297                 | 9.55%                     |
| Uttar Pradesh    | 3,685 | 7.37%                  | 3,685                 | 8.19%                     |
| Karnataka        | 2,701 | 5.40%                  | 2,701                 | 6.00%                     |
| Gujarat          | 2,642 | 5.28%                  | 2,639                 | 5.86%                     |
| Andhra Pradesh   | 2,367 | 4.73%                  | 2,366                 | 5.26%                     |
| Odisha           | 2,207 | 4.41%                  | 2,207                 | 4.90%                     |
| Chhattisgarh     | 1,967 | 3.93%                  | 1,967                 | 4.37%                     |
| Tamil Nadu       | 1,808 | 3.62%                  | 1,808                 | 4.02%                     |

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-scope samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation in this run.
* The evaluation used the same scoped boundary that the build used.

This confirms strict boundary containment within the declared `IN` v1.0.0 scope.

---

# 12. Artifact Metrics

| Artifact                 | Size     |
| ------------------------ | -------- |
| Package tarball          | 32 MB    |
| Unpacked staged dataset  | 55 MB    |
| `geometry.ffsf`          | 32 MB    |
| `geometry_meta.json`     | 15 MB    |
| `hierarchy.json`         | 8.0 MB   |
| `runtime_policy.json`    | 2.2 KB   |

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit: `bc0589536b07dbafab6efbcf2c5aea7cfc14242d`
* Cadis version prepared locally: `0.8.16`
* OSM source label: `geofabrik:asia/india`
* OSM file: `india-260509.osm.pbf`
* OSM replication timestamp: `2026-05-09T20:21:02Z`
* OSM file SHA256: `8e08957fe6ace40ca08056cc1742d744262b722f542cb09b31e16f8dd7f6f65b`

The engine repo was clean and committed before the staged dataset was generated.

---

# 14. Conclusion

The `in.admin v1.0.0` staged dataset demonstrates:

* Full pass rate under the declared India v1.0.0 scope
* Full inside-boundary coverage in the randomized evaluation run
* High OSM-only coverage with minimal nearby fallback
* No repair-layer requirement
* Strict boundary isolation under mixed inside/outside stress testing

