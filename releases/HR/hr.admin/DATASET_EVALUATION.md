# HR_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `hr.admin`
Version: `v1.0.0`
Country: `HR`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `hr.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the source-data scope and administrative model
* Quantifies lookup behavior under mixed inside/outside stress testing
* Documents deterministic policy effects
* Validates boundary isolation for the Croatia dataset scope
* Provides reproducible integrity metrics for release review

OSM data is not incorrect.
Observed sparse outcomes reflect structural coverage or boundary-edge behavior, not geometric invalidity.

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value             |
| ----------------------- | ----------------- |
| Dataset ID              | `hr.admin`        |
| Dataset Version         | `v1.0.0`          |
| Country                 | `HR`              |
| Country Name            | `Croatia`         |
| Policy Version          | `1.0`             |
| Cadis Version           | `v0.8.27`         |
| Hierarchy Required      | `True`            |
| Repair Required         | `False`           |
| Runtime Policy Detected | `True`            |
| Name Schema             | `multilingual_v1` |

---

# 3. Dataset Scope

| Field                    | Value                                  |
| ------------------------ | -------------------------------------- |
| Scope Label              | `full ISO2 country`                    |
| Boundary Builder         | `scripts/build_hr_boundaries.py`       |
| Generated Boundary       | `tmp/hr_country.json`                  |
| Boundary Source          | `OpenStreetMap administrative relations` |
| Boundary Selection Rule  | `Union OSM administrative relation areas at level 4 whose ISO3166-2 tag starts with HR-.` |
| Boundary BBox            | `[13.2104814, 42.1765993, 19.4472713, 46.5550254]` |
| Included Level-4 Units   | `21`                                   |
| OSM Source               | `geofabrik:europe/croatia`             |
| OSM Snapshot Timestamp   | `2026-05-10T20:20:31Z`                 |

The same scoped boundary was used for both build and evaluation.

Natural Earth admin-0 representative-point filtering excluded some coastal county relations during probe work. The tracked OSM level-4 county coverage boundary is therefore the release source of truth for Croatia `v1.0.0`.

---

# 4. Administrative Model

The initial Croatia engine exposes these OSM administrative levels:

| Level | Runtime Label                | Probe Count | Notes |
| ----: | ---------------------------- | ----------: | ----- |
| 4     | `admin_county`               | 21          | Counties and the City of Zagreb |
| 7     | `admin_city_or_municipality` | 555         | Cities and municipalities |
| 8     | `admin_settlement`           | 6,754       | Settlements |

Probe evidence also found:

| Level | Probe Count | Decision |
| ----: | ----------: | -------- |
| 9     | 260         | Excluded as sparse city district / local board coverage |
| 10    | 218         | Excluded as sparse local board coverage |

The engine uses canonical names from `name:hr`, `name`, `name:en`, and `official_name`, with bounded multilingual aliases for `hr`, `it`, `sr`, `sr-latn`, and `en`.

---

# 5. Test Methodology

## 5.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside samples: `9,000`
* Outside samples: `1,000`
* Expected inside ratio: `0.9`
* Expected outside ratio: `0.1`
* Dataset path: `HR/hr.admin/v1.0.0`

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
| Throughput                | `4069.550` QPS |
| Total Runtime             | `2.457 sec`    |

This run confirms no policy or inside-coverage failures.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status Counts (`ok`/`partial`/`failed`/`unknown`) |
| ------------ | --------: | ---------------: | -----: | ------------: | -------------------------------------------------- |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 9,025 / 1 / 974 / 0 |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 9,025 / 1 / 974 / 0 |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 9,025 / 1 / 974 / 0 |
| no_nearby    | 99.87%    | 99.86%           | 13     | 13            | 8,986 / 1 / 1,013 / 0 |
| osm_only     | 99.87%    | 99.86%           | 13     | 13            | 8,986 / 1 / 1,013 / 0 |

---

# 8. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 39              |
| Total vs OSM-only | 39              |

## Interpretation

* OSM-only success rate: `99.87%`
* Full policy success rate: `100.00%`
* Nearby fallback resolves boundary-adjacent and coastal-edge samples.
* Hierarchy and repair layers were not needed to rescue failing samples in this evaluation run.
* No explicit repair layer is required for `hr.admin v1.0.0`.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Shape     | Count |
| --------- | ----: |
| `[4,7,8]` | 5,763 |
| `[4]`     | 3,149 |
| `[]`      | 995   |
| `[4,8]`   | 83    |
| `[4,7]`   | 9     |
| `[8]`     | 1     |

Empty shapes correspond to:

* `empty_shape`: 974
* `offshore`: 21

## 9.2 Node Source Distribution

| Source          | Count |
| --------------- | ----: |
| polygon         | 8,987 |
| admin_tree_id   | 22    |
| nearby          | 18    |

## 9.3 Source Mix Distribution

| Mix                   | Count |
| --------------------- | ----: |
| polygon               | 8,965 |
| none                  | 995   |
| admin_tree_id\|polygon | 22    |
| nearby                | 18    |

## 9.4 Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,005 |
| empty_shape      | 974   |
| offshore         | 21    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `21`
* Total level-4 hits across all samples: `9,004`
* Total level-4 hits across inside samples: `8,999`

| Level-4 Unit                         | Hits  | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------------------------------ | ----: | ----------------------: | --------------------: | -------------------------: |
| Splitsko-dalmatinska županija        | 1,430 | 14.30%                  | 1,430                 | 15.89%                     |
| Dubrovačko-neretvanska županija      | 865   | 8.65%                   | 863                   | 9.59%                      |
| Primorsko-goranska županija          | 821   | 8.21%                   | 821                   | 9.12%                      |
| Zadarska županija                    | 735   | 7.35%                   | 735                   | 8.17%                      |
| Istarska županija                    | 678   | 6.78%                   | 678                   | 7.53%                      |
| Ličko-senjska županija               | 617   | 6.17%                   | 617                   | 6.86%                      |
| Šibensko-kninska županija            | 551   | 5.51%                   | 551                   | 6.12%                      |
| Sisačko-moslavačka županija          | 452   | 4.52%                   | 452                   | 5.02%                      |
| Osječko-baranjska županija           | 438   | 4.38%                   | 438                   | 4.87%                      |
| Karlovačka županija                  | 394   | 3.94%                   | 393                   | 4.37%                      |

Hit distribution reflects uniform land-area sampling.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No boundary-leak failure was observed in this sampled run.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation.
* The same `tmp/hr_country.json` scoped boundary was used for build and evaluation.

This confirms strict boundary containment within the Croatia dataset scope.

---

# 12. Structural Observations

1. Geometry integrity is high; no repair layer activation was needed.
2. The dominant lookup shape is `[4,7,8]`, reflecting county, city/municipality, and settlement coverage.
3. `[4]` outcomes are common because uniform land-area sampling often lands outside settlement polygons while still resolving county scope.
4. Nearby fallback is bounded and accounts for a small rescue effect.
5. Levels 9 and 10 are intentionally excluded from the initial runtime contract.
6. Dataset achieves full inside-boundary coverage under full policy mode.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

- cadis-dataset-engine commit:
  `0c84bf637d9dff1618001d238dccf7f63369f56e`
- Cadis version:
  `0.8.27`
- Boundary builder: `scripts/build_hr_boundaries.py`
- Build boundary: `tmp/hr_country.json`
- Evaluation boundary: `tmp/hr_country.json`
- Staged dataset: `HR/hr.admin/v1.0.0`

The dataset package was generated from a clean `cadis_dataset_engine` working tree.

---

# 14. Conclusion

The `hr.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy mode
* Strict boundary isolation in mixed inside/outside stress testing
* High geometric integrity
* No required repair layer
* Bounded nearby fallback behavior
* Stable administrative coverage for Croatia levels 4, 7, and 8

The dataset is suitable for human quality review.
