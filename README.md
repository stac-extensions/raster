# Raster Extension Specification

- **Title:** Raster
- **Identifier:** <https://stac-extensions.github.io/raster/v1.1.0/schema.json>
- **Field Name Prefix:** raster
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @emmanuelmathot

This document explains the Raster Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.

An item can describe assets that are rasters of one or multiple bands with some information common to them all (raster size, projection)
and also specific to each of them (data type, unit, number of bits used, nodata).
A raster is often strongly linked with the georeferencing transform and coordinate system definition
of all bands (using the [projection extension](https://github.com/radiantearth/stac-spec/tree/master/extensions/projection)).
In many applications, it is interesting to have some metadata about the rasters in the asset (values statistics, value interpretation, transforms).

- Examples:
  - [Planet Item example](examples/item-planet.json): Shows the basic usage of the extension in a STAC Item
  - [Sentinel-2 Item example](examples/item-sentinel2.json): Shows the statistics about individual bands and some RGB composites example
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Item Asset fields

| Field Name   | Type                                         | Description                                                                                                                     |
| ------------ | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| raster:bands | \[[Raster band Object](#raster-band-object)] | An array of available bands where each object is a \[[Band Object](#raster-band-object)]. If given, requires at least one band. |

## Raster Band Object

When specifying a raster band object at asset level, it is recommended to use
the [projection](https://github.com/radiantearth/stac-spec/tree/master/extensions/projection) extension
to specify information about the raster projection, especially `proj:shape` to specify the height and width of the raster.

| Field Name         | Type                                    | Description                                                                                                                                                                      |
| ------------------ | --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| nodata             | number\|string                          | Pixel values used to identify pixels that are nodata in the band either by the pixel value as a number or `nan`, `inf` or `-inf` (all strings).                                                                                                               |
| sampling           | string                                  | One of `area` or `point`. Indicates whether a pixel value should be assumed to represent a sampling over the region of the pixel or a point sample at the center of the pixel.   |
| data_type          | string                                  | The data type of the pixels in the band. One of the [data types as described below](#data-types).                                                                                |
| bits_per_sample    | number                                  | The actual number of bits used for this band. Normally only present when the number of bits is non-standard for the `datatype`, such as when a 1 bit TIFF is represented as byte. |
| spatial_resolution | number                                  | Average spatial resolution (in meters) of the pixels in the band.                                                                                                                |
| statistics         | [Statistics Object](#statistics-object) | Statistics of all the pixels in the band.                                                                                                                                         |
| unit               | string                                  | Unit denomination of the pixel value.                                                                                                                                             |
| scale              | number                                  | Multiplicator factor of the pixel value to transform into the value (i.e. translate digital number to reflectance).                                                              |
| offset             | number                                  | Number to be added to the pixel value (after scaling) to transform into the value (i.e. translate digital number to reflectance).                                                |
| histogram          | [Histogram Object](#histogram-object)   | Histogram distribution information of the pixels values in the band.                                                                                                              |

`scale` and `offset` define parameters to compute another value. The following paragraphs describe some use cases.

### Data Types

The data type gives information about the values in the file.
This can be used to indicate the (maximum) range of numerical values expected.
For example `uint8` indicates that the numbers are in a range between 0 and 255,
they can never be smaller or larger. This can help to pick the optimal numerical
data type when reading the files to keep memory consumption low.
Nevertheless, it doesn't necessarily mean that the expected values fill the whole range.
For example, there can be use cases for `uint8` that just use the numbers 0 to 10 for example.
Through other extensions it might be possible to specify an exact value range so
that visualizations can be optimized.
The allowed values for `data_type` are:

- `int8`: 8-bit integer
- `int16`: 16-bit integer
- `int32`: 32-bit integer
- `int64`: 64-bit integer
- `uint8`: unsigned 8-bit integer (common for 8-bit RGB PNG's)
- `uint16`: unsigned 16-bit integer
- `uint32`: unsigned 32-bit integer
- `uint64`: unsigned 64-bit integer
- `float16`: 16-bit float
- `float32`: 32-bit float
- `float64`: 64-big float
- `cint16`: 16-bit complex integer
- `cint32`: 32-bit complex integer
- `cfloat32`: 32-bit complex float
- `cfloat64`: 64-bit complex float
- `other`: Other data type than the ones listed above (e.g. boolean, string, higher precision numbers)

### Statistics Object

| Field Name    | Type   | Description                                        |
| ------------- | ------ | -------------------------------------------------- |
| mean          | number | mean value of all the pixels in the band           |
| minimum       | number | minimum value of the pixels in the band            |
| maximum       | number | maximum value of the pixels in the band            |
| stddev        | number | standard deviation value of the pixels in the band |
| valid_percent | number | percentage of valid (not `nodata`) pixel           |

### Use Scale and offset as radiometric calibration parameters

In remote sensing, many imagery raster corresponds to raw data without any radiometric processing.
Each pixel is given in digital numbers (DN), i.e. native pixel values from the sensor acquisition.
Those digital numbers quantify the energy recorded by the detector (optical or radar).
The sensor radiometric calibration aims to turn back the DN value into a
physical unit value (radiance, light power, backscatter).
Hereafter, some examples of the usage of the `values` dictionary to perform radiometric correction.

#### Digital Numbers to Radiance (optical sensor)

<!-- https://labo.obs-mip.fr/multitemp/radiometric-quantities-irradiance-radiance-reflectance/ -->

A conventional way of deriving Top Of Atmosphere (TOA) Radiance
in ![formula](https://render.githubusercontent.com/render/math?math=W.sr^{-1}.m^{-3})
from DN values using `scale` and `offset` in the following formula:

![formula](https://render.githubusercontent.com/render/math?math=L_\lambda%20=%20scale%20\times%20DN%20%2B%20offset)

where ![formula](https://render.githubusercontent.com/render/math?math=L_\lambda) is TOA Radiance
in ![formula](https://render.githubusercontent.com/render/math?math=W.sr^{-1}.m^{-3}).

For example, the above value conversion is described in the values dictionary as

```json
"assets": {
  "B4": {
      "title": "TOA reflectance",
      "raster:bands": [{
        "nodata": 0,
        "unit": "W⋅sr−1⋅m−3",
        "scale": 0.0145,
        "offset": 3.48
      }]
  }
}
```

#### Transform height measurement to water level

In remote sensing, radar altimeter instruments measures an absolute height from an absolute georeference (e.g. WGS 84 geoid).
In hydrology, you prefer having the water level relative to the "0 limnimetric scale".
Therefore, a usage of the value object here would be to indicate the offset between the reference height 0 of the sensor
and the 0 limnimetric scale to compute a water level.

In the following value definition example, 185 meters must be substracted from the pixel value to correspond to the water level.

```json
"assets": {
  "WaterLevel": {
      "title": "Water Level at station",
      "raster:bands": [{
        "unit": "m",
        "offset": -185
      }]
  }
}
```

### Histogram Object

The distribution of pixel values of a band can be provided with a histogram object. Those values are sampled in buckets.
A histogram object is atomic and all fields are **REQUIRED**.

| Field Name | Type      | Description                                                                 |
| ---------- | --------- | --------------------------------------------------------------------------- |
| count      | number    | number of buckets of the distribution.                                      |
| min        | number    | minimum value of the distribution. Also the mean value of the first bucket. |
| max        | number    | minimum value of the distribution. Also the mean value of the last bucket.  |
| buckets    | \[number] | Array of integer indicating the number of pixels included in the bucket.    |

The information in histogram objects may be useful to prepare a user interface in the perspective of the manipulation of the pixels value
for raster visualization such as true color composite balancing.

For instance, to enhance an image by changing properties such as brightness, contrast, and gamma through multiple stretch types
such as statistical functions.

Each bucket width all equals depending on the number of buckets. It can be computed with the following formula:
`Bucket width = ( max - min ) ÷ count`

![histogram](images/histogram.png)

The Histogram Object is part of the JSON document produced by [gdalinfo](https://gdal.org/programs/gdalinfo.html) command line tool
on the raster file with the `-hist` and `-json` argument. For instance

```console
gdalinfo -json -hist PT01S00_842547E119_8697242018100100000000MS00_GG001002003/PT01S00_842547E119_8697242018100100000000MS00_GG001002003.tif
```

produces this [file](gdalinfo.json) in wich there are `histogram` fields for each band.
The [planet example](examples/item-planet.json) includes them.
