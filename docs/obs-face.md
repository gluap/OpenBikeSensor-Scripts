# obs-face

This is a toolkit for **f**iltering, **a**nnotating, **c**onsolidating and **e**xporting OpenBikeSensor measurements stored in the
[OpenBikeSensor CSV-format](https://github.com/openbikesensor/OpenBikeSensorFirmware/blob/master/docs/software/firmware/csv_format.mdhttps://github.com/openbikesensor/OpenBikeSensorFirmware/blob/master/docs/software/firmware/csv_format.md).


## Features

### Filtering

Measurements with invalid time stamp, location or distance measurement are filtered out. Further, only confirmed measurements are kept.

In addition, usernames are replaced by pseudonyms, and only selected information are kept before exporting for visualization.

### Annotation

Each measurement is assigned to one _way_ as described by the OpenStreetMap data. GPS positions are corrected by 
projecting it to the way (snapping). Further, for each *confirmed* measurement, information from OpenStreetMap such as street name are duplicated to the measurement.

### Consolidating

All confirmed and valid measurements are collected and consolidated in one file in Json format.

### Export

The consolidated measurements are converted to GeoJson format and exported for visualization as required by the [OpenBikeSensor visualization](https://github.com/openbikesensor/OpenBikeSensor-Scripts/tree/main/OBS-FACE-Visualization).

## Installation

Please refer to the general [Installation Instructions](../README.md).

## Usage

### Basic Usage

The default data directory is assumed to be `./data`.

Place all OpenBikeSensor CSV files in subdirectories of `./data/input`, such as:

* `./data/input/User1/`
* `./data/input/User2/`
* `./data/input/User3/`

The expected file format is defined [here](https://github.com/openbikesensor/OpenBikeSensorFirmware/blob/master/docs/software/firmware/csv_format.md). 

To filter, annotate and collect OpenBikeSensor measurements and export to GeoJson visualization data, run:

```bash
obs-face -ACV -D "Stuttgart"
```

For measurement annotation, OpenStreetMap geographic data from district
"Stuttgart" is used. Additional districts (Landkreise) can be added in this way
(see below).

For each CSV file in `./data/input`, one JSON file with annotated measurements as well as a log file is created in `./data/annotated`.

The consolidated, valid and confirmed measurements are collected in JSON format
in `data/collected/measurements.json`, the data exported for visualization will
be located in `data/visualization/measurements.json`. Futher,
`./data/visualization/roads.json` contains confirmed, valid measurements
consolidated to *road segments* in GeoJSON format.

### Advanced Usage

#### Add more districts

```bash
obs-face -ACV -D "Stuttgart" -D "Pforzheim" -D "Enzkreis" -D "Landkreis Böblingen" -D "Landkreis Ludwigsburg" -D "Rems-Murr-Kreis" -D "Landkreis Esslingen"`
```

The name of the region is matched with the `name` tag of OpenStreetMap
relations describing regions (Germany: *Landkreis*), i.e. having the tag
`boundary=administrative`, e.g.
[Stuttgart](https://www.openstreetmap.org/relation/2793104). 

### Base Directory

In case your data is located in a different directory, use the `-b
BASE_DIRECTORY` command line flag:

```bash
obs-face -ACV -D "Stuttgart" -b ./data_collection/Stuttgart/
```

The base directory defaults to `./data`. 

In structure of the base directory is as follows:

* `$BASE_DIRECTORY/input/` input CSV files to be processed
* `$BASE_DIRECTORY/annotated/` annotated measurements
* `$BASE_DIRECTORY/collected/measurements.json` annotated, filtered and collected measurements
* `$BASE_DIRECTORY/visualization/measurements.json` confirmed, valid measurements in GeoJson format 
* `$BASE_DIRECTORY/visualization/roads.json` confirmed, valid measurements consolidated to road segments in GeoJson format

The paths can be further customized, see `--help`.

### Exclude Input Files

To exclude subdirectories of the input, use:

```bash
obs-face -ACV -D "Stuttgart" -e User3/
```

All files in `./data/input/User3/` and subdirectories will be excluded. The
exclusion path `User3/` is considered relative to the base directory.

### Parallel Processing

Some of the time-consuming computations can be parallelized. Use the `-p`
command line flag to set the number of processors, e.g.:

```
obs-face -ACV -D "Stuttgart" -p 4
```

This will use 4 worker processes. Note that this also increases the memory
consumption significantly. 

### Map Cache

Downloaded OpenStreetMap maps and datastructures derived from them are cached
in `./cache`.  This reduces processing time on subsequent calls, however map
updates are ignored until the cache is flushed by deleting all files in the
`./cache` directory.

### All Command Line Options

```
-h, --help            show this help message and exit

-A, --annotate        annotates measurements using OSM data

-C, --collect         collects all confirmed and valid measurements and stores it in one file

-V, --visualization   creates the GeoJson data required by the OBS visualization

-i INPUT, --input INPUT
                      path to the location where CSV data files are located

-e INPUT_EXCLUDE, --input-exclude INPUT_EXCLUDE
                      data to be excluded, given by path prefix in the input directory tree

-b BASE_PATH, --base-path BASE_PATH
                      base path to where all data is stored

--path-annotated PATH_ANNOTATED
                      path for storing annotated data

--output-collected OUTPUT_COLLECTED
                      filename for storing collected data

--output-geojson-roads OUTPUT_GEOJSON_ROADS
                      filename for storing roads visualization GeoJson data

--output-geojson-measurements OUTPUT_GEOJSON_MEASUREMENTS
                      filename for storing measurement visualization GeoJson data

--path-cache PATH_CACHE
                      path where the visualization data will be stored

-D DISTRICT, --district DISTRICT
                      name of a district (Landkreis) from which the OSM data should be used, can be used several times

--left-hand-traffic   switches to left-hand traffic (otherwise: right-hand
                      traffic); right instead left sensor is used, and the
                      exported visualization is adapted
 
-p PARALLEL, --parallel PARALLEL
                      disables parallel processing if 0, otherwise defines the number of worker processes

--recompute           always recompute annotation results
```