# RS_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `rs.admin`
Version: `v1.0.1`
Country: `RS`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `rs.admin v1.0.1` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Serbia dataset scope
* Provides reproducible integrity metrics for release review

Version `v1.0.1` intentionally removes dense level-9 locality polygons from the runtime dataset. The level-9 OSM data is valid, but it creates an oversized package relative to the Serbia source extract and is not needed for the runtime administrative contract.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `rs.admin`        |
| Dataset Version         | `v1.0.1`          |
| Country                 | `RS`              |
| Country Name            | `Serbia`          |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.160`        |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_rs_boundaries.py`       |
| Generated Boundary       | `tmp/rs_country.json`                  |
| Boundary Source          | `Natural Earth admin-0`                |
| Boundary Selection Rule  | `Full Natural Earth RS admin-0 country boundary.` |
| Boundary BBox            | `[18.84497847500006, 42.23494482000011, 22.98457076000011, 46.17387522400013]` |
| OSM Source               | `geofabrik:europe/serbia`              |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

---

# 4. Administrative Model

The reduced Serbia engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 6     | `admin_district`     | 25            | Districts and Belgrade |
| 8     | `admin_municipality` | 147           | Municipalities and cities |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |
| 4     | 2           | Excluded as broad regional coverage |
| 7     | 28          | Excluded as small city overlay coverage |
| 9     | 4,694       | Excluded from `v1.0.1` for package-size discipline |
| 10    | 329         | Excluded as sparse neighborhood detail |

The engine uses canonical names from `name:sr-Latn`, `name:sr`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `en`, `hr`, `hu`, `ro`, `sr`, and `sr-latn`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `RS/rs.admin/v1.0.1`

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
| Throughput                | `11989.260` QPS |
| Total Runtime             | `0.834 sec`     |

This run confirms no policy or inside-coverage failures after removing level-9 locality polygons.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,007 / 16 / 977 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,007 / 16 / 977 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,007 / 16 / 977 / 0 |
| no_nearby    | 98.70%    | 98.56%           | 130    | 130           | 8,855 / 16 / 1,129 / 0 |
| osm_only     | 98.70%    | 98.56%           | 130    | 130           | 8,855 / 16 / 1,129 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 152             |
| Total vs OSM-only | 152             |

## Interpretation

* OSM-only success rate: `98.70%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves boundary-adjacent and municipality-edge samples that otherwise miss polygon containment.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `rs.admin v1.0.1`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape | Count |
| ----- | ----: |
| `[6,8]` | 6,698 |
| `[6]`   | 2,295 |
| `[]`    | 991   |
| `[8]`   | 16    |

Empty shapes correspond to:

* `empty_shape`: 977
* `offshore`: 14

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,871 |
| nearby        | 138   |
| admin_tree_id | 9     |

## 9.3 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,009 |
| empty_shape      | 977   |
| offshore         | 14    |

---

# 10. Level-4 Coverage

No level-4 city hit rates are reported for this dataset because the Serbia runtime contract does not expose level 4.

The primary top-level runtime parent is level 6.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/rs_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Serbia dataset scope.

---

# 12. Structural Observations

1. Geometry integrity remains high after reducing the runtime contract to levels 6 and 8.
2. The dominant lookup shape is `[6,8]`, reflecting district and municipality coverage.
3. Level 9 was excluded from `v1.0.1` because 4,694 locality polygons made the package disproportionate to the Serbia source extract.
4. The package is now `0.1 MB` compressed and `0.2 MB` unpacked, down from the prior locality-heavy release.
5. Nearby fallback remains bounded and accounts for a moderate rescue effect around polygon edges.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `7dc998f10728c440c9f02d0e555a50b877470ec6`
- Cadis version:
  `0.8.160`
- Runtime compatibility minimum:
  `0.8.35`
- Source OSM SHA256:
  `912a6eff874fd229495f48d60e5edfde7fb51512c535ff02d614fbe63803fcf6`
- Boundary builder: `scripts/build_rs_boundaries.py`
- Build boundary: `tmp/rs_country.json`
- Evaluation boundary: `tmp/rs_country.json`
- Staged dataset: `RS/rs.admin/v1.0.1`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `rs.admin v1.0.1` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity for the reduced administrative contract
* No required repair layer
* Package sizing appropriate for the Serbia source extract

This dataset is suitable for release.
