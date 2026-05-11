# HU_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `hu.admin`
Version: `v1.0.0`
Country: `HU`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `hu.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Hungary dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `hu.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `HU`              |
| Country Name            | `Hungary`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.29`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_hu_boundaries.py`       |
| Generated Boundary       | `tmp/hu_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth HU admin-0 country boundary.` |
| Boundary BBox            | `[16.09403527800012, 45.74134348600002, 22.877600546000053, 48.569232890000066]` |
| OSM Source               | `geofabrik:europe/hungary`             |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Hungary engine exposes these OSM administrative levels:

| Level | Runtime Label           | Dataset Count | Notes |
| ----: | ----------------------- | ------------: | ----- |
| 6     | `admin_county`          | 20            | Counties and Budapest |
| 7     | `admin_district`        | 174           | Districts |
| 8     | `admin_municipality`    | 3,147         | Municipalities and settlements |
| 9     | `admin_capital_district` | 23           | Budapest districts |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 4     | 3           | Excluded as broad statistical region coverage |
| 5     | 7           | Excluded as broad planning/statistical region coverage |
| 10    | 271         | Excluded as sparse neighborhood / cadastral detail |

The engine uses canonical names from `name:hu`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `de`, `en`, `hu`, `ru`, and `sk`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `HU/hu.admin/v1.0.0`

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
| Throughput                | `11351.290` QPS |
| Total Runtime             | `0.881 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,017 / 5 / 978 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,017 / 5 / 978 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,017 / 5 / 978 / 0 |
| no_nearby    | 99.19%    | 99.10%           | 81     | 81            | 8,915 / 5 / 1,080 / 0 |
| osm_only     | 99.19%    | 99.10%           | 81     | 81            | 8,915 / 5 / 1,080 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 102             |
| Total vs OSM-only | 102             |

## Interpretation

* OSM-only success rate: `99.19%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves boundary-adjacent samples that otherwise miss polygon containment.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `hu.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape       | Count |
| ----------- | ----: |
| `[6,7,8]`   | 8,917 |
| `[]`        | 1,001 |
| `[6,8,9]`   | 52    |
| `[6,8]`     | 12    |
| `[6,7]`     | 9     |
| `[8]`       | 5     |
| `[6]`       | 3     |
| `[6,7,8,9]` | 1    |

Empty shapes correspond to:

* `empty_shape`: 978
* `offshore`: 23

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,920 |
| nearby        | 79    |
| admin_tree_id | 7     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,913 |
| none                   | 1,001 |
| nearby                 | 79    |
| admin_tree_id\|polygon | 7     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 8,999 |
| empty_shape      | 978   |
| offshore         | 23    |

---

# 10. Level-6 Coverage

* Included level-6 county units: `20`
* Level-6 units include Hungary's 19 counties plus Budapest.
* The evaluation summary does not emit a dedicated level-6 hit-rate table; the observed structural shapes show county-level coverage in nearly all non-empty outcomes.

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/hu_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Hungary dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[6,7,8]`, reflecting county, district, and municipality coverage.
3. Budapest district coverage appears through level 9 shapes such as `[6,8,9]`.
4. Nearby fallback is bounded and accounts for the only meaningful rescue effect.
5. Levels 4, 5, and 10 are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `4773f47e4b53c2d3c1fd7b451366cc5370b5fca7`
- Cadis version:
  `0.8.29`
- Boundary builder: `scripts/build_hu_boundaries.py`
- Build boundary: `tmp/hu_country.json`
- Evaluation boundary: `tmp/hu_country.json`
- Staged dataset: `HU/hu.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `hu.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Hungary levels 6, 7, 8, and 9

The dataset is suitable for human quality review.
