# PE_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `pe.admin`
Version: `v1.0.0`
Country: `PE`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `pe.admin v1.0.0` dataset under Cadis Runtime.

The initial Peru dataset normalizes OpenStreetMap administrative relations into a deterministic Cadis hierarchy:

* level 4: departments and constitutional province
* level 6: provinces
* level 8: districts

Cadis does not modify geography. It enforces deterministic runtime policy over the released dataset artifacts.

---

# 2. Dataset Identity

| Field                   | Value      |
| ----------------------- | ---------- |
| Dataset ID              | `pe.admin` |
| Dataset Version         | `v1.0.0`   |
| Country                 | `PE`       |
| Country Name            | `Peru`     |
| Policy Version          | `1.0`      |
| Hierarchy Required      | `True`     |
| Repair Required         | `False`    |
| Runtime Policy Detected | `True`     |
| Cadis Version Evaluated | `0.8.23`   |

Runtime compatibility declared by the staged release manifest:

| Field         | Value    |
| ------------- | -------- |
| Minimum Cadis | `0.8.23` |
| Max Exclusive | `1.0.0`  |

---

# 3. Dataset Scope

Dataset scope: Peru administrative coverage derived from OSM admin levels 4 and 6 within the PE Natural Earth admin-0 boundary.

Boundary builder:

* `scripts/build_pe_boundaries.py`

Generated scoped boundary:

* `tmp/pe_country.json`

Deterministic selection rule:

* build the Natural Earth PE admin-0 boundary
* scan the Peru OSM PBF for relation-based administrative areas at levels 4 and 6
* keep only administrative relation areas whose representative point is covered by the Natural Earth PE boundary
* union the selected level-4 and level-6 geometries into the build/evaluation scope boundary

Boundary builder output:

| Level | Included Units |
| ----- | -------------- |
| 4     | 25             |
| 6     | 196            |

Generated scoped boundary bbox:

`[-81.3283968, -18.3501167, -68.6519906, -0.0392818]`

Raw hierarchy probe evidence included neighboring-country relations from Ecuador, Bolivia, Chile, and Brazil. The scoped boundary prevents those relations from defining the released Peru dataset.

The same scoped boundary was used for both build and evaluation.

---

# 4. Staged Dataset Artifacts

Staged dataset path:

`PE/pe.admin/v1.0.0`

Release files:

| File                    | Size      |
| ----------------------- | --------- |
| `geometry.ffsf`         | 1.1 MB    |
| `geometry_meta.json`    | 524 KB    |
| `hierarchy.json`        | 275 KB    |
| `runtime_policy.json`   | 1.2 KB    |

Structural artifact counts:

| Level | Count |
| ----- | ----: |
| 4     | 25    |
| 6     | 196   |
| 8     | 1,888 |

All `2,109` geometry metadata records have `country_scope_flag=true`.

---

# 5. Test Methodology

Mass validation used the SOP staging command with evaluation enabled.

Sampling:

* total samples: `10,000`
* inside points: `9,000`
* outside points: `1,000`
* boundary source: generated PE scoped boundary JSON
* lookup mode: Cadis runtime

The test intentionally injects outside-boundary samples to validate boundary rejection, offshore classification, and cross-border isolation.

---

# 6. Performance Metrics

| Metric                    | Value          |
| ------------------------- | -------------- |
| Throughput                | `2796.980` QPS |
| Total Runtime             | `3.575 sec`    |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |

No policy failures, inside coverage failures, or HTTP/runtime failures were observed.

---

# 7. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (`ok`/`partial`/`failed`) |
| ------------ | --------: | ---------------: | -----: | ------------: | --------------------------------: |
| full_policy  | 100.00%   | 100.00%          | 0      | 0             | 8,997 / 12 / 991                  |
| no_hierarchy | 100.00%   | 100.00%          | 0      | 0             | 8,997 / 12 / 991                  |
| no_repair    | 100.00%   | 100.00%          | 0      | 0             | 8,997 / 12 / 991                  |
| no_nearby    | 100.00%   | 100.00%          | 0      | 0             | 8,988 / 12 / 1,000                |
| osm_only     | 100.00%   | 100.00%          | 0      | 0             | 8,988 / 12 / 1,000                |

Layer contribution:

| Layer             | Rescued Samples |
| ----------------- | --------------: |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 9               |
| Total vs OSM-only | 9               |

Interpretation:

* OSM polygon coverage is already strong for the sampled Peru scope.
* Hierarchy and repair layers did not need to rescue sampled lookups.
* Nearby behavior only affected 9 offshore/near-boundary samples.

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape   | Count |
| ------- | ----: |
| [4,6,8] | 8,988 |
| []      | 1,000 |
| [4,6]   | 8     |
| [4,8]   | 3     |
| [4]     | 1     |

Empty shapes correspond to expected outside-boundary samples:

* `empty_shape`: 991
* `offshore`: 9

## 8.2 Node Source Distribution

| Source        | Count |
| ------------- | ----: |
| polygon       | 9,000 |
| admin_tree_id | 7     |

Source mix:

| Mix                   | Count |
| --------------------- | ----: |
| polygon               | 8,993 |
| none                  | 1,000 |
| admin_tree_id\|polygon | 7     |

---

# 9. Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----: |
| shape_status_map | 9,000 |
| empty_shape      | 991   |
| offshore         | 9     |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `24`
* Total level-4 hits across all samples: `9,000`
* Total level-4 hits across inside samples: `9,000`

Top sampled level-4 units:

| Level-4 Unit    | Hits | Hit Rate |
| --------------- | ---: | -------: |
| Loreto          | 2,585 | 25.85%   |
| Ucayali         | 685   | 6.85%    |
| Madre de Dios   | 600   | 6.00%    |
| Cusco           | 527   | 5.27%    |
| Puno            | 477   | 4.77%    |
| Arequipa        | 465   | 4.65%    |
| San Martín      | 399   | 3.99%    |
| Ayacucho        | 311   | 3.11%    |
| Junín           | 308   | 3.08%    |
| Amazonas        | 277   | 2.77%    |

Hit distribution reflects uniform land-area sampling, not population weighting.

---

# 11. Direct Lookup Smoke Checks

Two direct lookups were run against the staged cache using local Cadis `0.8.23` support for `PE`.

| Point | Result |
| ----- | ------ |
| `-12.0464,-77.0428` | `Peru / Lima / Lima / Lima` |
| `-13.53195,-71.96746` | `Peru / Cusco / Cusco / Wanchaq` |

Both returned `lookup_status=ok` and `resolution_state=resolved`.

---

# 12. Reproducibility

Staged release manifest:

- cadis-dataset-engine commit:
  `dca45a858b27fce17b501f4d6d65edc7af5a1033`
- Cadis version:
  `0.8.23`
* dataset engine commit: `dca45a858b27fce17b501f4d6d65edc7af5a1033`
* OSM source: `geofabrik:south-america/peru`
* OSM replication timestamp: `2026-05-09T20:21:02Z`
* OSM file SHA256: `71ea15e33ea1d202256256d0f98e83c1ab60f296ce655c62d1a195c2adb05a07`

---

# 13. Conclusion

The `pe.admin v1.0.0` staged dataset is acceptable for release candidate review.

Observed results:

* 100% overall pass rate
* 100% inside coverage pass rate
* strict boundary isolation under mixed inside/outside sampling
* no repair layer required
* strong direct polygon coverage for the sampled scope
* sensible direct lookup results for Lima and Cusco

