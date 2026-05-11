# SI_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `si.admin`
Version: `v1.0.0`
Country: `SI`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `si.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Slovenia dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `si.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `SI`              |
| Country Name            | `Slovenia`        |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.37`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_si_boundaries.py`       |
| Generated Boundary       | `tmp/si_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth SI admin-0 country boundary.` |
| Boundary BBox            | `[13.365261271000094, 45.423636779999995, 16.515301554000075, 46.86396230100003]` |
| OSM Source               | `geofabrik:europe/slovenia`            |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Slovenia engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 8     | `admin_municipality` | 220           | Municipalities |
| 10    | `admin_locality`     | 91            | Locality/detail coverage |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 7     | 9           | Excluded as sparse administrative-unit overlay coverage |
| 9     | 1           | Excluded as a single local community feature |

The engine uses canonical names from `name:sl`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `en`, `hr`, `it`, and `sl`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `SI/si.admin/v1.0.0`

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
| Throughput                | `9019.560` QPS |
| Total Runtime             | `1.109 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,015 / 44 / 941 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,015 / 44 / 941 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,015 / 44 / 941 / 0 |
| no_nearby    | 97.90%    | 97.67%           | 210    | 210           | 8,747 / 44 / 1,209 / 0 |
| osm_only     | 97.90%    | 97.67%           | 210    | 210           | 8,747 / 44 / 1,209 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 268             |
| Total vs OSM-only | 268             |

## Interpretation

* OSM-only success rate: `97.90%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves boundary-adjacent and municipal edge samples that otherwise miss polygon containment.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `si.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape   | Count |
| ------- | ----: |
| `[8]`   | 8,872 |
| `[]`    | 984   |
| `[8,10]` | 100  |
| `[10]`  | 44    |

Empty shapes correspond to:

* `empty_shape`: 941
* `offshore`: 43

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,791 |
| nearby        | 225   |
| admin_tree_id | 1     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,790 |
| none                   | 984   |
| nearby                 | 225   |
| admin_tree_id\|polygon | 1     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,016 |
| empty_shape      | 941   |
| offshore         | 43    |

---

# 10. Level-4 Coverage

No level-4 city hit rates are reported for this dataset because the Slovenia runtime contract does not expose level 4.

The primary runtime parent layer is level 8 municipality coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/si_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Slovenia dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[8]`, reflecting municipality-level coverage.
3. Level 10 adds sparse locality/detail coverage where available.
4. Nearby fallback is bounded at 5 km and accounts for the expected rescue effect around municipal and boundary edges.
5. Levels 7 and 9 are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `1ea4430a4a800cb8ba1f236cdf986ec3875b4db0`
- Cadis version:
  `0.8.37`
- Boundary builder: `scripts/build_si_boundaries.py`
- Build boundary: `tmp/si_country.json`
- Evaluation boundary: `tmp/si_country.json`
- Staged dataset: `SI/si.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `si.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Slovenia levels 8 and 10

The dataset is suitable for human quality review.
