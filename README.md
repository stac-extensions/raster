# Raster Extension Specification

- **Title:** Raster
- **Identifier:** <https://stac-extensions.github.io/raster/v1.1.0/schema.json>
- **Field Name Prefix:** raster
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Candidate
- **Owner**: @emmanuelmathot

This document explains the Raster Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.

An item can describe assets that are rasters of one or multiple bands with some information common to them all (raster size, projection)
and also specific to each of them (number of bits used).
A raster is often strongly linked with the georeferencing transform and coordinate system definition
of all bands (using the [projection extension](https://github.com/radiantearth/stac-spec/tree/master/extensions/projection)).
In many applications, it is interesting to have some metadata about the rasters in the asset (values statistics, value interpretation, transforms).

- Examples:
  - [Planet Item example](examples/item-planet.json): Shows the basic usage of the extension in a STAC Item
  - [Sentinel-2 Item example](examples/item-sentinel2.json): Shows the statistics about individual bands and some RGB composites example
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [ ] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [x] Bands
- [ ] Links

When using the raster extension, it is recommended to use
the [projection](https://github.com/radiantearth/stac-spec/tree/master/extensions/projection) extension
to specify information about the raster projection, especially `proj:shape` to specify the height and width of the raster.

| Field Name         | Type                                    | Description                                                                                                                                                                       |
| ------------------ | --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| raster:sampling           | string                                  | One of `area` or `point`. Indicates whether a pixel value should be assumed to represent a sampling over the region of the pixel or a point sample at the center of the pixel.    |
| raster:bits_per_sample    | number                                  | The actual number of bits used for this band. Normally only present when the number of bits is non-standard for the `datatype`, such as when a 1 bit TIFF is represented as byte. |
| raster:spatial_resolution | number                                  | Average spatial resolution (in meters) of the pixels in the band.                                                                                                                 |
| raster:scale              | number                                  | Multiplicator factor of the pixel value to transform into the value (i.e. translate digital number to reflectance).                                                               |
| raster:offset             | number                                  | Number to be added to the pixel value (after scaling) to transform into the value (i.e. translate digital number to reflectance).                                                 |
| raster:histogram          | [Histogram Object](#histogram-object)   | Histogram distribution information of the pixels values in the band.                                                                                                              |

`raster:scale` and `raster:offset` define parameters to compute another value. The following paragraphs describe some use cases.

### Scale and Offset Uses and Examples

In remote sensing, most imagery raster corresponds to just unitless raw pixel values that may be converted
into specific units given a scale and an offset. The raw pixel values are referred to as 
Digital Numbers (DN). Using a Scale and Offset simply provide a more efficient
way to store data with less bytes. In these cases the data provider will include scale and offset
values for transforming the data into a physical measurement, such as radiance, power, altitude, or
backscatter. Several examples are given below.

Users should be careful to always apply any provided scale and offset

#### DN to Reflectance

A very common use case is to store reflectance values, which range from 0 - 1.0, as integers rather than
utilizing the larger floating point data type. Data is stored in a 2-byte Integer and ranges from
1 to 10,0000 by using a scale of 0.0001, resulting in a file half the size of one using 4 byte floats.

```json
"assets": {
  "B4": {
      "title": "TOA radiance band 4",
      "bands": [{
        "raster:nodata": 0,
        "raster:scale": 0.0001,
        "raster:offset": 0.0
      }]
  }
}
```

#### Digital Numbers to Optical Radiance

A conventional way of deriving Top Of Atmosphere (TOA) Radiance from $\mathrm{DN}$
values using `scale` and `offset` in the following formula:

$$L_\lambda=\mathrm{scale}\times\mathrm{DN}+\mathrm{offset}$$

where $L_\lambda$ is TOA Radiance in $\mathrm{W}\!\cdot\!sr^{-1}\!\cdot\!m^{-3}$.

For example, the above value conversion is described in the values dictionary as

```json
"assets": {
  "B4": {
      "title": "TOA radiance band 4",
      "bands": [{
        "nodata": 0,
        "unit": "W⋅sr−1⋅m−2",
        "raster:scale": 0.0145,
        "raster:offset": 3.48
      }]
  }
}
```

##### Radiance to TOA Optical Reflectance

In order to convert the above TOA radiance to TOA reflectance, the following formula can be used:

$$R=\frac{pi \times L \times d \times d}{ESUN(b) \times cos(s)}$$

where:

- $L$ is the spectral radiance for the band (see previous section)
- $d$ is the earth-sun distance (in astronomical units) and depends on the acquisition’s day and month
- $ESUN(b)$ is the mean TOA solar irradiance (or [solar illumination](https://github.com/stac-extensions/eo#solar_illumination))
  in $W/m^2/micrometers$
- $s$ is the [solar zenith angle](https://github.com/stac-extensions/view#item-properties) in degrees.

source: <https://www.orfeo-toolbox.org/CookBook/Applications/app_OpticalCalibration.html>

#### Altitude to water level

In remote sensing, radar altimeter instruments measures an absolute height from an absolute georeference (e.g. WGS 84 geoid).
In hydrology, you prefer having the water level relative to the "0 limnimetric scale".
Therefore, a usage of the value object here would be to indicate the offset between the reference height 0 of the sensor
and the 0 limnimetric scale to compute a water level.

In the following value definition example, 185 meters must be substracted from the pixel value to correspond to the water level.

```json
"assets": {
  "WaterLevel": {
      "title": "Water Level at station",
      "bands": [{
        "unit": "m",
        "raster:offset": -185
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

## Relation types

The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type | Description                                                                                                                                                         |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| gcps | This link points to a [document](https://gdal.org/drivers/raster/vrt.html#vrtdataset) providing with a list of Ground Control Points for the dataset, mapping between pixel/line coordinates and georeferenced coordinates |

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
