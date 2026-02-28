# cadis-dataset

Cadis dataset release repository (CDN surface).

This repository contains immutable, versioned country dataset releases
consumed by the Cadis runtime.

It serves as the public distribution surface for released datasets only.

## Scope

This repository contains release artifacts, including:

- `releases/dataset_manifest.json` (repository-level dataset index)
- `dataset_release_manifest.json`
- `runtime_policy.json`
- geometry and metadata files
- policy-declared structural layers (e.g. `hierarchy.json`, `repair.json`)
- optional overlay files declared by policy

It does **not** contain:

- dataset transformation engine code
- internal build/governance tooling
- OpenStreetMap extract files

## Release Contract

1. Releases are immutable once published.
2. Existing versions are never modified in place.
3. Any dataset change must be published as a new version.
4. Cadis runtime treats this repository as release truth for integrity and policy-layer validation.

## Layout

```text
cadis-dataset/
└── releases/
    ├── dataset_manifest.json
    └── <ISO2>/
        └── <dataset_id>/
            └── <version>/
                ├── dataset_release_manifest.json
                ├── runtime_policy.json
                ├── geometry.ffsf
                ├── geometry_meta.json
                ├── hierarchy.json         (if required)
                ├── repair.json            (if required)
                └── <optional overlays>
```

## Manifest Profile

Release manifests use:

- `profile = "cadis.dataset.release"`
- `schema_version = 2`

Dataset index manifest uses:

- `path = "releases/dataset_manifest.json"`
- `schema_version = 1`

## Licensing and Attribution

Datasets in this repository are derived from OpenStreetMap data.

© OpenStreetMap contributors  
Licensed under the Open Database License (ODbL) 1.0  
[https://www.openstreetmap.org/copyright](https://www.openstreetmap.org/copyright)

This repository does not distribute raw OpenStreetMap extracts.
