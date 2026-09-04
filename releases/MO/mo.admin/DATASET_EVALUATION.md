# MO_DATASET_EVALUATION

## Cadis Dataset Evaluation Report

Dataset: `mo.admin`

Version: `v1.0.1`

Country: `MO`

Policy Version: `1.0`

Evaluation Date: `2026-09-04`

---

# 1. Purpose

This report evaluates the administrative coverage, structural integrity, runtime
behavior, and boundary handling of `mo.admin v1.0.1` under Cadis Runtime.

v1.0.1 is a coverage correction. The published v1.0.0 dataset contained two
features and covered only Coloane; the Macau Peninsula, Cotai and Taipa returned
no administrative result, and Cotai coordinates resolved to Coloane, a different
island. This release restores the peninsula and Cotai. Taipa remains uncovered
and is recorded as a known gap rather than left implicit.

---

# 2. Dataset Identity

| Field | Value |
| ----- | ----- |
| Dataset ID | `mo.admin` |
| Dataset Version | `v1.0.1` |
| Country | `MO` |
| Country Name | `Macau` |
| Policy Version | `1.0` |
| Cadis Version | `v0.8.155` |
| Hierarchy Required | `True` |
| Repair Required | `False` |
| Runtime Policy Detected | `True` |
| Name Schema | `multilingual_v1` |
| Administrative Features | 8 (v1.0.0: 2) |
| Payload | 8,815 bytes |

---

# 3. Dataset Scope

| Field | Value |
| ----- | ----- |
| Scope Label | `administrative coverage` |
| Boundary Builder | `scripts/build_mo_boundaries.py` |
| Generated Boundary | `tmp/mo_v101/mo_country.json` |
| Boundary Source | `Natural Earth admin-0 + OSM administrative relations` |
| Boundary Selection Rule | Levels 5 and 6, admitted when tagged `ISO3166-1:alpha2=MO` or overlapping the Natural Earth MO polygon by at least 20% of their own area, then a 0.002 degree inward buffer |
| Boundary BBox | `[113.53018662917238, 22.111964207239268, 113.59004511949867, 22.214890875596666]` |
| OSM Source | `geofabrik:asia/macau` |
| OSM Snapshot Timestamp | `2026-05-11T20:20:52Z` |

The same scoped boundary was used for both build and evaluation.

## 3.1 What v1.0.0 got wrong

Three independent defects compounded, each sufficient to hide the others. The
source snapshot is unchanged between v1.0.0 and v1.0.1, so every difference below
comes from build configuration rather than from new data.

1. **Scope was built from level 5 alone.** Macau has only two level-5 units, Taipa
   and Coloane. Taipa does not assemble from this extract, so the scope collapsed
   to Coloane. Its bbox was
   `[113.55187423235276, 22.111964207239268, 113.59004511949867, 22.136743717018202]`,
   which is Coloane and nothing else. The engine targets levels 5 and 6, so every
   peninsula parish, which exists only at level 6, was then removed by the country
   filter.
2. **Membership used a representative-point test against Natural Earth.** After
   level 6 was added, Sé and the Cotai Landfill Zone were still rejected: the
   Natural Earth MO polygon predates much of Macau's reclaimed land, so their
   representative points fall outside it. Sé contains Senado Square, the Ruins of
   St Paul and Macau Tower. Membership is now proportional, and Macau's units
   overlap that polygon by 26% to 100% while neighbouring mainland units do not
   assemble in this extract at all, so the 20% threshold separates them with wide
   margin.
3. **Simplify tolerances were scaled for large countries.** Levels 5 and 6 used
   0.002 and 0.001 degrees, roughly 220 m and 110 m, applied to parishes under
   1 km across. That displaced parish boundaries far enough to push Senado Square
   outside every polygon even once Sé was in scope. Tolerances are now 0.0002 and
   0.0001, at which all sampled landmarks fall inside their parish and the total
   area change is under 0.01 km².

**None of this was visible to the v1.0.0 evaluation, which reported a 100% inside
coverage pass rate.** Build and evaluation shared the collapsed boundary, so the
missing territory was outside the sampled population by construction. A mass pass
rate measured against a generated scope cannot detect scope that is itself wrong.
The fixed-location probes in section 11 exist for that reason.

