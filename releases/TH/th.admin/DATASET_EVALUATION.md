# TH_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `th.admin`
Version: `v1.0.0`
Country: `TH`
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates the `th.admin v1.0.0` dataset under Cadis Runtime.

The evaluation validates:

* Thailand runtime policy behavior for the selected administrative model
* Boundary and offshore behavior under mixed inside/outside sampling
* Dataset-scope consistency between build and evaluation boundaries
* Multilingual Thai/English name extraction

OSM data is not modified by Cadis. The dataset encodes a deterministic runtime interpretation of available administrative geometry and hierarchy.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `th.admin` |
| Dataset Version | `v1.0.0` |
| Country | `TH` |
| Country Name | `Thailand` |
| Policy Version | `1.0` |
| Cadis Version | `0.8.11` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Minimum Cadis Version | `0.8.11` |

---

# 3. Dataset Scope

Scope: `OSM admin-level-4 coverage union`

The source OSM extract is:

* `geofabrik:asia/thailand`

The build and evaluation both use a tracked boundary builder:

* Natural Earth source: `ne_10m_admin_0_countries.dbf`
* OSM source: `thailand-latest.osm.pbf`
* Selection rule: union all extracted Thailand `admin_level=4` province polygons after Natural Earth `TH` admin-0 filtering
* Boundary bbox: `[97.3438072, 5.613522, 105.636812, 20.4648135]`
* Admin-level-4 polygon count: `65`

This v1.0.0 scope intentionally evaluates the same OSM admin-level-4 coverage union used for the build. Plain Natural Earth `TH` admin-0 sampling includes small coastal/island components where the OSM administrative polygon coverage does not produce a runtime admin hierarchy; using the OSM admin-level-4 union avoids publishing a report whose sampling boundary is broader than the actual dataset scope.

---

# 4. Runtime Policy

| Policy Field | Value |
| --- | --- |
| Allowed Levels | `[4, 6, 8, 10]` |
| Hierarchy Parent Level | `4` |
| Hierarchy Child Levels | `[6, 8, 10]` |
| Repair Layer | Disabled |
| Nearby Policy | Enabled |
| Nearby Max Distance | `2.0 km` |
| Offshore Max Distance | `20.0 km` |
| Name Schema | `multilingual_v1` |
| Alias Languages | `th`, `en` |

Level meanings:

| Level | Label |
| --- | --- |
| 4 | Province / Bangkok |
| 6 | District |
| 8 | Subdistrict |
| 10 | Neighborhood |

Level 7 local-authority relations were observed in the raw OSM hierarchy probe, but were not included in v1.0.0 because they were mostly disconnected from the province/district/subdistrict hierarchy and would introduce a separate overlapping administrative model.

---

# 5. Test Methodology

| Field | Value |
| --- | --- |
| Total samples | `50,000` |
| Inside ratio | `0.9` |
| Inside samples | `45,000` |
| Outside samples | `5,000` |
| Lookup mode | Runtime |

The test intentionally injects approximately 10% out-of-scope points to validate boundary rejection, offshore classification, and cross-border isolation.

---

# 6. Performance Metrics

| Metric | Value |
| --- | --- |
| Throughput | `12257.990` QPS |
| Total Runtime | `4.079 sec` |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| --- | ---: | ---: | ---: | ---: |
| `full_policy` | 100.00% | 100.00% | 0 | 0 |
| `no_hierarchy` | 100.00% | 100.00% | 0 | 0 |
| `no_repair` | 100.00% | 100.00% | 0 | 0 |
| `no_nearby` | 100.00% | 100.00% | 0 | 0 |
| `osm_only` | 100.00% | 100.00% | 0 | 0 |

---

# 8. Layer Contribution Analysis

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | 0 |
| Repair | 0 |
| Nearby | 62 |
| Total vs OSM-only | 62 |

The hierarchy layer is still required because the runtime dataset ships `hierarchy.json`, but this sampled run did not require hierarchy-only rescue to pass coverage. No repair layer is required.

---

# 9. Structural Distribution

The shape distribution is intentionally dominated by `[4,6]`. This is a source-data coverage characteristic, not a build-time exclusion of lower levels.

Build artifacts include level 8 and level 10 polygons, and the runtime hierarchy has parent chains for all emitted level 6, 8, and 10 nodes. However, the union area of lower-level polygons is thin relative to the dataset scope:

| Level | Built Polygon Count | Approx. Scope Coverage |
| --- | ---: | ---: |
| 4 | 65 | 100.000% |
| 6 | 815 | 92.979% |
| 8 | 679 | 3.904% |
| 10 | 164 | 0.026% |

Therefore, low `[4,6,8]` and `[4,6,8,10]` hit rates reflect OSM lower-level polygon coverage, not hierarchy incompleteness or build-time filtering. Level 7 local-authority relations are excluded from v1.0.0 because they form an overlapping, mostly disconnected local-government model rather than the territorial `4 -> 6 -> 8 -> 10` hierarchy.

## 9.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[4,6]` | 40,568 |
| `[]` | 4,991 |
| `[4]` | 2,629 |
| `[4,6,8]` | 1,795 |
| `[4,8]` | 9 |
| `[4,6,8,10]` | 8 |

Empty-shape outcomes correspond to expected out-of-scope or offshore samples:

| Reason | Count |
| --- | ---: |
| `empty_shape` | 4,936 |
| `offshore` | 55 |

## 9.2 Node Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | 45,002 |
| `nearby` | 7 |
| `admin_tree_id` | 2 |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `65`
* Total level-4 hits, all samples: `45,009`
* Total level-4 hits, inside samples: `45,000`

Top sampled level-4 units by inside hit count:

| Level-4 Unit | Inside Hits |
| --- | ---: |
| `จังหวัดเชียงใหม่` | 2,100 |
| `จังหวัดนครราชสีมา` | 1,963 |
| `จังหวัดกาญจนบุรี` | 1,768 |
| `จังหวัดตาก` | 1,554 |
| `จังหวัดอุบลราชธานี` | 1,441 |
| `จังหวัดสุราษฎร์ธานี` | 1,252 |
| `จังหวัดแม่ฮ่องสอน` | 1,224 |
| `จังหวัดลำปาง` | 1,216 |
| `จังหวัดชัยภูมิ` | 1,184 |
| `จังหวัดเพชรบูรณ์` | 1,125 |

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-scope samples:

* No inside-boundary coverage failure was observed.
* No policy failure was observed.
* Empty-shape and offshore outcomes were limited to expected out-of-scope samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation in this run.

---

# 12. Reproducibility

All dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit: `60e55d634557f1104a53d756a4326a6f854225f2`
* Cadis version: `0.8.11`

The cadis-dataset-engine build was generated from a clean committed engine tree.

---

# 13. Conclusion

The `th.admin v1.0.0` dataset is acceptable for staging under the declared OSM admin-level-4 coverage scope.

The evaluation shows:

* 100% overall pass rate
* 100% inside coverage pass rate
* 100% policy pass rate
* No repair requirement
* Full sampled coverage of all 65 level-4 Thailand units
* Deterministic Thai/English multilingual naming support
