# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [v2.0.0]

## [v2.0.0-beta.1]

### Added

- Add link relation type `gcps` for Ground Control Points documents
- `raster:sampling`, `raster:bits_per_sample`, `raster:spatial_resolution`, `raster:scale`, `raster:offset` and `raster:histogram`
  can be used in Assets and Item Properties

### Changed

- Some clarifications and fixes in the README and examples [#41](https://github.com/stac-extensions/raster/pull/41)
- `raster:bands` is now using the more general `bands` construct from STAC common metadata
- The following fields in the Band Object have been moved/renamed:
  - `nodata`, `data_type`, `statistics` and `unit` were *not* renamed, but have been moved to STAC common metadata
  - `sampling` has been renamed to `raster:sampling`
  - `bits_per_sample` has been renamed to `raster:bits_per_sample`
  - `spatial_resolution` has been renamed to `raster:spatial_resolution`
  - `scale` has been renamed to `raster:scale`
  - `offset` has been renamed to `raster:offset`
  - `histogram` has been renamed to `raster:histogram`

### Removed

- `raster:bands` - use `bands` in instead

### Fixed

- Improved and stricter JSON Schema

## [v1.1.0]

### Added

- Allow "inf" and "nan" as valid nodata values [#18](https://github.com/stac-extensions/raster/issues/18)

## [v1.0.0]

- Initial release

[Unreleased]: <https://github.com/stac-extensions/raster/compare/v2.0.0...HEAD>
[v2.0.0]: <https://github.com/stac-extensions/raster/compare/v2.0.0-beta.1...v2.0.0>
[v2.0.0-beta.1]: <https://github.com/stac-extensions/tree/v2.0.0-beta.1>
[v1.1.0]: <https://github.com/stac-extensions/tree/v1.1.0>
[v1.0.0]: <https://github.com/stac-extensions/tree/v1.0.0>
