# GT_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `gt.admin`
Version: `v1.0.0`
Country: `GT`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `gt.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Guatemala dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `gt.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `GT`              |
| Country Name            | `Guatemala`       |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.56`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `administrative coverage`              |
| Boundary Builder         | `scripts/build_gt_boundaries.py`       |
| Generated Boundary       | `tmp/gt_country.json`                  |
| Boundary Source          | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at levels 4 and 6 whose representative point is covered by the Natural Earth GT boundary, then apply a 0.002 degree inward buffer for deterministic runtime-edge evaluation.` |
| Boundary BBox            | `[-92.22480650000438, 13.742597815233212, -88.21556223392024, 17.814594571234473]` |
| OSM Source               | `geofabrik:central-america/guatemala`  |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

The v1.0.0 scope is Guatemala administrative coverage represented by materialized OSM department and municipality relations. The country envelope is excluded from the initial runtime contract.

---

# 4. Administrative Model

The initial Guatemala engine exposes these OSM administrative levels:

| Level | Runtime Label        | Dataset Count | Notes |
| ----: | -------------------- | ------------: | ----- |
| 4     | `admin_department`   | 22            | Department-level coverage |
| 6     | `admin_municipality` | 316           | Municipality-level coverage |
| 8     | `admin_zone`         | 43            | Zone-level coverage where available |
| 10    | `admin_detail`       | 1             | Detail coverage where available |

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
* Dataset path: `GT/gt.admin/v1.0.0`

The test intentionally injects about 10% out-of-country points to validate boundary rejection behavior, offshore classification, policy-layer containment, and cross-border isolation.

Sampling is uniform over scoped administrative coverage, not population-weighted.

---

# 6. Evaluation Results

| Metric                    | Value           |
| ------------------------- | --------------- |
| Overall Pass Rate         | `100.00%`       |
| Inside Coverage Pass Rate | `100.00%`       |
| Policy Pass Rate          | `100.00%`       |
| Failed Samples            | `0`             |
| Throughput                | `11314.820` QPS |
| Total Runtime             | `0.884 sec`     |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,028 / 0 / 972 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,028 / 0 / 972 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,028 / 0 / 972 / 0 |
| no_nearby    | 99.99%    | 99.99%           | 1      | 1             | 9,000 / 0 / 1,000 / 0 |
| osm_only     | 99.99%    | 99.99%           | 1      | 1             | 9,000 / 0 / 1,000 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 28              |
| Total vs OSM-only | 28              |

Nearby fallback resolves boundary-adjacent and local geometry-edge samples that otherwise miss polygon containment. No explicit repair layer is required for `gt.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[4,6]`   | 8,710 |
| `[]`      | 999   |
| `[4]`     | 263   |
| `[4,6,8]` | 28    |

Empty shapes correspond to:

* `empty_shape`: 972
* `offshore`: 27

## 9.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 9,000 |
| admin_tree_id | 17    |
| nearby        | 1     |

## 9.3 Source Mix Distribution

| Mix                    | Count |
| ---------------------- | ----: |
| polygon                | 8,983 |
| none                   | 999   |
| admin_tree_id\|polygon | 17    |
| nearby                 | 1     |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,001 |
| empty_shape      | 972   |
| offshore         | 27    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `22`
* Total level-4 hits across all samples: `9,001`
* Total level-4 hits across inside samples: `9,000`

| Level-4 Unit | Hits | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------ | ---: | ----------------------: | --------------------: | -------------------------: |
| Petén | 2,931 | 29.31% | 2,931 | 32.57% |
| Alta Verapaz | 843 | 8.43% | 843 | 9.37% |
| Izabal | 763 | 7.63% | 763 | 8.48% |
| Quiché | 625 | 6.25% | 625 | 6.94% |
| Huehuetenango | 593 | 5.93% | 593 | 6.59% |
| Escuintla | 377 | 3.77% | 377 | 4.19% |
| San Marcos | 292 | 2.92% | 292 | 3.24% |
| Santa Rosa | 274 | 2.74% | 274 | 3.04% |
| Jutiapa | 272 | 2.72% | 272 | 3.02% |
| Baja Verapaz | 254 | 2.54% | 254 | 2.82% |
| Zacapa | 223 | 2.23% | 223 | 2.48% |
| Chiquimula | 206 | 2.06% | 206 | 2.29% |
| Quetzaltenango | 184 | 1.84% | 184 | 2.04% |
| Departamento de Guatemala | 176 | 1.76% | 176 | 1.96% |
| Jalapa | 170 | 1.70% | 170 | 1.89% |
| Suchitepéquez | 164 | 1.64% | 164 | 1.82% |
| Retalhuleu | 159 | 1.59% | 158 | 1.76% |
| El Progreso | 155 | 1.55% | 155 | 1.72% |
| Chimaltenango | 135 | 1.35% | 135 | 1.50% |
| Sololá | 87 | 0.87% | 87 | 0.97% |
| Totonicapán | 69 | 0.69% | 69 | 0.77% |
| Sacatepéquez | 49 | 0.49% | 49 | 0.54% |

Hit distribution reflects uniform sampling over the scoped administrative coverage.

---

# 11. Boundary Isolation Validation

The mixed test includes 1,000 outside samples. Under full policy:

| Metric | Value |
| ------ | ----: |
| Outside Samples | 1,000 |
| Outside Policy Failures | 0 |
| Offshore Samples | 27 |
| Failed Samples | 0 |

The result confirms the scoped Guatemala dataset rejects out-of-scope points deterministically while permitting explicit offshore classification near the boundary where allowed by runtime policy.

---

# 12. Integrity Summary

| Field | Value |
| ----- | ----- |
| Engine Commit | `2cc6a276d457074a19686a109eb74ca635d9a4e2` |
| Source OSM SHA256 | `b619e2b035b16591b4860a95e5ff34f93435e4ada7b3a5339bfe364dad926888` |
| `geometry.ffsf` SHA256 | `509c493bbe0f4f2e3955b9a141d18ed483241b1ad9ed552c8c0e1114ac6bf116` |
| `geometry_meta.json` SHA256 | `8ac7525b904a54600fe501e3382b9e7c0a9fd4d840aa65179a344548da7cc8b5` |
| `hierarchy.json` SHA256 | `f0f9dfb680b21bb165319272b1fbdec501eb63bea11defcddbf74e52aa0a6ef6` |
| `runtime_policy.json` SHA256 | `66ce32e073ebc575fe55d6d4f571d9ae0b98a3cf35fd045ddff3b068e1446b74` |

---

# 13. Review Conclusion

`gt.admin v1.0.0` passes the SOP evaluation with 100.00% overall, policy, and inside-coverage pass rates.

The dataset is structurally consistent, deterministic under policy comparison, and scoped to materialized Guatemala administrative coverage. It is suitable for release once the report is reviewed and approved.
