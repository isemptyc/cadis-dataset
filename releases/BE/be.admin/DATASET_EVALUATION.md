# BE_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `be.admin`
Version: `v1.0.0`
Country: `BE`
Policy Version: `1.0`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `be.admin v1.0.0` dataset under Cadis Runtime.

This report:

* Describes the finalized Belgium dataset shape
* Quantifies runtime policy behavior under mixed inside/outside sampling
* Documents hierarchy and nearby fallback effects
* Validates country-scope isolation under stress testing
* Records the runtime-side scope-geometry correction required for efficient execution

Observed issues during onboarding were not primarily OSM data-quality failures.
The main runtime issue was that exporter-provided `country_scope_flag` was initially consumed too literally for execution geometry, causing country-scope checks to run against all in-scope features instead of a minimal boundary shell.

Cadis does not modify geography.
It enforces structural determinism and boundary integrity.

---

# 2. Dataset Identity

| Field                   | Value      |
| ----------------------- | ---------- |
| Dataset ID              | `be.admin` |
| Dataset Version         | `v1.0.0`   |
| Country                 | `BE`       |
| Policy Version          | `1.0`      |
| Hierarchy Required      | `True`     |
| Repair Required         | `False`    |
| Runtime Policy Detected | `True`     |

Final engine structure:

* Level `4`: Region
* Level `6`: Province-equivalent unit
* Level `8`: Municipality

Final geometry metadata counts:

* Total features: `575`
* Level distribution:
  * `4`: `3`
  * `6`: `10`
  * `8`: `562`

---

# 3. Test Methodology

## 3.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside ratio: `0.9`
* Expected outside ratio: `0.1`

The test intentionally injects ~10% out-of-country points to validate:

* Boundary rejection behavior
* Offshore classification
* Policy-layer containment
* Cross-border isolation

Sampling is uniform over land area and uses the Belgium boundary JSON generated from Natural Earth.

## 3.2 Runtime Context

During validation, Cadis runtime country-scope execution geometry was corrected to use the minimum flagged admin level rather than every `country_scope_flag=true` feature. This reduced Belgium country-scope execution geometry from:

* `575` features / `627` parts

to:

* `3` features / `27` parts

This fix preserved scope truth while removing the pathological runtime slowdown observed before the correction.

---

# 4. Observed Distribution

| Category                               | Count  |
| -------------------------------------- | ------ |
| Sample labels: expected inside points  | 9,000  |
| Sample labels: expected outside points | 1,000  |
| Structural non-empty outcomes          | 9,039  |
| Empty-shape outcomes (`[]`)            | 998    |
| Offshore outcomes                      | 37     |

Outside-labeled samples are intentional and do not indicate dataset coverage gaps.
`expected inside/outside` comes from Belgium boundary sampling labels; structural outcomes come from Cadis runtime evaluation.

---

# 5. Performance Metrics

| Metric                    | Value          |
| ------------------------- | -------------- |
| Throughput                | `1321.370` QPS |
| Total Runtime             | `7.568 sec`    |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |

This run confirms that the Belgium dataset is both policy-correct and operationally fast after the runtime scope-geometry fix.

---

# 6. Scenario Comparison

| Scenario     | Pass Rate | Inside Pass Rate | Failed |
| ------------ | --------- | ---------------- | ------ |
| full_policy  | 100.00%   | 100.00%          | 0      |
| no_hierarchy | 100.00%   | 100.00%          | 0      |
| no_repair    | 100.00%   | 100.00%          | 0      |
| no_nearby    | 98.84%    | 98.71%           | 116    |
| osm_only     | 98.84%    | 98.71%           | 116    |

---

# 7. Layer Contribution Analysis

| Layer             | Rescued Samples |
| ----------------- | --------------- |
| Hierarchy         | 0               |
| Repair            | 0               |
| Nearby            | 154             |
| Total vs OSM-only | 154             |

## Interpretation

* OSM polygon coverage is already structurally strong for Belgium.
* Hierarchy is still observed in runtime usage (`20` samples), mainly in `[4]` or `[4,8]` normalization paths, but it does not contribute net rescue over the sampled corpus.
* Nearby fallback remains materially useful for Belgium and rescues `154` samples compared with `osm_only`.
* No repair operations were triggered.

