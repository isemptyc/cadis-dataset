# SR_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `sr.admin`
Version: `v1.0.0`
Country: `SR`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `sr.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Suriname dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `sr.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `SR`              |
| Country Name            | `Suriname`        |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.48`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_sr_boundaries.py`       |
| Generated Boundary       | `tmp/sr_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth SR admin-0 country boundary.` |
| Boundary BBox            | `[-58.067691202999924, 1.8335067750000604, -53.986357453999915, 6.011573766000083]` |
| OSM Source               | `geofabrik:south-america/suriname`     |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Suriname engine exposes these OSM administrative levels:

| Level | Runtime Label   | Dataset Count | Notes |
| ----: | --------------- | ------------: | ----- |
| 4     | `admin_district` | 8             | Districts represented in the selected source PBF |
| 6     | `admin_resort`   | 63            | Resorts/lower administrative units |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 0           | Excluded because no scoped features were present |
| 3     | 0           | Excluded because no scoped features were present |
| 5     | 0           | Excluded because no scoped features were present |
| 7     | 0           | Excluded because no scoped features were present |
| 8     | 0           | Excluded because no scoped features were present |
| 9     | 0           | Excluded because no scoped features were present |
| 10    | 0           | Excluded because no scoped features were present |

The engine uses canonical names from `name:nl`, `name`, `official_name`, and `name:en`, with bounded multilingual aliases for `nl` and `en`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `SR/sr.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over land area, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value           |
| ------------------------- | --------------- |
| Overall Pass Rate         | `100.00%`       |
| Inside Coverage Pass Rate | `100.00%`       |
| Policy Pass Rate          | `100.00%`       |
| Failed Samples            | `0`             |
| Throughput                | `20088.060` QPS |
| Total Runtime             | `0.498 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 8,719 / 313 / 968 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 8,719 / 313 / 968 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 8,719 / 313 / 968 / 0 |
| no_nearby    | 99.71%    | 99.68%           | 29     | 29            | 8,665 / 313 / 1,022 / 0 |
| osm_only     | 99.71%    | 99.68%           | 29     | 29            | 8,665 / 313 / 1,022 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 54              |
| Total vs OSM-only | 54              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `sr.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape   | Count |
| ------- | ----: |
| `[4,6]` | 8,675 |
| `[]`    | 998   |
| `[6]`   | 313   |
| `[4]`   | 14    |

Empty shapes correspond to:

* `empty_shape`: 968
* `offshore`: 30

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,978 |
| nearby        | 24    |
| admin_tree_id | 15    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,963 |
| none                   | 998   |
| nearby                 | 24    |
| admin_tree_id\|polygon | 15    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,002 |
| empty_shape      | 968   |
| offshore         | 30    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `8`
* Total level-4 hits across all samples: `8,689`
* Total level-4 hits across inside samples: `8,681`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Sipaliwini | 7,009 | 70.09% | 7,009 | 77.88% |
| Brokopondo | 422 | 4.22% | 422 | 4.69% |
| Para | 347 | 3.47% | 347 | 3.86% |
| Marowijne | 289 | 2.89% | 285 | 3.17% |
| Saramacca | 241 | 2.41% | 240 | 2.67% |
| Coronie | 222 | 2.22% | 220 | 2.44% |
| Commewijne | 140 | 1.40% | 139 | 1.54% |
| Wanica | 19 | 0.19% | 19 | 0.21% |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/sr_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Suriname dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,6]`, reflecting district and resort coverage.
3. The source extract contains a minimal administrative model under the Suriname scope; the runtime contract intentionally exposes only levels 4 and 6.
4. A small number of `[6]`-only samples are marked `partial` because their level-4 parent was not present in the polygon result.
5. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `610a954a829f42ecfe0c4dd931d77bd33473c820`
- Cadis version:
  `0.8.48`
- Boundary builder: `scripts/build_sr_boundaries.py`
- Build boundary: `tmp/sr_country.json`
- Evaluation boundary: `tmp/sr_country.json`
- Staged dataset: `SR/sr.admin/v1.0.0`
- Source OSM SHA256:
  `cbc0b3480c75a194ddebc39874b4c02f736e7d2539bcba94fee26be7ae95ef80`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `sr.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Suriname levels 4 and 6

The dataset is suitable for human quality review.
