# LV_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `lv.admin`
Version: `v1.0.0`
Country: `LV`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `lv.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Latvia dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `lv.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `LV`              |
| Country Name            | `Latvia`          |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.30`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_lv_boundaries.py`       |
| Generated Boundary       | `tmp/lv_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth LV admin-0 country boundary.` |
| Boundary BBox            | `[20.968597852000073, 55.66699086600006, 28.217274617000072, 58.07513844900008]` |
| OSM Source               | `geofabrik:europe/latvia`              |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Latvia engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 5     | `admin_municipality` | 42            | Municipalities and state cities |
| 7     | `admin_city_or_town` | 74            | City/town units |
| 8     | `admin_parish`       | 539           | Parishes and local subdivision units |
| 9     | `admin_locality`     | 1,336         | Villages and localities |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 10    | 125         | Excluded as sparse neighborhood coverage |

The engine uses canonical names from `name:lv`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `be`, `lt`, `lv`, `pl`, and `ru`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `LV/lv.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over land area, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value           |
| ------------------------- | --------------- |
| Overall Pass Rate         | `100.00%`       |
| Inside Coverage Pass Rate | `100.00%`       |
| Policy Pass Rate          | `100.00%`       |
| Failed Samples            | `0`             |
| Throughput                | `12791.850` QPS |
| Total Runtime             | `0.782 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,022 / 16 / 962 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,022 / 16 / 962 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,022 / 16 / 962 / 0 |
| no_nearby    | 99.06%    | 98.96%           | 94     | 94            | 8,890 / 16 / 1,094 / 0 |
| osm_only     | 99.06%    | 98.96%           | 94     | 94            | 8,890 / 16 / 1,094 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 132             |
| Total vs OSM-only | 132             |

## Interpretation

* OSM-only success rate: `99.06%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves boundary-adjacent and coastal-edge samples.
* Latvia uses a 5 km nearby radius because the initial 2 km policy left one inside-boundary coastal sample unresolved.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `lv.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape       | Count |
| ----------- | ----: |
| `[5,8]`     | 8,507 |
| `[]`        | 991   |
| `[5,8,9]`   | 277   |
| `[5]`       | 79    |
| `[5,7]`     | 72    |
| `[5,9]`     | 45    |
| `[8]`       | 14    |
| `[5,7,8]`   | 10    |
| `[5,7,8,9]` | 3    |
| `[8,9]`     | 2     |

Empty shapes correspond to:

* `empty_shape`: 962
* `offshore`: 29

## 9.2 Node Source Distribution

| Source  | Count |
| ------- | ----: |
| polygon | 8,906 |
| nearby  | 103   |

## 9.3 Source Mix Distribution

| Mix     | Count |
| ------- | ----: |
| polygon | 8,906 |
| none    | 991   |
| nearby  | 103   |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,009 |
| empty_shape      | 962   |
| offshore         | 29    |

---

# 10. Level-5 Coverage

* Included level-5 municipality/state-city units: `42`
* Level-5 is the required parent scope for the Latvia runtime policy.
* The evaluation summary does not emit a dedicated level-5 hit-rate table; the observed structural shapes show level-5 coverage in nearly all non-empty outcomes.

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/lv_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Latvia dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[5,8]`, reflecting municipality/state-city and parish coverage.
3. Level 9 localities add settlement detail where OSM boundaries are available.
4. Nearby fallback is bounded and accounts for the only meaningful rescue effect.
5. Level 10 is intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `05172be2076d2507434398f15f0c26012738147e`
- Cadis version:
  `0.8.30`
- Boundary builder: `scripts/build_lv_boundaries.py`
- Build boundary: `tmp/lv_country.json`
- Evaluation boundary: `tmp/lv_country.json`
- Staged dataset: `LV/lv.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `lv.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Latvia levels 5, 7, 8, and 9

The dataset is suitable for human quality review.
