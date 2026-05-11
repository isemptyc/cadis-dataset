# DO_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `do.admin`
Version: `v1.0.0`
Country: `DO`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `do.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Dominican Republic dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value                 |
| ----------------------- | --------------------- |
| Dataset ID              | `do.admin`            |
| Dataset Version         | `v1.0.0`              |
| Country                 | `DO`                  |
| Country Name            | `Dominican Republic`  |
| Policy Version          | `1.0`                 |
| Cadis Version           | `v0.8.58`             |
| Hierarchy Required      | `True`                |
| Repair Required         | `False`               |
| Runtime Policy Detected | `True`                |
| Name Schema             | `multilingual_v1`     |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_do_boundaries.py`       |
| Generated Boundary       | `tmp/do_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4 and 6 whose representative point is covered by the Natural Earth DO boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox            | `[-72.00557934988278, 17.473205421647805, -68.32556492397616, 19.930008754091745]` |
| OSM Source               | `geofabrik:central-america/haiti-and-domrep` |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Dominican Republic administrative coverage represented by materialized OSM province and municipality relations. The country envelope is excluded from the initial runtime contract.

---

# 4. Administrative Model

The initial Dominican Republic engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_province`     | 32            | Province-level coverage |
| 6     | `admin_municipality` | 155           | Municipality-level coverage |
| 10    | `admin_detail`       | 0             | Detail level exposed by policy; no scoped nodes materialized in this build |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 2     | 1           | Excluded as country-level envelope |

The engine uses canonical names from `name:es`, `name`, `official_name`, and `name:en`, with bounded multilingual aliases for `es` and `en`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `DO/do.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over scoped administrative coverage, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value          |
| ------------------------- | -------------- |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |
| Failed Samples            | `0`            |
| Throughput                | `5553.260` QPS |
| Total Runtime             | `1.801 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,023 / 0 / 977 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,023 / 0 / 977 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,023 / 0 / 977 / 0 |
| no_nearby    | 99.98%    | 99.98%           | 2      | 2             | 8,999 / 0 / 1,001 / 0 |
| osm_only     | 99.98%    | 99.98%           | 2      | 2             | 8,999 / 0 / 1,001 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 24              |
| Total vs OSM-only | 24              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `do.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape   | Count |
| ------- | ----: |
| `[4,6]` | 8,980 |
| `[]`    | 997   |
| `[4]`   | 23    |

Empty shapes correspond to:

* `empty_shape`: 977
* `offshore`: 20

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 8,999 |
| admin_tree_id | 37    |
| nearby        | 4     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,962 |
| none                   | 997   |
| admin_tree_id\|polygon | 37    |
| nearby                 | 4     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,003 |
| empty_shape      | 977   |
| offshore         | 20    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `32`
* Total level-4 hits across all samples: `9,003`
* Total level-4 hits across inside samples: `9,000`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| San Juan | 633 | 6.33% | 633 | 7.03% |
| La Altagracia | 569 | 5.69% | 569 | 6.32% |
| Santiago | 567 | 5.67% | 567 | 6.30% |
| Azua | 509 | 5.09% | 509 | 5.66% |
| La Vega | 470 | 4.70% | 470 | 5.22% |
| Monte Plata | 447 | 4.47% | 447 | 4.97% |
| Pedernales | 389 | 3.89% | 388 | 4.31% |
| Independencia | 362 | 3.62% | 362 | 4.02% |
| Monte Cristi | 361 | 3.61% | 361 | 4.01% |
| El Seibo | 329 | 3.29% | 329 | 3.66% |
| Duarte | 317 | 3.17% | 317 | 3.52% |
| Puerto Plata | 301 | 3.01% | 301 | 3.34% |
| Barahona | 263 | 2.63% | 262 | 2.91% |
| Hato Mayor | 252 | 2.52% | 252 | 2.80% |
| Santo Domingo | 247 | 2.47% | 247 | 2.74% |
| Elías Piña | 242 | 2.42% | 242 | 2.69% |
| Santiago Rodríguez | 236 | 2.36% | 236 | 2.62% |
| María Trinidad Sánchez | 235 | 2.35% | 235 | 2.61% |
| San Pedro de Macorís | 226 | 2.26% | 226 | 2.51% |
| San Cristóbal | 225 | 2.25% | 225 | 2.50% |
| Baoruco | 223 | 2.23% | 223 | 2.48% |
| Sánchez Ramírez | 206 | 2.06% | 206 | 2.29% |
| Monseñor Nouel | 202 | 2.02% | 202 | 2.24% |
| Dajabón | 197 | 1.97% | 197 | 2.19% |
| Peravia | 164 | 1.64% | 164 | 1.82% |
| Valverde | 164 | 1.64% | 164 | 1.82% |
| Espaillat | 160 | 1.60% | 160 | 1.78% |
| San José de Ocoa | 153 | 1.53% | 153 | 1.70% |
| Samaná | 137 | 1.37% | 136 | 1.51% |
| La Romana | 114 | 1.14% | 114 | 1.27% |
| Hermanas Mirabal | 83 | 0.83% | 83 | 0.92% |
| Distrito Nacional | 20 | 0.20% | 20 | 0.22% |

Hit distribution reflects uniform sampling over the scoped administrative coverage.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/do_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Dominican Republic dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,6]`, reflecting strong province and municipality coverage.
3. Level 10 is part of the runtime policy surface but has no scoped materialized nodes in this build.
4. Nearby fallback is bounded at 2 km and accounts for a small rescue effect around administrative edges.
5. Dataset achieves full inside-boundary coverage under full policy mode for the scoped administrative coverage.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `4f30c8f9a30930d6f372f4d818802cebddd34fbc`
- Cadis version:
  `0.8.58`
- Boundary builder: `scripts/build_do_boundaries.py`
- Build boundary: `tmp/do_country.json`
- Evaluation boundary: `tmp/do_country.json`
- Staged dataset: `DO/do.admin/v1.0.0`
- Source OSM SHA256:
  `4ebf62b545fa6b17e4870dabcc6b63cb37dffe24a4e82ab101c172ad12192cc7`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `do.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Dominican Republic levels 4 and 6, with level 10 reserved in policy for future scoped detail

The dataset is suitable for human quality review.