**Known gap: Taipa.** Its level-5 relation and its Carmo parish both depend on
member ways absent from the published extract, and the parish declares no
subareas, so it cannot be derived from lower levels the way other units can.
Macau International Airport sits on Taipa and is likewise uncovered.

---

# 4. Administrative Model

| Level | Runtime Label | Dataset Count |
| ----- | ------------- | ------------- |
| 5 | `admin_district` | 1 |
| 6 | `admin_municipality` | 7 |

The level-5 unit is Coloane. The seven level-6 units are the five peninsula
parishes (Sé, Santo António, São Lázaro, São Lourenço, Nossa Senhora de Fátima),
São Francisco Xavier on Coloane, and the Cotai Landfill Zone.

Macau's peninsula has no level-5 unit in OSM, so peninsula lookups return a
level-6 result with no level-5 parent. The hierarchy holds 8 nodes, 1 parent link
and 7 roots. Under runtime policy a level-6-only shape maps to `partial`, which is
why partial dominates the sampled outcomes below; it reflects Macau's
administrative structure, not a geometry defect.

Multilingual aliases are sparse in the source: 3 features carry `en` and 5 carry
`ru`. Alias counts describe source availability, not translation completeness.

---

# 5. Test Methodology

- Runtime-mode evaluation through the standard Cadis mass tester.
- **10,000 samples**: 9,000 inside and 1,000 outside the generated boundary.
- Full-policy results compared with `no_hierarchy`, `no_repair`, `no_nearby` and
  `osm_only`.
- Fixed-location probes at named Macau landmarks, chosen independently of the
  build, and compared directly against the published v1.0.0 dataset.

The mass pass rate describes the declared scope only. Because the scope grew
between v1.0.0 and v1.0.1, the two sample populations differ and their aggregate
counts are not a longitudinal comparison.

---

# 6. Evaluation Results

| Metric | Value |
| ------ | ----- |
| Overall Pass Rate | 100.00% (10,000/10,000) |
| Inside Coverage Pass Rate | 100.00% (9,000/9,000) |
| Policy Pass Rate | 100.00% |
| Validation Failures | 0 |
| Throughput | 24,092 QPS |
| Total Runtime | 0.415 seconds |
| Runtime Lookup Status: ok | 2,762 |
| Runtime Lookup Status: partial | 7,207 |
| Runtime Lookup Status: failed | 31 |
| Offshore Outcomes | 921 |

Macau is roughly 33 km² and largely surrounded by water, so offshore outcomes are
a substantial share of sampling near the boundary. Expected outside points may
return `failed`; that is distinct from a validation failure.

---

# 7. Scenario Comparison

| Scenario | Pass Rate | Failed | Inside Failed | Statuses: ok / partial / failed |
| -------- | --------- | ------ | ------------- | ------------------------------- |
| `full_policy` | 100.00% | 0 | 0 | 2,762 / 7,207 / 31 |
| `no_hierarchy` | 100.00% | 0 | 0 | 2,762 / 7,207 / 31 |
| `no_repair` | 100.00% | 0 | 0 | 2,762 / 7,207 / 31 |
| `no_nearby` | 99.97% | 3 | 3 | 1,798 / 7,207 / 995 |
| `osm_only` | 99.97% | 3 | 3 | 1,798 / 7,207 / 995 |

Rates are recomputed from exact counts rather than copied from rounded summary
percentages.

---

# 8. Layer Contribution Analysis

| Scenario-Delta Metric | Samples |
| --------------------- | ------- |
| Rescued by hierarchy | 0 |
| Rescued by repair | 0 |
| Rescued by nearby | 964 |
| Total versus OSM-only | 964 |

The nearby delta is large in proportion because Macau's coastline dominates its
boundary; it measures scenario-outcome changes including offshore handling, not
inside validation failures prevented. Only three inside failures occur without
nearby behavior.

---

# 9. Structural Distribution

## 9.1 Shape Distribution

| Category | Samples |
| -------- | ------- |
| `[6]` | 7,207 |
| `[5,6]` | 1,841 |
| `[]` | 952 |

## 9.2 Node Source Distribution

| Category | Samples |
| -------- | ------- |
| `polygon` | 9,005 |
| `nearby` | 43 |
| `admin_tree_id` | 3 |

