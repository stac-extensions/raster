# Raster Extension Specification

- **Title:** Raster
- **Identifier:** https://stac-extensions.github.io/raster/v1.0.0/schema.json
- **Field Name Prefix:** raster
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @emmanuelmathot

This document explains the Raster Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.

An item can describe assets as an assembly of related raster bands and some information common to them all. An item can have the concept of the raster size (in pixels and lines) that applies to all the bands. The raster is strongly linked with the the georeferencing transform and coordinate system definition of all bands (using the [projection extension](https://github.com/radiantearth/stac-spec/tree/master/extensions/projection)). Some raster information are interesting for several applications such as rendering properly the bands as image with statistics about pixels values (histogram range) or nodata value.

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Item Properties or Item Asset fields

| Field Name    | Type                                         | Description                                                                                                              |
| ------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| raster:bands  | \[[Raster band Object](#raster-band-object)] | An array of available bands where each object is a \[[Band Object](#band-object)]. If given, requires at least one band. |
| raster:height | number                                       | Number of lines of pixels in the raster                                                                                  |


## Raster Band Object

| Field Name      | Type   | Description                                                                                                                                                                      |
| --------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| width           | number | Number of pixels in a line of the raster.                                                                                                                                        |
| height          | number | Number of lines of pixels in the raster.                                                                                                                                         |
| datatype        | string | One of Byte, UInt16, Int16, UInt32, Int32, Float32, Float64, and the complex types CInt16, CInt32, CFloat32, and CFloat64                                                        |
| nodata          | number | Pixel values used to identify pixels that are nodata in the assets .                                                                                                             |
| sampling        | string | One of `area` or `point`. Indicates whether a pixel value should be assumed to represent a sampling over the region of the pixel or a point sample at the center of the pixel.   |
| nbits           | number | The actual number of bits used for this band. Normally only present when the number of bits is non-standard for the `datatype`, such as when a 1 bit TIFF is represented as byte |
| statistics_mean | number | mean value of all the pixels in the band                                                                                                                                         |
| statistics_min | number | minimum value of the pixels in the band |
| statistics_miax | number | maximum value of the pixels in the band |
| statistics_stdev | number |  standard deviation value of the pixels in the band |

### Additional Field Information

#### template:new_field

TBD

## Relation types

TBD

<!-- The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type           | Description                           |
| -------------- | ------------------------------------- |
| fancy-rel-type | This link points to a fancy resource. | -->
