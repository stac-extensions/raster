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
  - [Item example](examples/item-planet.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Item Properties or Item Asset fields

| Field Name        | Type                                                   | Description                                                                                                                                                         |
| ----------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| raster:bands      | \[[Raster band Object](#raster-band-object)]           | **Asset level**. An array of available bands where each object is a \[[Band Object](#band-object)]. If given, requires at least one band.                           |
| raster:composites | \[[Raster Composite Object](#raster-composite-object)] | **Item level**. An array of possible band composition where each object is a \[[Composite Object](#raster-composite-object)]. If given, requires at least one band. |

## Raster Band Object

When specifying a raster band object at asset level. It is recommended to use the [projection](https://github.com/radiantearth/stac-spec/tree/master/extensions/projection) extension to specify information about the raster projection, especially `proj:shape` to specify the height and width of the raster.

| Field Name           | Type                                        | Description                                                                                                                                                                                                        |
| -------------------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| nodata               | number                                      | Pixel values used to identify pixels that are nodata in the assets .                                                                                                                                               |
| sampling             | string                                      | One of `area` or `point`. Indicates whether a pixel value should be assumed to represent a sampling over the region of the pixel or a point sample at the center of the pixel.                                     |
| data_type            | string                                      | The data type of the band. One of the [data types as decribed in file extension](https://github.com/stac-extensions/file#data-types).                                                                              |
| nbits                | number                                      | The actual number of bits used for this band. Normally only present when the number of bits is non-standard for the `datatype`, such as when a 1 bit TIFF is represented as byte                                   |
| stats_mean           | number                                      | mean value of all the pixels in the band                                                                                                                                                                           |
| stats_min            | number                                      | minimum value of the pixels in the band                                                                                                                                                                            |
| stats_max            | number                                      | maximum value of the pixels in the band                                                                                                                                                                            |
| stats_stdev          | number                                      | standard deviation value of the pixels in the band                                                                                                                                                                 |
| stats_valid_percent  | number                                      | percentage of valid (not `nodata`) pixel                                                                                                                                                                           |
| values               | Map<string, [Value](#value-object) Object>] | Dictionary of value objects that can be computed, each with a unique key.                                                                                                                                          |
| overview_max_gsd     | number                                      | The maximum Ground Sample Distance represented in an overview. This should be the GSD of the highest level overview, generally of a [Cloud Optimized GeoTIFF](http://cogeo.org/), but should work with any format. |
| color_interpretation | string                                      | the color interpretation of the pixels in the bands. One of the [color interpreation](#color-interpretation)) below.                                                                                               |

**overview_max_gsd**: This field helps renderers of understand what zoom levels they can efficiently show. It is generally used in conjunction with gsd (from [common metadata](https://github.com/radiantearth/stac-spec/blob/master/item-spec/common-metadata.md#instrument)). `overview_max_gsd` enables the calculation of the 'minimum' zoom level that a renderer would want to show, and then the maximum zoom level is calculated from the gsd - the resolution of the image. The former is based on the highest level of overview (also known as a pyramid) contained in the asset.

<img src="https://user-images.githubusercontent.com/407017/90821250-75ce5280-e2e7-11ea-9008-6c073e083be0.png" alt="image pyramid" width="300">

So in the above image it would be the ground sample distance of 'level 4', which will be a much higher gsd than the image,
as each pixel is greatly down-sampled. Dynamic tile servers (like [titiler](https://github.com/developmentseed/titiler)) will
generally convert the gsd to [zoom 
levels](https://wiki.openstreetmap.org/wiki/Zoom_levels), [Web Mercator](https://en.wikipedia.org/wiki/Web_Mercator_projection) 
or others, which is easily done (example python [to webmercator](https://github.com/cogeotiff/rio-cogeo/blob/b9b57301c2b7a4be560c887176c282e68ca63c27/rio_cogeo/utils.py#L62-L66) or arbitrary [TileMatrixSet](https://github.com/cogeotiff/rio-tiler-crs/blob/834bcf3d39cdc555b3ce930439ab186d00fd5fc5/rio_tiler_crs/cogeo.py#L98-L105))

## Value Object

| Field Name | Type   | Description                                                                                                                                                               |
| ---------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| unit       | string | unit denomination of the value                                                                                                                                            |
| from       | string | key of another value in the [`values`](#raster-band-object) dictionary to be used as input to compute the value. If empty the value of the pixel in the band is the input |
| scale      | number | multiplicator factor of the pixel value to transform into the value (i.e. translate digital number to reflectance).                                                       |
| offset     | number | number to be added to the pixel value to transform into the value (i.e. translate digital number to reflectance).                                                         |

### Use Scale and offset as radiometric calibration parameters

In remote sensing, many imagery raster corresponds to raw data without any radiometric processing. Each pixel is given in digital numbers (DN), i.e. native pixel values from the sensor acquisition. Those digital numbers quantify the energy recorded by the detector (optical or radar). The sensor radiometric calibration aims to turn back the DN value into a physical unit value (radiance, light power, backscatter). Hereafter, some examples of the usage of the `values` dictionary to perform radiometric correction.

#### Digital Numbers to Radiance (optical sensor)

<!-- https://labo.obs-mip.fr/multitemp/radiometric-quantities-irradiance-radiance-reflectance/ -->

A conventional way of deriving Top Of Atmosphere (TOA) Radiance in ![formula](https://render.githubusercontent.com/render/math?math=W.sr^{-1}.m^{-3}) from DN values using `scale` and `offset` in the following formula:

![formula](https://render.githubusercontent.com/render/math?math=L_\lambda%20=%20scale%20\times%20DN%20%2B%20offset)

where ![formula](https://render.githubusercontent.com/render/math?math=L_\lambda) is TOA Radiance in ![formula](https://render.githubusercontent.com/render/math?math=W.sr^{-1}.m^{-3}).

For example, the above value conversion is described in the values dictionary as

```json
"values": {
  "TOA radiance": {
      "unit": "W⋅sr−1⋅m−3",
      "scale": 0.0145,
      "offset": 3.48
  }
}
```

## Raster Composite Object

| Field Name        | Type                               | Description                                                                                                                             |
| ----------------- | ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| name              | string                             | **REQUIRED**. Denomination of the band composition (e.g. `ndvi`, `Color Infrared (vegetation)`  )                                       |
| nodata            | number                             | Pixel values used to identify pixels that are nodata in the composition .                                                               |
| range             | \[number]                          | range of valid pixels values in the composition                                                                                         |
| bands             | \[[Band Selector](#band-selector)] | **REQUIRED**. An array of bands selection where each object is a [Band Selector](#band-selector). If given, requires at least one band. |
| band_math_formula | string                             | Band math expression (e.g `(b4-b1)/(b4+b1)`)                                                                                            |


## Band Selector

| Field Name | Type   | Description                                    |
| ---------- | ------ | ---------------------------------------------- |
| asset_key  | string | **REQUIRED**. Asset key in the item  )         |
| band_index | number | **REQUIRED**. Band position index in the asset |

## Dynamic tile servers integration

Dynamic tile servers could exploit the information in the raster extension to automatically set some parameters.

The Sentinel-2 example will be used to illustrate the usage of the raster extension information for dynamic tiling.

### Titiler

[titiler](https://github.com/developmentseed/titiler) offers a native [STAC reader](https://github.com/developmentseed/titiler/blob/master/docs/endpoints/stac.md). Some query parameters could be set with the information from raster extension.

| <!--   | Query key                                  | value    | Example value |
| ------ | ------------------------------------------ | -------- |
| url    | STAC Item URL                              | REQUIRED |
| assets | list of comma (',') delimited asset names. |
expression: rio-tiler's band math expression (e.g B1/B2). OPTIONAL*
bidx: Comma (',') delimited band indexes. OPTIONAL
nodata: Overwrite internal Nodata value. OPTIONAL
rescale: Comma (',') delimited Min,Max bounds. OPTIONAL
color_formula: rio-color formula. OPTIONAL
colormap_name: rio-tiler color map name. OPTIONAL
colormap: JSON encoded custom Colormap. OPTIONAL -->