## 9.3 Policy Reason Distribution

| Category | Samples |
| -------- | ------- |
| `shape_status_map` | 9,048 |
| `offshore` | 921 |
| `empty_shape` | 31 |

---

# 10. Level-4 Coverage

Not applicable. Macau has no level-4 administrative unit; the runtime contract is
anchored at levels 5 and 6.

---

# 11. Fixed-Location Probes

Named landmarks, resolved against the published v1.0.0 dataset and this release.
These probes are independent of the generated scope and are the primary evidence
for this correction.

| Location | v1.0.0 | v1.0.1 |
| -------- | ------ | ------ |
| Senado Square | *(empty)* | `大堂區 Sé` |
| Ruins of St Paul | *(empty)* | `Santo António` |
| Macau Tower | *(empty)* | `大堂區 Sé` |
| A-Ma Temple | *(empty)* | `風順堂區 São Lourenço` |
| Guia Fortress | *(empty)* | `望德堂區 São Lázaro` |
| Macau Peninsula (north) | *(empty)* | `Santo António` |
| Cotai / Venetian | `Coloane > 聖方濟各堂區` (wrong island) | `Cotai Landfill Zone` |
| Coloane village | `Coloane > 聖方濟各堂區` | unchanged |
| Taipa village | *(empty)* | *(empty)* — known gap |
| Macau International Airport | `Coloane > 聖方濟各堂區` (wrong island) | `Coloane > Cotai Landfill Zone` |

Cotai and the airport were not merely absent in v1.0.0: they resolved to Coloane,
a different island. Correcting a wrong answer matters as much as filling an empty
one.

---

# 12. Boundary Isolation Validation

The 1,000 expected-outside samples produced no validation failure. Neighbouring
mainland units do not assemble in this extract, and a mainland unit would overlap
the Natural Earth MO polygon by only a few percent against the 20% admission
threshold, so the proportional rule does not admit cross-border geometry. A
Zhuhai control point tests as outside the generated scope.

---

# 13. Structural Observations and Known Issues

1. Taipa is uncovered, as described in section 3.1.
2. The 0.002 degree inward buffer removes about 27% of Macau's land area, from
   27.6 km² to 20.0 km². Every landmark above still falls inside, but the buffer
   is proportionally far more aggressive here than on a large country. Recorded as
   a known issue; not changed in this release.
3. Simplify tolerances tuned for large countries are unlikely to be a Macau-only
   concern. Other small or fragmented territories may lose boundary-adjacent
   locations the same way. Recorded as a known issue for a separate sweep.
4. Seven of eight features are hierarchy roots, because Macau's peninsula has no
   level-5 unit in OSM.

---

# 14. Reproducibility

- Engine commit: `08d70b3af39681c77615616741c69e7da4990e3a`.
- Build environment: pinned Docker image
  `ghcr.io/isemptyc/cadis-dataset-engine@sha256:e5b134214a51368391576e5bbe6b06b403e3acf19d9d91124063caba72f1ae76`.
- Boundary generation and evaluation environment: conda `photo-importer-geo`.
- Source: `macau-260511.osm.pbf`, replication timestamp `2026-05-11T20:20:52Z`,
  SHA-256 `222cdb54cae22bdf5581ef102de80f24809b1bbf15fe7c3dc03a95dbe1312109`.
  Identical to the snapshot used for v1.0.0.
- Build/evaluation boundary SHA-256:
  `f39a5046f7603e67e60b7ed15777abf091adda107538d09df453a2b0bfca938c`.
- Release recipe: `scripts/mo_dataset_release.txt`.
- Release destination: `releases/MO/mo.admin/v1.0.1`.

Because the source is unchanged, the entire coverage difference between v1.0.0
and v1.0.1 is attributable to the three build-configuration corrections in
section 3.1.

---

# 15. Conclusion

`mo.admin v1.0.1` corrects a coverage defect that left the Macau Peninsula and
Cotai without administrative results and returned the wrong island for Cotai and
the airport. Coverage rises from 2 features to 8, and every sampled landmark
except Taipa now resolves to its correct parish. Taipa, the inward buffer and the
simplification tolerances are recorded as known issues rather than left implicit.
The release is ready for operator review before Git and R2 distribution.
