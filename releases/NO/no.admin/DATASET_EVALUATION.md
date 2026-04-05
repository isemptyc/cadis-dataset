# NO_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `no.admin`  
Version: `v1.0.1`  
Country: `NO`  
Policy Version: `1.0`

---

# 1. Purpose

This document records the structural and runtime evaluation for the staged
Norway dataset `no.admin v1.0.1`.

This report is based on the staged runtime dataset produced from the committed
Norway engine revision:

- cadis-dataset-engine commit:
  `5ff06d4d5aafb8b5bf958cbcc45ffb42013db285`
- cadis version:
  `v0.4.6`
- OSM snapshot: `norway-latest.osm.pbf`
- OSM replication timestamp: `2025-10-13T20:21:02Z`

Observed findings:

- The Norway engine builds a stable `4 -> 7` administrative spine.
- The initial stray foreign municipality (`Eda kommun`) was removed from the
  engine before the validated evaluation pass.
- Runtime behavior is fully passing on the 10,000-point mass test.

---

# 2. Dataset Identity

| Field | Value |
| --- | --- |
| Dataset ID | `no.admin` |
| Dataset Version | `v1.0.1` |
| Country | `NO` |
| Country Name | `Norway` |
| Policy Version | `1.0` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Runtime Compat Min | `0.4.3` |
| Runtime Compat Max Exclusive | `1.0.0` |

---

# 3. Structural Build Summary

## 3.1 Engine Output

Observed structure:

- Level `4` units: `17`
- Level `7` units: `357`
- Missing municipality parents: `0`
- Foreign municipality leakage after pruning: `0`

## 3.2 Runtime Policy

Configured levels and shapes:

- `allowed_levels = [4, 7]`
- `allowed_shapes = [4], [4,7], [7]`

Status semantics:

- `[4,7]` => `ok`
- `[4]` => `partial`
- `[7]` => `partial`

---

# 4. Test Methodology

## 4.1 Sampling Strategy

Mass runtime validation was executed with:

- Total samples: `10,000`
- Inside ratio: `0.9`
- Expected inside points: `9,000`
- Expected outside points: `1,000`
- Lookup mode: `runtime`
- Workers: `32`
- Sampling source: Natural Earth country boundary DBF

Runtime command notes:

- The default sampler attempt cap was insufficient for Norway.
- Evaluation succeeded with `--max-attempts 2000000`.
- The SOP runner was patched to expose `--evaluate-max-attempts` and `--skip-pull`
  so Norway can be evaluated with the proven sampler budget and a locally
  available Docker image.

---

# 5. Performance Metrics

| Metric | Value |
| --- | --- |
| Throughput | `1302.840` QPS |
| Total Runtime | `7.676 sec` |
| Overall Pass Rate | `100.00%` |
| Policy Pass Rate | `100.00%` |
| Coverage Pass Rate | `100.00%` |
| Inside Coverage Pass Rate | `100.00%` |
| Failed Samples | `0` |

---

# 6. Scenario Comparison

| Scenario | Pass Rate | Inside Pass Rate | Failed |
| --- | ---: | ---: | ---: |
| `full_policy` | `100.00%` | `100.00%` | `0` |
| `no_hierarchy` | `100.00%` | `100.00%` | `0` |
| `no_repair` | `100.00%` | `100.00%` | `0` |
| `no_nearby` | `99.25%` | `99.17%` | `75` |
| `osm_only` | `99.25%` | `99.17%` | `75` |

Interpretation:

- Hierarchy is present but contributes almost nothing in this sampled run.
- Repair is not needed.
- Nearby fallback is the only materially rescuing layer.

---

# 7. Layer Contribution Analysis

| Layer | Rescued Samples |
| --- | ---: |
| Hierarchy | `0` |
| Repair | `0` |
| Nearby | `76` |
| Total vs OSM-only | `76` |

Observed direct layer usage:

- Polygon source samples: `8927`
- Nearby source samples: `68`
- Hierarchy source samples: `2`
- Offshore samples: `8`

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape | Count |
| --- | ---: |
| `[4,7]` | `6248` |
| `[4]` | `2747` |
| `[]` | `1005` |

## 8.2 Node Source Distribution

| Source | Count |
| --- | ---: |
| `polygon` | `8927` |
| `nearby` | `68` |
| `admin_tree_name` | `2` |

## 8.3 Policy Reason Distribution

| Reason | Count |
| --- | ---: |
| `shape_status_map` | `8995` |
| `empty_shape` | `997` |
| `offshore` | `8` |

---

# 9. Level-4 Coverage

- Unique level-4 units hit: `17`
- Total level-4 hits (all samples): `8995`
- Total level-4 hits (inside samples): `8993`

Top hit distribution:

| Level-4 Unit | Hits | Hit Rate (All Samples) |
| --- | ---: | ---: |
| `Svalbard` | `2736` | `27.36%` |
| `Finnmark` | `1217` | `12.17%` |
| `Innlandet` | `918` | `9.18%` |
| `Trøndelag` | `818` | `8.18%` |
| `Nordland` | `793` | `7.93%` |

Note:

- Area-weighted sampling strongly favors large northern territories such as
  `Svalbard`. This is expected under uniform land-area sampling and does not
  indicate a lookup defect.

---

# 10. Boundary Isolation Validation

Under mixed inside/outside stress testing:

- No failed samples were observed.
- No cross-border escalation was observed in the final validated engine build.
- Offshore handling remained bounded (`8` offshore samples).

This confirms strict boundary containment in the evaluated runtime dataset.

---

# 11. Conclusion

Current status:

- Norway `no.admin v1.0.1` passes the 10,000-point runtime mass test at
  `100.00%`.
- The committed Norway engine build is structurally clean for the `4 -> 7`
  model.
- The dataset is ready for release workflow continuation from a quality
  perspective.

