# CY_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `cy.admin`
Version: `v1.0.0`
Country: `CY`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `cy.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Cyprus dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `cy.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `CY`              |
| Country Name            | `Cyprus`          |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.38`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_cy_boundaries.py`       |
| Generated Boundary       | `tmp/cy_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth CY admin-0 country boundary.` |
| Boundary BBox            | `[32.27173912900008, 34.62501943000002, 34.09913170700008, 35.18708213000012]` |
| OSM Source               | `geofabrik:europe/cyprus`              |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The initial Cyprus engine exposes these OSM administrative levels:

| Level | Runtime Label         | Dataset Count | Notes |
| ----: | --------------------- | ------------: | ----- |
| 5     | `admin_district`      | 5             | Districts |
| 6     | `admin_municipality`  | 47            | Municipalities and community clusters |
| 8     | `admin_community`     | 391           | Communities and localities |
| 9     | `admin_neighborhood`  | 71            | Neighborhood-level areas |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 3     | 1           | Excluded as country-level envelope |
| 7     | 2           | Excluded as sparse local overlay coverage |
| 10    | 0           | Excluded because no scoped features were found |

The engine uses canonical names from `name:el`, `name:tr`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `el`, `en`, and `tr`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `CY/cy.admin/v1.0.0`

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
| Throughput                | `12572.420` QPS |
| Total Runtime             | `0.795 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,106 / 2 / 892 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,106 / 2 / 892 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,106 / 2 / 892 / 0 |
| no_nearby    | 98.75%    | 98.61%           | 125    | 125           | 8,876 / 2 / 1,122 / 0 |
| osm_only     | 98.75%    | 98.61%           | 125    | 125           | 8,876 / 2 / 1,122 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 230             |
| Total vs OSM-only | 230             |

## Interpretation

* OSM-only success rate: `98.75%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves small-island, district-edge, and boundary-adjacent samples that otherwise miss polygon containment.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `cy.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape       | Count |
| ----------- | ----: |
| `[5,6,8]`   | 8,058 |
| `[]`        | 973   |
| `[5,6,8,9]` | 481  |
| `[5,8]`     | 202  |
| `[5,6]`     | 147  |
| `[5]`       | 94   |
| `[5,6,9]`   | 41   |
| `[6,8]`     | 2    |

Additional low-count shape observed in the full report was `[5,8,9]`.

Empty shapes correspond to:

* `empty_shape`: 892
* `offshore`: 81

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,878 |
| nearby        | 149   |
| admin_tree_id | 20    |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,858 |
| none                   | 973   |
| nearby                 | 149   |
| admin_tree_id\|polygon | 20    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,027 |
| empty_shape      | 892   |
| offshore         | 81    |

---

# 10. Level-4 Coverage

No level-4 city hit rates are reported for this dataset because the Cyprus runtime contract does not expose level 4.

The primary runtime parent layer is level 5 district coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/cy_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Cyprus dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[5,6,8]`, reflecting district, municipality, and community coverage.
3. Level 9 adds neighborhood-level detail where available.
4. Nearby fallback is bounded at 5 km and accounts for the expected rescue effect around island and administrative edges.
5. Levels 3, 7, and 10 are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `e77015be3484338e9b66f4655c724c5ffb6a3c9c`
- Cadis version:
  `0.8.38`
- Boundary builder: `scripts/build_cy_boundaries.py`
- Build boundary: `tmp/cy_country.json`
- Evaluation boundary: `tmp/cy_country.json`
- Staged dataset: `CY/cy.admin/v1.0.0`
- Source OSM SHA256:
  `837a85874057d3dede19cc14c23b04a733264c71c428f78216cead90b85990a9`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `cy.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Cyprus levels 5, 6, 8, and 9

The dataset is suitable for human quality review.
