# MC_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `mc.admin`
Version: `v1.0.0`
Country: `MC`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `mc.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Monaco dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `mc.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `MC`              |
| Country Name            | `Monaco`          |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.32`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_mc_boundaries.py`       |
| Generated Boundary       | `tmp/mc_country.json`                  |
| Boundary Source          | `OpenStreetMap administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at level 10 whose ISO3166-2 tag starts with MC-.` |
| Boundary BBox            | `[7.4090279, 43.7247599, 7.4398704, 43.7519173]` |
| Included Level-10 Units  | `9`                                    |
| OSM Source               | `geofabrik:europe/monaco`              |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

Natural Earth admin-0 representative-point filtering excluded some quarter relations during probe work. The tracked OSM level-10 quarter coverage boundary is therefore the release source of truth for Monaco `v1.0.0`.

---

# 4. Administrative Model

The initial Monaco engine exposes these OSM administrative levels:

| Level | Runtime Label  | Dataset Count | Notes |
| ----: | -------------- | ------------: | ----- |
| 8     | `admin_city`   | 1             | Monaco city boundary |
| 10    | `admin_quarter` | 9            | Monaco quarters |

Included level-10 units:

| Level-10 Unit |
| ------------- |
| Fontvieille |
| Jardin Exotique |
| La Condamine |
| La Rousse |
| Larvotto |
| Les Monegetti |
| Monaco-Ville |
| Monte-Carlo |
| Ravin de Sainte-Dévote |

The engine uses canonical names from `name:fr`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `ar`, `fr`, `it`, `oc`, and `ru`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `MC/mc.admin/v1.0.0`

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
| Throughput                | `13465.560` QPS |
| Total Runtime             | `0.743 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,261 / 0 / 739 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,261 / 0 / 739 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,261 / 0 / 739 / 0 |
| no_nearby    | 100.00%   | 100.00%          | 0      | 0             | 9,000 / 0 / 1,000 / 0 |
| osm_only     | 100.00%   | 100.00%          | 0      | 0             | 9,000 / 0 / 1,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 261             |
| Total vs OSM-only | 261             |

## Interpretation

* OSM-only success rate: `100.00%`
* Full policy success rate: `100.00%`
* Nearby fallback changes some expected outside classifications near Monaco's very small boundary but does not rescue failing inside samples.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `mc.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape    | Count |
| -------- | ----: |
| `[8,10]` | 9,021 |
| `[]`     | 977   |
| `[8]`    | 2     |

Empty shapes correspond to:

* `empty_shape`: 739
* `offshore`: 238

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 9,000 |
| admin_tree_id | 40    |
| nearby        | 23    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,960 |
| none                   | 977   |
| admin_tree_id\|polygon | 40    |
| nearby                 | 23    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,023 |
| empty_shape      | 739   |
| offshore         | 238   |

---

# 10. Level-10 Coverage

* Included level-10 quarter units: `9`
* Level-10 units are sourced by `ISO3166-2` tags beginning with `MC-`.
* The observed dominant shape `[8,10]` confirms quarter-level coverage for Monaco's land-area samples.

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/mc_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Monaco dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[8,10]`, reflecting city and quarter coverage.
3. The OSM-derived quarter coverage boundary is necessary to keep all Monaco quarters in scope.
4. Nearby fallback is bounded and does not affect inside coverage pass/fail behavior in this run.
5. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `e4ca98de30719ee3ea98d18f2ca4deb9dddaba5a`
- Cadis version:
  `0.8.32`
- Boundary builder: `scripts/build_mc_boundaries.py`
- Build boundary: `tmp/mc_country.json`
- Evaluation boundary: `tmp/mc_country.json`
- Staged dataset: `MC/mc.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `mc.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Stable administrative coverage for Monaco levels 8 and 10

The dataset is suitable for human quality review.
