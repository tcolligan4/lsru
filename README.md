# Landsat-ESPA-util
*Interface to ESPA API for Landsat surface reflectance data ordering*

Before, downloading Landsat surface reflectance data for a given area meant:
- Manually query the sceneIDs on Earth Explorer
- Saving these lists of sceneIDs to text files
- Manually uploading these files to ESPA to place the order
- Downloading the processed data with a download manager

Now, thanks to [mapbox usgs](https://github.com/mapbox/usgs), the [espa API](https://github.com/USGS-EROS/espa-api), this utility, and [espa bulk downloader](https://github.com/USGS-EROS/espa-bulk-downloader), it can all be done from the command line. What a relief!!

## Why can't I just retrieve my Landsat data from Earth Explorer, Amazon or Google cloud?

You can but it will be top of atmosphere (TOA) radiance, not surface reflectance. If you aim to have a scientific use of the data, you probably want to have surface reflectance.

The critical part between TOA radiance and surface reflectance is the atmospheric correction. That means that surface reflectance data are corrected for atmospheric effects, therefore providing accurate measurements of the target's spectral properties.

Today (October 2016), there are a few ways to obtain Landsat surface reflectance data (All of them have been processed by LEDAPS, the reference high level Landsat processing tool):

1. By ordering them via the ESPA system
  - this is what this utility helps you to do
  - ESPA does on demand pre-processing of full (reprojected) scenes or subsets
  - A cloud mask (fmask) and vegetation indices can also be added to the order
1. By ordering them from Google Earth Engine
  - GEE has a 'virtual' surface reflectance collection (processed on the fly by LEDAPS) and is currently ingesting the entired ESPA archive.
1. By downloading TOA data from any source and processing with a local installation of LEDAPS
  - If you have a server and you're good with system administration, why not...


## Usage

`lsru` (Landsat Surface Reflectance Utility) queries available data from the earth explorer API and order them via the espa API.

```
lsru --help
Usage: lsru [OPTIONS] COMMAND [ARGS]...

Options:
  --help  Show this message and exit.

Commands:
  login        Login to USGS Earth Explorer api Key expires...
  order        Place espa order for a list of Landsat scenes...
  order_batch  Query and order data from a list of...
  query        Perform a spatial query on the Earth Explorer...
  sp_order     Query and order data for a given location
```

Every command has a help page. Run for example `lsru query --help`

### Query data

To be able to query data, you must run `lsru login` and enter your USGS credentials. The API key generated is then valid for 1 hour, and is accessed automatically by the other functions.
After that you can perform the query using either 

```
# Perform a query using center coordinates and radius
lsru query --collection LE7 --long_0 44.78 --lat_0 4.71 --radius 2000 --start_date 2015-01-01
```
will return the following list of Landsat scene IDs

```
LE71640572016276SG100
LE71640572016260SG100
LE71640572016244SG100
LE71640572016228SG100
LE71640572016212SG100
LE71640572016196SG100
LE71640572016180SG100
LE71640572016164SG100
LE71640572016148SG100
LE71640572016132SG100
LE71640572016116SG100
LE71640572016100SG100
LE71640572016084SG100
LE71640572016068SG100
LE71640572016052NPA00
LE71640572016036NPA00
LE71640572016020NPA00
LE71640572016004NPA00
LE71640572015353NPA00
LE71640572015337NPA00
LE71640572015321NPA00
LE71640572015305NPA00
LE71640572015289NPA00
LE71640572015257SG100
LE71640572015241SG100
LE71640572015225SG100
LE71640572015209SG100
LE71640572015193SG100
LE71640572015177SG101
LE71640572015161SG100
LE71640572015145PFS00
LE71640572015129PFS00
LE71640572015113PFS00
LE71640572015097PFS00
LE71640572015081PFS00
LE71640572015065PFS00
LE71640572015049PFS00
LE71640572015033PFS00
LE71640572015017PFS00
LE71640572015001PFS00
```

Which you can directly write to a file, and reuse later for your order.

```
# Query data using center coordinates and radius and write output to file
lsru query --collection LE7 --long_0 44.78 --lat_0 4.71 --radius 2000 --start_date 2015-01-01 > ~/sceneList.txt
```

The query can be done either via bounding box coordinates (`xmin`, `xmax`, `ymin`, `ymax`), or via center coord and radius (`long_0`, `lat_0`, `radius`), or spatial object (filename)

### Order data on espa

Using the scene list previously generated

```
lsru order --proj aea --resize --long_0 44.78 --lat_0 4.71 --radius 2000 ~/sceneList.txt
```

You can also do this in 1 step, using the `lsru sp_order`

```
lsru sp_order --collection LE7 --long_0 4.71 --lat_0 44.78 --radius 2000 --start_date 2015-01-01 --proj aea --resize
```


### Batch query + order

If you want to order data for multiple locations, it's possible. Save your coordinates to a file (left column is longitude and right column is latitude) as shown below and run `lsru order_batch`

```
4.71,44.78
5.66,51.96
89.00,17.18
```

For example, if I want to order post 2014 LE7 data for a 2000 m radius around these 3 locations, I can save the file as `filename.csv` and use:

```
lsru order_batch --collection LE7 --radius 2000 --proj aea --start_date 2014-01-01 --resize filename.csv
```

### Download data

See [espa bulk downloader](https://github.com/USGS-EROS/espa-bulk-downloader).


## Installing landsat-espa-util

First you must have geos and gdal installed.

```
sudo apt-get install libgeos-dev libgdal1-dev
```

Then, preferably in a virtualenv, run:

```
pip install git+https://github.com/loicdtx/landsat-espa-util.git
```

![](https://i.imgflip.com/1c7eet.jpg)


## TODO

- Make fiona (and shapely) dependencies optional
- Figure out SSL certificate stuff
- How to pass products as cli arguments
- Fully test --file option
- python 3 ???
- Add support for all espa projections
- Pass comment to espa order
- Make orderList not crash when empty scene_list is provided
- Setup a downloader with schedule