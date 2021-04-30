# Raster Extension Specification

- **Title:** Raster
- **Identifier:** `https://stac-extensions.github.io/raster/v1.0.0/schema.json`
- **Field Name Prefix:** raster
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @emmanuelmathot

This document explains the Raster Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.

An item can describe assets that are rasters of one or multiple bands with some information common to them all (raster size, projection)
and also specific to each of them (data type, unit, number of bits used, nodata).
A raster is ofthen strongly linked with the the georeferencing transform and coordinate system definition
of all bands (using the [projection extension](https://github.com/radiantearth/stac-spec/tree/master/extensions/projection)).
In many applications, it is interesting to have some metadata about the raster in the asset (values statistics, value interpretation, transforms).

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
| nodata             | number                                  | Pixel values used to identify pixels that are nodata in the assets .                                                                                                             |
| sampling           | string                                  | One of `area` or `point`. Indicates whether a pixel value should be assumed to represent a sampling over the region of the pixel or a point sample at the center of the pixel.   |
| data_type          | string                                  | The data type of the band. One of the [data types as described above](#data-types).                                                                                              |
| bits_per_sample    | number                                  | The actual number of bits used for this band. Normally only present when the number of bits is non-standard for the `datatype`, such as when a 1 bit TIFF is represented as byte |
| spatial_resolution | number                                  | Average spatial resolution (in meters) of the pixels in the band.                                                                                                                |
| statistics         | [Statistics Object](#statistics-object) | Statistics of all the pixels in the band                                                                                                                                         |
| unit               | string                                  | unit denomination of the pixel value                                                                                                                                             |
| scale              | number                                  | multiplicator factor of the pixel value to transform into the value (i.e. translate digital number to reflectance).                                                              |
| offset             | number                                  | number to be added to the pixel value (after scaling) to transform into the value (i.e. translate digital number to reflectance).                                                |
| gdalinfo | [gdalinfo Band Object](#gdalinfo-band-object) | [gdalinfo](https://gdal.org/programs/gdalinfo.html) Band object representing information about a band in a raster dataset. |

`scale` and `offset` defines parameters to compute another value. Next paragraphs describe some use cases.

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
The allowed values for `file:data_type` are:

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
| stdev         | number | standard deviation value of the pixels in the band |
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

### Transform height measurement to water level

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

## [gdalinfo](https://gdal.org/programs/gdalinfo.html) band Object

[gdalinfo](https://gdal.org/programs/gdalinfo.html) commands lists exhaustive information about a raster dataset. It provides options to get statistics and histograms of the values in the bands of the raster. This object can be obtained from GDAL([GDALRasterBand](https://gdal.org/doxygen/classGDALRasterBand.html)). To get it on the command line you can use the `gdalinfo` CLI with the info command: `$ gdalinfo -json` and get the corresponfing object in the array of bands.

Example:

```console
$ gdalinfo -json PT01S00_842547E119_8697242018100100000000MS00_GG001002003/PT01S00_842547E119_8697242018100100000000MS00_GG001002003.tif
{
  "description":"PT01S00_842547E119_8697242018100100000000MS00_GG001002003.tif",
  "driverShortName":"GTiff",
  "driverLongName":"GeoTIFF",
  [...]
}
```

returns a json with a property `bands`. Every item in the array can be used in the `gdalinfo` field.

```json
{
  "bands":[
    {
      "band":1,
      "block":[
        256,
        256
      ],
      "type":"UInt16",
      "colorInterpretation":"Red",
      "min":1962.0,
      "max":32925.0,
      "minimum":1962.0,
      "maximum":32925.0,
      "mean":8498.94,
      "stdDev":5056.129,
      "noDataValue":0.0,
      "overviews":[
        {
          "size":[
            2989,
            1471
          ]
        },
        {
          "size":[
            997,
            491
          ]
        },
        {
          "size":[
            333,
            164
          ]
        }
      ],
      "metadata":{
        "":{
          "STATISTICS_MAXIMUM":"32925",
          "STATISTICS_MEAN":"8498.9400644319",
          "STATISTICS_MINIMUM":"1962",
          "STATISTICS_STDDEV":"5056.1292002722",
          "STATISTICS_VALID_PERCENT":"61.09"
        }
      }
    },
    {
      "band":2,
      "block":[
        256,
        256
      ],
      "type":"UInt16",
      "colorInterpretation":"Green",
      "min":3884.0,
      "max":22063.0,
      "minimum":3884.0,
      "maximum":22063.0,
      "mean":7185.212,
      "stdDev":3799.456,
      "noDataValue":0.0,
      "overviews":[
        {
          "size":[
            2989,
            1471
          ]
        },
        {
          "size":[
            997,
            491
          ]
        },
        {
          "size":[
            333,
            164
          ]
        }
      ],
      "metadata":{
        "":{
          "STATISTICS_MAXIMUM":"22063",
          "STATISTICS_MEAN":"7185.2123645206",
          "STATISTICS_MINIMUM":"3884",
          "STATISTICS_STDDEV":"3799.4562788636",
          "STATISTICS_VALID_PERCENT":"61.09"
        }
      }
    },
    {
      "band":3,
      "block":[
        256,
        256
      ],
      "type":"UInt16",
      "colorInterpretation":"Blue",
      "min":1061.0,
      "max":29693.0,
      "minimum":1061.0,
      "maximum":29693.0,
      "mean":5829.584,
      "stdDev":4683.065,
      "noDataValue":0.0,
      "overviews":[
        {
          "size":[
            2989,
            1471
          ]
        },
        {
          "size":[
            997,
            491
          ]
        },
        {
          "size":[
            333,
            164
          ]
        }
      ],
      "metadata":{
        "":{
          "STATISTICS_MAXIMUM":"29693",
          "STATISTICS_MEAN":"5829.583942362",
          "STATISTICS_MINIMUM":"1061",
          "STATISTICS_STDDEV":"4683.0650025253",
          "STATISTICS_VALID_PERCENT":"61.09"
        }
      }
    },
    {
      "band":4,
      "block":[
        256,
        256
      ],
      "type":"UInt16",
      "colorInterpretation":"Undefined",
      "min":1685.0,
      "max":31836.0,
      "minimum":1685.0,
      "maximum":31836.0,
      "mean":6785.251,
      "stdDev":3842.432,
      "noDataValue":0.0,
      "overviews":[
        {
          "size":[
            2989,
            1471
          ]
        },
        {
          "size":[
            997,
            491
          ]
        },
        {
          "size":[
            333,
            164
          ]
        }
      ],
      "metadata":{
        "":{
          "STATISTICS_MAXIMUM":"31836",
          "STATISTICS_MEAN":"6785.2511255265",
          "STATISTICS_MINIMUM":"1685",
          "STATISTICS_STDDEV":"3842.4324936707",
          "STATISTICS_VALID_PERCENT":"61.09"
        }
      }
    }
  ]
```
