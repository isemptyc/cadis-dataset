# NZ_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `nz.admin`
Version: `v1.0.0`
Country: `NZ`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `nz.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the selected New Zealand administrative model
* Documents the scoped build/evaluation boundary
* Quantifies runtime lookup behavior under mixed inside/outside sampling
* Validates policy-layer containment and nearby fallback behavior
* Records reproducible integrity metrics for the release candidate

Cadis does not modify geography.
It enforces structural determinism.

---

# 2. Dataset Identity

| Field                   | Value      |
| ----------------------- | ---------- |
| Dataset ID              | `nz.admin` |
| Dataset Version         | `v1.0.0`   |
| Country                 | `NZ`       |
| Policy Version          | `1.0`      |
| Hierarchy Required      | `True`     |
| Repair Required         | `False`    |
| Runtime Policy Detected | `True`     |
| Cadis Version           | `0.8.1`    |

---

# 3. Dataset Scope

The New Zealand release uses a tracked scoped-boundary builder:

* Boundary source: OSM level-4 administrative relation geometry from the Geofabrik New Zealand extract
* Deterministic selection rule: union OSM administrative relation areas at level 4 from the New Zealand source PBF

Included level-4 units:

* `Auckland`
* `Bay of Plenty`
* `Canterbury`
* `Chatham Islands`
* `Gisborne`
* `Hawke's Bay`
* `Manawatū-Whanganui`
* `Marlborough`
* `Nelson`
* `Northland`
* `Otago`
* `Southland`
* `Taranaki`
* `Tasman`
* `Waikato`
* `Wellington`
* `West Coast`

Major Natural Earth sovereign outlying components excluded from this initial administrative-coverage scope:

* `Auckland Islands`
* `Campbell Island / Motu Ihupuku`
* `Kermadec Islands`

Scoped boundary bbox:

```json
[-177.2446864, -47.7240454, 178.8362541, -34.1935434]
```

This scope avoids sampling Natural Earth sovereign island components that do not have matching OSM level-4 administrative coverage in the initial dataset.

---

# 4. Administrative Model

The engine exposes these levels:

| Level | Label |
| ----- | ----- |
| 4     | Region |
| 6     | Territorial authority |
| 8     | Local board / community |

Level 10 exists in the source data but is intentionally excluded from `v1.0.0` because it is sparse and concentrated in a small number of local areas.

Build-stage structural summary:

| Metric | Value |
| ------ | ----- |
| Node count | `131` |
| Edge count | `114` |
| Level 4 count | `17` |
| Level 6 count | `67` |
| Level 8 count | `47` |
| Unresolved samples | `0` |

Canonical naming prefers `name:en`, then `name`, then `name:mi`, then `official_name`.
Bounded multilingual aliases are enabled for `en` and `mi`.

---

# 5. Test Methodology

* Total samples: `100,000`
* Inside samples: `90,000`
* Outside samples: `10,000`
* Lookup mode: Cadis runtime
* Evaluation boundary: `tmp/nz_country.json`
* Evaluation workers: `1`

The test intentionally injects outside-boundary points to validate boundary rejection behavior and policy-layer containment.

---

# 6. Performance Metrics

| Metric                    | Value          |
| ------------------------- | -------------- |
| Throughput                | `7635.740` QPS |
| Total Runtime             | `13.096 sec`   |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |

No failed samples were reported.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed |
| ------------ | --------- | ---------------- | ------ |
| full_policy  | 100.00%   | 100.00%          | 0      |
| no_hierarchy | 100.00%   | 100.00%          | 0      |
| no_repair    | 100.00%   | 100.00%          | 0      |
| no_nearby    | 99.77%    | 99.74%           | 231    |
| osm_only     | 99.77%    | 99.74%           | 231    |

Layer contribution:

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 241             |
| Total vs OSM-only | 241             |

Hierarchy data is required by policy, but this random sample did not require hierarchy supplementation for coverage.
Nearby fallback handled coastal adjacency cases.
No repair layer is required.

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape   | Count |
| ------- | ----- |
| [4,6]   | 49,606 |
| [4]     | 26,688 |
| [4,6,8] | 13,686 |
| []      | 10,000 |
| [4,8]   | 20     |

The empty shapes correspond to expected outside-boundary samples:

| Reason | Count |
| ------ | ----- |
| empty_shape | 9,990 |
| offshore | 10 |

## 8.2 Node Source Distribution

| Source | Count |
| ------ | ----- |
| polygon | 89,769 |
| nearby | 231 |
| admin_tree_id | 67 |

## 8.3 Source Mix Distribution

| Mix | Count |
| --- | ----- |
| polygon | 89,702 |
| none | 10,000 |
| nearby | 231 |
| admin_tree_id\|polygon | 67 |

---

# 9. Level-4 Coverage

* Unique level-4 units hit: `17`
* Total level-4 hits: `90,000`
* Total level-4 hits for inside samples: `90,000`

Top level-4 hit counts:

| Level-4 Unit | Hits |
| ------------ | ---- |
| Canterbury | 12,155 |
| Southland | 11,855 |
| Otago | 8,118 |
| West Coast | 7,328 |
| Chatham Islands | 6,633 |
| Waikato | 6,633 |
| Northland | 5,653 |
| Manawatū-Whanganui | 5,003 |
| Bay of Plenty | 4,284 |
| Hawke's Bay | 4,203 |

Hit distribution reflects uniform area sampling, not population weighting.

---

# 10. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* All 90,000 inside-boundary samples returned covered results.
* All 10,000 outside-boundary samples were handled as failed/empty or offshore outcomes.
* No policy failure or coverage failure was observed.
* Nearby fallback did not create cross-boundary leakage in this sampled run.

This confirms boundary containment within the scoped NZ administrative coverage.

---

# 11. Reproducibility

Dataset package was generated from:

* cadis-dataset-engine commit:
  `4fe22785aaa899299948ef0643e33862f2c79c84`
* Cadis version:
  `0.8.1`
* OSM source:
  `geofabrik:oceania/New-Zealand`
* OSM replication timestamp:
  `2026-03-31T20:21:06Z`
* OSM file SHA256:
  `4d36de26ca4c86666009909087c00f272cca20d18bc359b8c5baf9a3c384e1a2`

---

# 12. Conclusion

The `nz.admin v1.0.0` dataset is promotion-ready for the scoped New Zealand OSM level-4 administrative coverage:

* 100% overall pass rate
* 100% inside coverage pass rate
* 100% policy pass rate
* No failed samples
* No repair layer required
* Scoped boundary explicitly documents excluded outlying components

Proceed to official dataset publish only after accepting this scoped-release policy.
