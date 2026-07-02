# TH_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `th.admin`  
Version: `v1.0.1`  
Country: `TH`  
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates the rebuilt `th.admin v1.0.1` dataset under Cadis Runtime.

This rebuild fixes the `v1.0.0` scope defect where Thailand coastal admin-level-4 provinces were filtered out before runtime export. The triggering regression was Suvarnabhumi Airport at `13.683897, 100.745262`, which previously returned `Thailand / offshore` with no admin hierarchy.

OSM data is not modified by Cadis. The dataset encodes a deterministic runtime interpretation of available administrative geometry and hierarchy.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `th.admin` |
| Dataset Version | `v1.0.1` |
| Country | `TH` |
| Country Name | `Thailand` |
| Policy Version | `1.0` |
| Cadis Version | `0.10.4` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Minimum Cadis Version | `0.10.4` |

---

# 3. Dataset Scope

Scope: `OSM admin-level-4 coverage union`

The source OSM extract is:

* `geofabrik:asia/thailand`

The tracked boundary builder is:

* `scripts/build_th_boundaries.py`

Boundary generation uses:

* Natural Earth source: `ne_10m_admin_0_countries.dbf`
* OSM source: `thailand-latest.osm.pbf`
* Selection rule: union all extracted Thailand admin_level=4 province polygons from the Geofabrik Thailand PBF; Natural Earth TH is retained as provenance only
* Boundary bbox: `[97.3438072, 5.613522, 105.636812, 20.4648135]`
* Admin-level-4 polygon count: `77`

`v1.0.1` intentionally keeps Natural Earth as provenance only for this scope. It does not filter Thailand provinces by Natural Earth representative-point containment, because coastal province multipolygons can have representative points outside the Natural Earth land polygon.

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

Feature counts in the staged runtime artifacts:

| Level | Built Polygon Count |
| --- | ---: |
| `4` | `77` |
| `6` | `927` |
| `8` | `750` |
| `10` | `164` |

---

# 5. Regression Validation

The Suvarnabhumi Airport coordinate now resolves inside the staged dataset:

| Feature ID | Level | Name | English Name | Parent ID |
| --- | ---: | --- | --- | --- |
| `th_r1908815` | `4` | `จังหวัดสมุทรปราการ` | `Samut Prakan Province` | `` |
| `th_r11580982` | `6` | `อำเภอบางพลี` | `Bang Phli District` | `th_r1908815` |
| `th_r13005046` | `8` | `ตำบลหนองปรือ` | `Nong Prue Subdistrict` | `th_r11580982` |
| `th_r13005044` | `8` | `ตำบลราชาเทวะ` | `Racha Thewa Subdistrict` | `th_r11580982` |

Runtime lookup for `13.683897, 100.745262` returns:

| Rank | Level | Feature ID | English Name | Source |
| --- | ---: | --- | --- | --- |
| `0` | `4` | `th_r1908815` | `Samut Prakan Province` | `polygon` |
| `1` | `6` | `th_r11580982` | `Bang Phli District` | `polygon` |
| `2` | `8` | `th_r13005046` | `Nong Prue Subdistrict` | `polygon` |

Note: OSM `Racha Thewa Subdistrict` (`th_r13005044`) is present in the dataset and is a sibling of `Nong Prue Subdistrict`, but the exact test coordinate falls inside `Nong Prue` according to the source OSM polygon geometry.

---

# 6. Test Methodology

| Field | Value |
| --- | --- |
| Total samples | `10,000` |
| Inside ratio | `0.9` |
| Inside samples | `9,000` |
| Outside samples | `1,000` |
| Lookup mode | `Runtime` |
| Evaluation boundary | `tmp/th_country.json` |

The test intentionally injects approximately 10% out-of-scope points to validate boundary rejection, offshore classification, and cross-border isolation.

---

# 7. Performance Metrics

| Metric | Value |
| --- | ---: |
| Throughput | `10879.910 QPS` |
| Total Runtime | `0.919 sec` |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |

---

# 8. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed |
| --- | ---: | ---: | ---: | ---: |
| `full_policy` | `100.00%` | `100.00%` | `0` | `0` |
| `no_hierarchy` | `100.00%` | `100.00%` | `0` | `0` |
| `no_repair` | `100.00%` | `100.00%` | `0` | `0` |
| `no_nearby` | `100.00%` | `100.00%` | `0` | `0` |
| `osm_only` | `100.00%` | `100.00%` | `0` | `0` |

---

# 9. Layer Contribution Analysis

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | `0` |
| Repair | `0` |
| Nearby | `15` |
| Total vs OSM-only | `15` |

---

# 10. Structural Distribution

## 10.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[4,6]` | `7070` |
| `[4]` | `1607` |
| `[]` | `997` |
| `[4,6,8]` | `324` |
| `[4,8]` | `1` |
| `[4,8,10]` | `1` |

## 10.2 Node Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | `9000` |
| `nearby` | `3` |

## 10.3 Source Mix Distribution

| Source Mix | Count |
| --- | ---: |
| `polygon` | `9000` |
| `__none__` | `997` |
| `nearby` | `3` |

## 10.4 Policy Reason Distribution

| Reason | Count |
| --- | ---: |
| `shape_status_map` | `9003` |
| `empty_shape` | `985` |
| `offshore` | `12` |

---

# 11. Level-4 Coverage

* Unique level-4 units hit: `77`
* Total level-4 hits, all samples: `9003`
* Total level-4 hits, inside samples: `9000`

The sampled run hit all 77 Thailand level-4 units in the rebuilt runtime scope.

---

# 12. Boundary Isolation Validation

Under stress testing with 10% forced out-of-scope samples:

* No inside-boundary coverage failure was observed.
* No policy failure was observed.
* Empty-shape and offshore outcomes were limited to expected out-of-scope samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation in this run.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit: `b05d3af01aab376aafb75fad43a31dc51817e587`
* Cadis version: `0.10.4`

The staged dataset package was generated without publishing.

---

# 14. Conclusion

The `th.admin v1.0.1` staged dataset resolves the Suvarnabhumi Airport regression and restores the missing coastal province coverage caused by the previous Natural Earth representative-point filter.

The staged evaluation shows:

* 100% overall pass rate
* 100% inside coverage pass rate
* 100% policy pass rate
* Full sampled coverage of all 77 level-4 Thailand units
* Deterministic Thai/English multilingual naming support
