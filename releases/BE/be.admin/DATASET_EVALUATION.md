# BE_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `be.admin`
Version: `v1.0.1`
Country: `BE`
Policy Version: `1.0`
Name Schema: `multilingual_v1`

---

# 1. Purpose

This document provides a structural, behavioral, and boundary-integrity evaluation of the `be.admin v1.0.1` dataset under Cadis Runtime.

This report:

* describes the finalized Belgium dataset shape
* validates the multilingual naming contract introduced for Belgium
* quantifies runtime policy behavior under mixed inside/outside sampling
* documents hierarchy and nearby fallback effects
* validates country-scope isolation under stress testing

Observed onboarding issues were not primarily OSM data-quality failures.
The first major issue was runtime overuse of all scope-flagged features as country-scope execution geometry.
The second issue was dataset policy: canonical naming had been biased toward French instead of preserving the local/default OSM name and carrying multilingual aliases separately.

Cadis does not modify geography.
Cadis also does not select display language.
It enforces structural determinism, boundary integrity, and dataset-defined canonical naming.

---

# 2. Dataset Identity

| Field                   | Value                |
| ----------------------- | -------------------- |
| Dataset ID              | `be.admin`           |
| Dataset Version         | `v1.0.1`             |
| Country                 | `BE`                 |
| Policy Version          | `1.0`                |
| Name Schema             | `multilingual_v1`    |
| Hierarchy Required      | `True`               |
| Repair Required         | `False`              |
| Runtime Policy Detected | `True`               |

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

Canonical naming policy for Belgium:

* canonical `name` uses local/default OSM preference
* multilingual aliases are exported separately in `names`
* Belgium alias export is intentionally restricted to:
  * `nl`
  * `fr`
  * `de`

Example:

```json
{
  "name": "Brugge",
  "names": {
    "nl": "Brugge",
    "fr": "Bruges",
    "de": "Brügge"
  }
}
```

---

# 3. Test Methodology

## 3.1 Sampling Strategy

* Total samples: `10,000`
* Sampling mode: mixed inside/outside stress testing
* Inside ratio: `0.9`
* Expected outside ratio: `0.1`

The test intentionally injects ~10% out-of-country points to validate:

* boundary rejection behavior
* offshore classification
* policy-layer containment
* cross-border isolation

Sampling is uniform over land area and uses the Belgium boundary JSON generated from Natural Earth.

## 3.2 Runtime Context

Cadis runtime country-scope execution geometry uses the minimum flagged admin level rather than every `country_scope_flag=true` feature. For Belgium this reduces country-scope execution geometry from:

* `575` features / `627` parts

to:

* `3` features / `27` parts

This preserves exporter scope truth while avoiding pathological country-scope checks over all municipalities.

The runtime also now passes multilingual aliases through as metadata:

* `name` remains canonical and deterministic
* `names` is returned unchanged
* no locale-aware behavior is introduced

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
| Throughput                | `1269.680` QPS |
| Total Runtime             | `7.876 sec`    |
| Overall Pass Rate         | `100.00%`      |
| Inside Coverage Pass Rate | `100.00%`      |
| Policy Pass Rate          | `100.00%`      |

This run confirms that the multilingual Belgium dataset remains policy-correct and operationally fast after the runtime scope-geometry fix and naming-contract evolution.

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

* OSM polygon coverage remains structurally strong for Belgium.
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

| Level-4 Unit                                      | Hits  | Hit Rate (All Samples) | Hits (Inside Samples) | Hit Rate (Inside Samples) |
| ------------------------------------------------- | ----- | ---------------------- | --------------------- | ------------------------- |
| Wallonie                                          | 4,920 | 49.20%                 | 4,919                 | 54.66%                    |
| Vlaanderen                                        | 4,032 | 40.32%                 | 4,031                 | 44.79%                    |
| Région de Bruxelles-Capitale - Brussels Hoofdstedelijk Gewest | 50 | 0.50% | 50 | 0.56% |

The low Brussels hit rate is expected under uniform land-area sampling because the Brussels region is geographically much smaller than the other two level-4 regions.

The level-4 canonical names now reflect the local/default OSM naming policy rather than a French-first override.

---

# 11. Boundary Isolation Validation

Under stress testing with 10% forced out-of-bound samples:

* no validation failure was observed
* empty-shape and offshore outcomes dominated expected outside-labeled samples
* no evidence was observed that hierarchy or nearby layers caused cross-border escalation
* export-time scope truth remained intact while runtime execution used a minimized country-scope shell

This confirms strict boundary containment for the Belgium dataset.

---

# 12. Structural Observations

1. Belgium geometry is materially denser than Norway and more expensive per part than Italy, but the finalized runtime remains fast after the scope-geometry fix.
2. The main onboarding problem was not a fatal OSM integrity issue; it was runtime overuse of all scope-flagged features as country-scope execution geometry.
3. Belgium-specific engine cleanup was still required:
   * foreign-border municipality leakage was pruned
   * Brussels municipalities were attached directly to `Bruxelles-Capitale`
4. `v1.0.1` introduces the multilingual naming contract:
   * canonical `name` is deterministic and local/default-first
   * `names` carries auxiliary aliases for query recall
   * runtime remains language-agnostic
5. Belgium alias export is intentionally narrow and dataset-specific:
   * `nl`
   * `fr`
   * `de`
6. Final staged dataset structure is clean:
   * `3` roots
   * `0` missing level-8 parents in the built admin dataset
7. Nearby fallback remains the only layer providing net rescue over OSM-only behavior.

---

# 13. Reproducibility

All dataset transformations and evaluation results are reproducible using:

* cadis-dataset-engine commit:
  `7e437fd5a8ecf6d3421f680294c5f90855e5cbf7`
* cadis-dataset-engine semantic-layer multilingual preservation commit:
  `c6d0e93895d129623008e6d4280e12c8eea67413`
* Cadis runtime multilingual passthrough commit:
  `95592727750f77d740580e46dd21fcff602d29cc`
* Cadis runtime version baseline during evaluation:
  `0.4.6`

---

# 14. Conclusion

The `be.admin v1.0.1` dataset demonstrates:

* full inside-boundary coverage under policy
* strict cross-border isolation
* clean finalized Belgium administrative structure
* no repair-layer dependence
* useful nearby fallback contribution
* stable runtime performance after the scope-geometry correction
* successful adoption of the `multilingual_v1` dataset contract

Belgium `v1.0.1` resolves the earlier naming-policy issue by separating:

* canonical deterministic identity (`name`)
* multilingual query aliases (`names`)

With those corrections in place, the dataset is evaluation-green and ready for the next release steps.