Observed non-`ok` results in the full-policy run are policy-consistent outcomes, not validation failures:

* `partial`: `21`
* `failed`: `961`

These correspond to permitted structural shapes, empty-shape outside results, or offshore classification, not incorrect country assignment.

---

# 8. Structural Distribution

## 8.1 Shape Distribution

| Shape   | Count |
| ------- | ----- |
| [4,6,8] | 8,911 |
| []      | 998   |
| [4,8]   | 58    |
| [4]     | 21    |
| [4,6]   | 12    |

Empty shapes correspond to:

* `empty_shape`: `961`
* `offshore`: `37`

## 8.2 Node Source Distribution

| Source          | Count |
| --------------- | ----- |
| polygon         | 8,885 |
| nearby          | 117   |
| admin_tree_name | 20    |

### Source Mix

| Mix                     | Count |
| ----------------------- | ----- |
| polygon                 | 8,865 |
| none                    | 998   |
| nearby                  | 117   |
| admin_tree_name|polygon | 20    |

---

# 9. Policy Reason Distribution

| Reason           | Count |
| ---------------- | ----- |
| shape_status_map | 9,002 |
| empty_shape      | 961   |
| offshore         | 37    |

---

# 10. Level-4 Coverage

* Unique level-4 units hit: `3`
* Total level-4 hits (all samples): `9,002`
* Total level-4 hits (inside samples): `9,000`

Hit distribution reflects uniform land-area sampling.

## 10.1 Level-4 Hit Rates

| Level-4 Unit          | Hits  | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| --------------------- | ----- | ---------------------- | --------------------- | ------------------------- |
| Wallonie              | 4,920 | 49.20%                 | 4,919                 | 54.66%                    |
| Flandre               | 4,032 | 40.32%                 | 4,031                 | 44.79%                    |
| Bruxelles-Capitale    | 50    | 0.50%                  | 50                    | 0.56%                     |

The low Brussels hit rate is expected under uniform land-area sampling because Bruxelles-Capitale is geographically much smaller than the other two level-4 regions.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* No validation failure was observed.
* Empty-shape and offshore outcomes dominated expected outside-labeled samples.
* No evidence was observed that hierarchy or nearby layers caused cross-border escalation.
* Export-time scope truth remained intact while runtime execution used a minimized country-scope shell.

This confirms strict boundary containment for the Belgium dataset.

---

# 12. Structural Observations

1. Belgium geometry is materially denser than Norway and more expensive per part than Italy, but the finalized runtime remains fast after the scope-geometry fix.
2. The main onboarding problem was not a fatal OSM integrity issue; it was runtime overuse of all scope-flagged features as country-scope execution geometry.
3. Belgium-specific engine cleanup was still required:
   * foreign-border municipality leakage was pruned
   * Brussels municipalities were attached directly to `Bruxelles-Capitale`
4. Final staged dataset structure is clean:
   * `3` roots
   * `0` missing level-8 parents in the built admin dataset
5. Nearby fallback is bounded, useful, and contributes the only net rescue over OSM-only behavior.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit:
  `4c1233410b11eaf22974d8c0843fcaea9cde034a`
* Cadis runtime commit for scope-geometry fix:
  `2bde182e69fd9340e8168baa31973276ff95030d`
* Cadis local runtime version during evaluation:
  `0.4.5`

The engine repository was committed before staged dataset generation.
The Cadis runtime evaluation included the scope-geometry fix and local Belgium enablement/version preparation required by the SOP.

---

# 14. Conclusion

The `be.admin v1.0.0` dataset demonstrates:

* Full inside-boundary coverage under policy
* Strict cross-border isolation
* Clean finalized Belgium administrative structure
* No repair-layer dependence
* Useful nearby fallback contribution
* Stable runtime performance after correcting country-scope execution geometry

Belgium onboarding required both:

* engine-side structural cleanup
* runtime-side country-scope geometry minimization

With those corrections in place, the dataset is evaluation-green and suitable to proceed to the publish phase.
