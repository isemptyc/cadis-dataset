# PH_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `ph.admin`  
Version: `v1.0.2`  
Country: `PH`  
Policy Version: `1.0`

---

# 1. Purpose

This document evaluates the rebuilt `ph.admin v1.0.2` dataset under Cadis Runtime.

This rebuild fixes the `v1.0.1` scope defect where Philippines runtime country scope was derived only from admin-level-4 province polygons. That omitted Metro Manila / Manila, because Manila is represented in the source PBF as an admin-level-6 boundary outside province coverage. The triggering regression was `14.608753147958526, 120.97594995915485`, which previously returned `Philippines` with no admin hierarchy and an offshore source.

OSM data is not modified by Cadis. The dataset encodes a deterministic runtime interpretation of available administrative geometry and hierarchy.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `ph.admin` |
| Dataset Version | `v1.0.2` |
| Country | `PH` |
| Country Name | `Philippines` |
| Policy Version | `1.0` |
| Cadis Version | `0.10.5` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Minimum Cadis Version | `0.10.5` |
| Engine Commit | `b05d3af01aab376aafb75fad43a31dc51817e587` |

---

# 3. Dataset Scope

Scope: `OSM admin-level-4/6 coverage union`

The source OSM extract is:

* `geofabrik:asia/philippines`

The tracked boundary builder is:

* `scripts/build_ph_boundaries.py`

Boundary generation uses:

* Natural Earth source: `ne_10m_admin_0_countries.dbf`
* OSM source: `philippines-260509.osm.pbf`
* Selection rule: union all extracted Philippines admin_level=4 province polygons plus admin_level=6 city/municipality polygons from the Geofabrik Philippines PBF; Natural Earth PH is retained as provenance only
* Boundary bbox: `[112.6723041, 4.4528442, 126.7396349, 21.2572899]`
* Source scope polygon counts: level `4` = `82`, level `6` = `899`, total = `981`

`v1.0.2` intentionally adds admin-level-6 polygons to the country scope so highly urbanized / independent city boundaries, including Manila, are inside the runtime scope even when they are not covered by a province polygon.

---

# 4. Runtime Policy

| Policy Field | Value |
| --- | --- |
| Allowed Levels | `[4, 6, 10]` |
| Hierarchy Parent Level | `4` |
| Hierarchy Child Levels | `[6, 10]` |
| Repair Layer | Disabled |
| Nearby Policy | Enabled |
| Nearby Max Distance | `2.0 km` |
| Offshore Max Distance | `20.0 km` |
| Name Schema | `multilingual_v1` |
| Alias Languages | `en`, `es`, `ja`, `ko`, `tl` |

Feature counts in the staged runtime artifacts:

| Level | Built Polygon Count |
| --- | ---: |
| `4` | `82` |
| `6` | `898` |
| `10` | `4,472` |

---

# 5. Regression Validation

The Manila coordinate now resolves inside the staged and published dataset:

| Coordinate | Expected Area | Runtime Result |
| --- | --- | --- |
| `14.608753147958526, 120.97594995915485` | Manila, Metro Manila, Philippines | `Manila` |

Runtime lookup for `14.608753147958526, 120.97594995915485` returns:

| Rank | Level | Feature ID | Name | Source |
| --- | ---: | --- | --- | --- |
| `0` | `6` | `ph_r103703` | `Manila` | `polygon` |

The result status is `partial`, which is expected for a point with level-6 coverage but no enclosing admin-level-4 province. The important regression fix is that the point is no longer outside the dataset scope and no longer returns an offshore source.

---

# 6. Test Methodology

| Field | Value |
| --- | --- |
| Total samples | `10,000` |
| Inside ratio | `0.9` |
| Inside samples | `9,000` |
| Outside samples | `1,000` |
| Lookup mode | `Runtime` |
| Evaluation boundary | `tmp/ph_country.json` |

The test intentionally injects approximately 10% out-of-scope points to validate boundary rejection, offshore classification, and cross-border isolation.

---

# 7. Performance Metrics

| Metric | Value |
| --- | ---: |
| Throughput | `9897.880 QPS` |
| Total Runtime | `1.010 sec` |
| Overall Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Failed Samples | `0` |

---

# 8. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed | Inside Failed | Status (ok/partial/failed/unknown) |
| --- | ---: | ---: | ---: | ---: | --- |
| `full_policy` | `100.00%` | `100.00%` | `0` | `0` | `209/8,795/996/0` |
| `no_hierarchy` | `100.00%` | `100.00%` | `0` | `0` | `209/8,795/996/0` |
| `no_repair` | `100.00%` | `100.00%` | `0` | `0` | `209/8,795/996/0` |
| `no_nearby` | `99.98%` | `99.98%` | `2` | `2` | `205/8,793/1,002/0` |
| `osm_only` | `99.98%` | `99.98%` | `2` | `2` | `205/8,793/1,002/0` |

---

# 9. Layer Contribution Analysis

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | `0` |
| Repair | `0` |
| Nearby | `6` |
| Total vs OSM-only | `6` |

---

# 10. Structural Distribution

## 10.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[4,6]` | `5,010` |
| `[4]` | `3,349` |
| `[]` | `1,000` |
| `[6]` | `264` |
| `[4,6,10]` | `205` |
| `[6,10]` | `166` |
| `[4,10]` | `6` |

## 10.2 Node Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | `8,998` |
| `admin_tree_id` | `9` |
| `nearby` | `2` |

## 10.3 Source Mix Distribution

| Source Mix | Count |
| --- | ---: |
| `polygon` | `8,989` |
| `__none__` | `1,000` |
| `admin_tree_id|polygon` | `9` |
| `nearby` | `2` |

## 10.4 Policy Reason Distribution

| Reason | Count |
| --- | ---: |
| `shape_status_map` | `9,000` |
| `empty_shape` | `996` |
| `offshore` | `4` |

---

# 11. Level-4 Coverage

* Unique level-4 units hit: `82`
* Total level-4 hits, all samples: `8,570`
* Total level-4 hits, inside samples: `8,570`

The sampled run hit all 82 Philippines admin-level-4 units in the rebuilt runtime scope.

---

# 12. Boundary Isolation Validation

Under stress testing with 10% forced out-of-scope samples:

* No inside-boundary coverage failure was observed.
* No policy failure was observed.
* Empty-shape and offshore outcomes were limited to expected out-of-scope samples.
* No evidence was observed that hierarchy or nearby layers created cross-border escalation in this run.

---

# 13. Release Conclusion

`ph.admin v1.0.2` is valid for release. The Manila regression is fixed by expanding the PH dataset scope from province-only coverage to admin-level-4/6 coverage, while preserving the runtime policy contract and passing the mass evaluation.
