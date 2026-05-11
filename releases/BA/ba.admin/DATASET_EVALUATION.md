# BA_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ba.admin`
Version: `v1.0.0`
Country: `BA`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ba.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Bosnia and Herzegovina dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value                       |
| ----------------------- | --------------------------- |
| Dataset ID              | `ba.admin`                  |
| Dataset Version         | `v1.0.0`                    |
| Country                 | `BA`                        |
| Country Name            | `Bosnia and Herzegovina`    |
| Policy Version          | `1.0`                       |
| Cadis Version           | `v0.8.25`                   |
| Hierarchy Required      | `True`                      |
| Repair Required         | `False`                     |
| Runtime Policy Detected | `True`                      |
| Name Schema             | `multilingual_v1`           |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_ba_boundaries.py`       |
| Generated Boundary       | `tmp/ba_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth BA admin-0 country boundary.` |
| Boundary BBox            | `[15.716073852000108, 42.55921213800009, 19.618884725000044, 45.28452382500008]` |
| OSM Source               | `geofabrik:europe/bosnia-herzegovina`  |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Bosnia and Herzegovina engine exposes these OSM administrative levels:

| Level | Runtime Label               | Probe Count | Notes |
| ----: | --------------------------- | ----------: | ----- |
| 4     | `admin_entity_or_district`  | 3           | Federation, Republika Srpska, and Brčko District |
| 5     | `admin_canton`              | 10          | Federation cantons |
| 6     | `admin_city`                | 34          | City-level administrative units |
| 7     | `admin_municipality`        | 111         | Municipalities |
| 8     | `admin_local_community`     | 129         | Local communities and related local units |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 9     | 3,497       | Excluded as broad settlement-level coverage |
| 10    | 5           | Excluded as sparse neighborhood coverage |

The engine uses canonical names from `name:bs`, `name:sr`, `name:hr`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `bs`, `sr`, `hr`, `sr-latn`, and `en`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `BA/ba.admin/v1.0.0`

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
| Throughput                | `13257.030` QPS |
| Total Runtime             | `0.754 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,005 / 26 / 969 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,005 / 26 / 969 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,005 / 26 / 969 / 0 |
| no_nearby    | 98.34%    | 98.16%           | 166    | 166           | 8,810 / 26 / 1,164 / 0 |
| osm_only     | 98.34%    | 98.16%           | 166    | 166           | 8,810 / 26 / 1,164 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 195             |
| Total vs OSM-only | 195             |

## Interpretation

* OSM-only success rate: `98.34%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves boundary-adjacent and coastal-edge samples.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `ba.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape           | Count |
| --------------- | ----: |
| `[4,7]`         | 2,876 |
| `[4,5,7]`       | 2,754 |
| `[4,5,6]`       | 1,616 |
| `[4,6]`         | 1,053 |
| `[]`            | 1,020 |
| `[4,6,7]`       | 236   |
| `[4,6,8]`       | 137   |
| `[4]`           | 114   |
| `[4,5,6,8]`     | 87    |
| `[4,5,6,7]`     | 30    |
| `[7]`           | 15    |
| `[4,5,7,8]`     | 14    |
| `[4,6,7,8]`     | 13    |
| `[4,5]`         | 12    |
| `[4,5,6,7,8]`   | 10    |
| `[8]`           | 6     |
| `[5,7]`         | 3     |
| `[6]`           | 2     |
| `[4,7,8]`       | 1     |
| `[4,8]`         | 1     |

Empty shapes correspond to:

* `empty_shape`: 969
* `offshore`: 51

## 9.2 Node Source Distribution

| Source          | Count |
| --------------- | ----: |
| polygon         | 8,836 |
| nearby          | 144   |
| admin_tree_id   | 8     |

## 9.3 Source Mix Distribution

| Mix                   | Count |
| --------------------- | ----: |
| polygon               | 8,828 |
| none                  | 1,020 |
| nearby                | 144   |
| admin_tree_id\|polygon | 8     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 8,980 |
| empty_shape      | 969   |
| offshore         | 51    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `3`
* Total level-4 hits across all samples: `8,954`
* Total level-4 hits across inside samples: `8,948`

| Level-4 Unit                    | Hits  | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| -------------------------------- | ----: | ----------------------: | --------------------: | -------------------------: |
| Federacija Bosne i Hercegovine   | 4,549 | 45.49%                  | 4,546                 | 50.51%                     |
| Republika Srpska                 | 4,311 | 43.11%                  | 4,308                 | 47.87%                     |
| Brčko distrikt                   | 94    | 0.94%                   | 94                    | 1.04%                      |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ba_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Bosnia and Herzegovina dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shapes reflect Bosnia and Herzegovina's asymmetric structure: Federation cantons appear at level 5, while Republika Srpska and Brčko paths often skip that level.
3. Sparse level-8 and partial shapes are small and policy-accepted.
4. Nearby fallback is bounded and accounts for the only meaningful rescue effect.
5. Level 9 settlements and level 10 neighborhoods are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `bd0b5f98fe9b6e6b33a818873ec288b04874f80d`
- Cadis version:
  `0.8.25`
- Boundary builder: `scripts/build_ba_boundaries.py`
- Build boundary: `tmp/ba_country.json`
- Evaluation boundary: `tmp/ba_country.json`
- Staged dataset: `BA/ba.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ba.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Bosnia and Herzegovina levels 4, 5, 6, 7, and 8

The dataset is suitable for human quality review.
