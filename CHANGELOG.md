# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Add link `rel="gcps"` to Ground Control Points document
- All raster fields can now be used in the new bands field in STAC 1.1

### Changed

- Some clarifications and fixes in the README and examples [#41](https://github.com/stac-extensions/raster/pull/41)
- `raster:bands` is now using the more general `bands` field from STAC 1.1 common metadata
- All of the fields in the `raster:bands` object have been renamed to have a prefix of `raster:`

### Removed

- `raster:bands` - use `bands` in STAC core instead

## [v1.1.0]

### Added

- Allow "inf" and "nan" as valid nodata values [#18](https://github.com/stac-extensions/raster/issues/18)

## [v1.0.0]

- Initial release

[Unreleased]: <https://github.com/stac-extensions/raster/compare/v1.1.0...HEAD>
[v1.1.0]: <https://github.com/stac-extensions/tree/v1.1.0>
[v1.0.0]: <https://github.com/stac-extensions/tree/v1.0.0>
