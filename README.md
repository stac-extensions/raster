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

| Field Name   | Type                                         | Description                                                                                                              |
|--------------|----------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|
| raster:bands | \[[Raster band Object](#raster-band-object)] | An array of available bands where each object is a \[[Band Object](#band-object)]. If given, requires at least one band. |

## Raster Band Object

When specifying a raster band object at asset level. It is recommended to use 

- the [file](https://github.com/stac-extensions/file) extension to specify the `file:data_type` and `file:unit` to indicate both the encoding type nd unit of each pixel and.
- the [proj](https://github.com/radiantearth/stac-spec/tree/master/extensions/projection) extension to specify information about the raster projection, especially `proj:shape` to specify the height and width of the raster.

| Field Name           | Type   | Description                                                                                                                                                                      |
|----------------------|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| nodata               | number | Pixel values used to identify pixels that are nodata in the assets .                                                                                                             |
| sampling             | string | One of `area` or `point`. Indicates whether a pixel value should be assumed to represent a sampling over the region of the pixel or a point sample at the center of the pixel.   |
| nbits                | number | The actual number of bits used for this band. Normally only present when the number of bits is non-standard for the `datatype`, such as when a 1 bit TIFF is represented as byte |
| stats_mean           | number | mean value of all the pixels in the band                                                                                                                                         |
| stats_min            | number | minimum value of the pixels in the band                                                                                                                                          |
| stats_max            | number | maximum value of the pixels in the band                                                                                                                                          |
| stats_stdev          | number | standard deviation value of the pixels in the band                                                                                                                               |
| stats_valid_percent  | number | percentage of valid (not `nodata`) pixel                                                                                                                                         |
| scale                | number | multiplicator factor of the pixel value to transform into meaningful values (i.e. translate digital number to reflectance).                                                      |
| offset               | number | number to be added to the pixel value to transform into meaningful values (i.e. translate digital number to reflectance).                                                        |
| color_interpretation | string | the color interpretation of the pixels in the bands. One of the [color interpreation](#color-interpretation)) below.                                                             |

### Additional Field Information

#### Scale and offset as radiometric calibration parameters

In remote sensing, many imagery raster corresponds to raw data without any radiometric processing. Each pixel is given in digital numbers (DN), i.e. native pixel values from the sensor acquisition. Those digital numbers quantify the energy recorded by the detector (optical or radar). The sensor radiometric calibration aims to turn back the DN value into a physical unit value (radiance, light power, backscatter). Hereafter, some examples of the usage of the `scale` and `offset` fields to perform a radiometric correction.

##### Digital Numbers to Radiance (optical sensor)

Top Of Atmosphere (TOA) Radiance in Wμm-1 m-2 sr-1 is derived from DN values using the following formula:

![formula](https://render.githubusercontent.com/render/math?math=L_\lambda%20=%20scale%20\times%20DN%20%2B%20offset)

where:

\(L_\lambda\) is TOA Radiance in Wμm-1 m-2 sr-1.

#### Color interpretation

TBD

## Relation types

TBD

<!-- The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type           | Description                           |     |
|----------------|---------------------------------------|-----|
| fancy-rel-type | This link points to a fancy resource. | --> |
