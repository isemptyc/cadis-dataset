# EE_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ee.admin`
Version: `v1.0.0`
Country: `EE`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `ee.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Estonia dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `ee.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `EE`              |
| Country Name            | `Estonia`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.28`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_ee_boundaries.py`       |
| Generated Boundary       | `tmp/ee_country.json`                  |
| Boundary Source          | `OpenStreetMap administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at level 6 whose ISO3166-2 tag starts with EE-.` |
| Boundary BBox            | `[21.3826069, 57.5093328, 28.2100175, 59.9383333]` |
| Included Level-6 Units   | `15`                                   |
| OSM Source               | `geofabrik:europe/estonia`             |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

Natural Earth admin-0 representative-point filtering excluded some island county relations during probe work. The tracked OSM level-6 county coverage boundary is therefore the release source of truth for Estonia `v1.0.0`.

---

# 4. Administrative Model

The initial Estonia engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 6     | `admin_county`       | 15            | Counties |
| 7     | `admin_municipality` | 78            | Municipalities and cities |
| 9     | `admin_settlement`   | 4,709         | Villages, boroughs, and small settlements |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 8     | 2           | Excluded as sparse city subarea coverage after country scoping |
| 10    | 83          | Excluded as sparse neighborhood coverage |

The engine uses canonical names from `name:et`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `de`, `en`, `et`, `lt`, and `ru`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `EE/ee.admin/v1.0.0`

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
| Throughput                | `8664.900` QPS |
| Total Runtime             | `1.154 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,028 / 0 / 972 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,028 / 0 / 972 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,028 / 0 / 972 / 0 |
| no_nearby    | 99.95%    | 99.94%           | 5      | 5             | 8,995 / 0 / 1,005 / 0 |
| osm_only     | 99.95%    | 99.94%           | 5      | 5             | 8,995 / 0 / 1,005 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 33              |
| Total vs OSM-only | 33              |

## Interpretation

* OSM-only success rate: `99.95%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves a small number of boundary-adjacent and coastal-edge samples.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `ee.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[6,7,9]` | 5,482 |
| `[6]`     | 3,491 |
| `[]`      | 999   |
| `[6,7]`   | 18    |
| `[6,9]`   | 10    |

Empty shapes correspond to:

* `empty_shape`: 972
* `offshore`: 27

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,995 |
| admin_tree_id | 4     |
| nearby        | 6     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,991 |
| none                   | 999   |
| admin_tree_id\|polygon | 4     |
| nearby                 | 6     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,001 |
| empty_shape      | 972   |
| offshore         | 27    |

---

# 10. Level-6 Coverage

* Included level-6 county units: `15`
* Level-6 units are sourced by `ISO3166-2` tags beginning with `EE-`.
* The evaluation summary does not emit a dedicated level-6 hit-rate table; the observed structural shapes show county-level coverage in all non-empty outcomes.

Included level-6 units:

| Level-6 Unit |
| ------------ |
| Harju maakond |
| Hiiu maakond |
| Ida-Viru maakond |
| Järva maakond |
| Jõgeva maakond |
| Lääne maakond |
| Lääne-Viru maakond |
| Pärnu maakond |
| Põlva maakond |
| Rapla maakond |
| Saare maakond |
| Tartu maakond |
| Valga maakond |
| Viljandi maakond |
| Võru maakond |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/ee_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Estonia dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[6,7,9]`, reflecting county, municipality, and settlement coverage.
3. `[6]` outcomes are common because uniform land-area sampling often lands outside settlement polygons while still resolving county scope.
4. Nearby fallback is bounded and accounts for a small rescue effect.
5. Levels 8 and 10 are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `acbe2c8fa8336f74bcf3b9eb7dd0e6b200e768e1`
- Cadis version:
  `0.8.28`
- Boundary builder: `scripts/build_ee_boundaries.py`
- Build boundary: `tmp/ee_country.json`
- Evaluation boundary: `tmp/ee_country.json`
- Staged dataset: `EE/ee.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `ee.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Estonia levels 6, 7, and 9

The dataset is suitable for human quality review.
