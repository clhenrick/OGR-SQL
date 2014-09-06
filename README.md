OGR SQL Tutorial
================

OGR2OGR is part of the GDAL utility that can be run in the command line (Terminal in Mac and Linux or Powershell in Windows). The benefit of using OGR is that you can process data in mulitple steps with a single command. You can also script the commands to make them easily repeatable as well as batch process multiple files in a directory.

### OGR comes with two utilities:  

- `ogrinfo` for getting information about a dataset (projection, datum, etc.) only. You can't alter the data this way.
- `ogr2ogr` for creating new data from old data such as reprojecting, clipping or querying with SQL.

### The syntax of using OGRINFO is like so:

`ogrinfo <command> <(optional) paramater(s)> file_name`

### The syntax of using OGR2OGR is like so:  

`ogr2ogr <command> <(optional) paramater(s)> output_file input_file`

### Resources:
*  [OGR SQL](http://www.gdal.org/ogr/ogr_sql.html)
*  [Nathaniel Kelso's Geo How To Wiki](https://github.com/nvkelso/geo-how-to/wiki/OGR-to-reproject,-modify-Shapefiles)
*  [Sara Safavi's Intro to OGR tutorial](http://www.sarasafavi.com/intro-to-ogr-part-i-exploring-data.html)

## The Examples
The following two examples come from [Sara Safavi](http://www.sarasafavi.com/)'s awesome [tutorial on using OGR](http://www.sarasafavi.com/intro-to-ogr-part-i-exploring-data.html).

### Get all the column names, data types and projection info
`ogrinfo -so city_of_austin_parks.shp -sql "SELECT * FROM city_of_austin_parks"`

Will output this:  

```
INFO: Open of `city_of_austin_parks.shp'
          using driver `ESRI Shapefile' successful.

Layer name: city_of_austin_parks
Geometry: Polygon
Feature Count: 267
Extent: (3060414.510114, 10021121.749968) - (3167828.500010, 10161094.720022)
Layer SRS WKT:
PROJCS["NAD_1983_StatePlane_Texas_Central_FIPS_4203_Feet",
        GEOGCS["GCS_North_American_1983",
                DATUM["North_American_Datum_1983",
                        SPHEROID["GRS_1980",6378137.0,298.257222101]],
                PRIMEM["Greenwich",0.0],
                UNIT["Degree",0.0174532925199433]],
        PROJECTION["Lambert_Conformal_Conic_2SP"],
        PARAMETER["False_Easting",2296583.333333333],
        PARAMETER["False_Northing",9842500.0],
        PARAMETER["Central_Meridian",-100.3333333333333],
        PARAMETER["Standard_Parallel_1",30.11666666666667],
        PARAMETER["Standard_Parallel_2",31.88333333333333],
        PARAMETER["Latitude_Of_Origin",29.66666666666667],
        UNIT["Foot_US",0.3048006096012192]]
PARK_ID: Integer (10.0)
PARK_NAME: String (100.0)
PARK_TYPE: String (30.0)
PARK_ADDRE: String (100.0)
PARK_STATU: String (30.0)
POLYGON_PA: Integer (5.0)
ACRES: Real (19.8)
CREATED_BY: String (10.0)
CREATED_DA: Date (10.0)
MODIFIED_B: String (10.0)
MODIFIED_D: Date (10.0)
PARK_LABEL: String (100.0)
SHAPE_AREA: Real (19.11)
SHAPE_LEN: Real (19.11)
```


### Return all field values for a single feature
`ogrinfo -q city_of_austin_parks.shp -sql "SELECT * FROM city_of_austin_parks" -fid 0`

Will output this:  

```
Layer name: city_of_austin_parks
OGRFeature(city_of_austin_parks):0
  PARK_ID (Integer) = 167
  PARK_NAME (String) = Givens
  PARK_TYPE (String) = District
  PARK_ADDRE (String) = 3811 E 12th St.
  PARK_STATU (String) = Developed
  POLYGON_PA (Integer) = 1
  ACRES (Real) =         41.43263805
  CREATED_BY (String) = (null)
  CREATED_DA (Date) = 0000/00/00
  MODIFIED_B (String) = ahardy
  MODIFIED_D (Date) = 2009/05/01
  PARK_LABEL (String) = Givens District Park
  SHAPE_AREA (Real) = 1804805.71356000006
  SHAPE_LEN (Real) =    5518.67090340000
  POLYGON ((3131001.980158895254135 (..etc...)
```

### Query all admin 0 capitals (country capitals) and create a new shapefile
  `ogr2ogr -sql "SELECT * FROM ne_50m_populated_places_simple WHERE featurecla='Admin-0 capital'"  ne_50m_pop_place_admin0_cap.shp ne_50m_populated_places_simple.shp`

###similar as above but returns a new shapefile only of admin 1 capitals 
`ogr2ogr -sql "SELECT * FROM ne_50m_populated_places_simple WHERE featurecla IN ('Admin-1 capital', 'Admin-1 region capital')"  ne_50m_pop_place_admin1_cap.shp ne_50m_populated_places_simple.shp`

### Query each value in the featurecla field with no duplicates:
`ogrinfo -sql "select DISTINCT featurecla from ne_50m_populated_places_simple"  ne_50m_populated_places_simple.shp ne_50m_populated_places_simple`
  
### Return the number of values in ADMO_A3 without duplicate values:
`ogrinfo -sql "select COUNT(DISTINCT ADM0_A3) from ne_50m_populated_places_simple"  ne_50m_populated_places_simple.shp ne_50m_populated_places_simple`  
  
--------------------
## Working with Road Data Extracted From OpenStreetMap
A single `roads.shp` file from either [Metro Extracts](https://mapzen.com/metro-extracts/) or [Extract.BBBike.org](http://extract.bbbike.org/) will contain many different tags (OSM classifications). The data is much easier to work with if you separate it out into multiple layers by grouping the tag names logically. The following commands separate an OSM extract roads.shp file containing every type of OSM road tag into different layers to make styling roads cartographically much easier. I also created a bash script that executes all of these commands in one sweep, it can be found [here](https://github.com/clhenrick/shell_scripts).

**Note:** being able to perform the 'group by' function in OGR SQL would be helpful for joining road geometry based on field names which helps with cartographic rendering, ie:  

`ogr2ogr -sql "select * from roads group by ('name', 'type', 'ref')" roads_grouped.shp roads.shp` 
         
But OGR sql does not support the 'group by' SQL aggregation command. However, this is possible with using PostGIS.

### First, let's take a look at what attributes are in the `type` field and order them alphabetically:
`ogrinfo -sql "select distinct type from roads order by type" roads.shp roads`
  
### Query all freeways from a osm roads.shp and create a new shapefile:
`ogr2ogr -sql "select * from roads where type in ('motorway', 'trunk')"  superhwy_osm.shp roads.shp`
    
Same but reprojecting to a new CRS:  
`ogr2ogr -sql "select * from roads where type in ('motorway', 'trunk')" -t_srs EPSG:2274 superhwy_osm_2274.shp roads.shp`

### Query freeway ramps / links:
`ogr2ogr -sql "select * from roads where type in ('motorway_link', 'trunk_link')"  superhwy_links_osm.shp roads.shp`  

Same but reprojecting to a new CRS:
`ogr2ogr -sql "select * from roads where type in ('motorway_link', 'trunk_link')"  -t_srs EPSG:2274 superhwy_links_osm_2274.shp roads.shp`

### Query all main roads... :
`ogr2ogr -sql "select * from roads where type in ('primary','secondary','tertiary')"  main-rd_osm.shp roads.shp`  

Same but reprojecting to a new CRS:
`ogr2ogr -sql "select * from roads where type in ('primary','secondary','tertiary')" -t_srs EPSG:2274 main-rd_osm_2274.shp roads.shp`

### Query all other types of road links (to keep if needed at larger scale maps):
`ogr2ogr -sql "select * from roads where type in ('primary_link','secondary_link','tertiary_link')"  main-rd_links_osm.shp roads.shp` 
 
Same but reprojecting to a different CRS:
`ogr2ogr -sql "select * from roads where type in ('primary_link','secondary_link','tertiary_link')" -t_srs EPSG:2274 main-rd_links_osm_2274.shp roads.shp`

#### Or use the like operator:
`ogr2ogr -sql "select * from roads where type like '%link' and type != 'motorway_link'" main-rd_links_osm.shp roads.shp`

### Query all local roads:
`ogr2ogr -sql "select * from roads where type in ('residential', 'service', 'living_street', 'unclassified')"  other-rd_osm.shp roads.shp`  

Same but reprojecting to a different CRS:
`ogr2ogr -sql "select * from roads where type in ('residential', 'service', 'living_street', 'unclassified')" -t_srs EPSG:2274 other-rd_osm_2274.shp roads.shp`

### Query all dirt roads:
`ogr2ogr -sql "select * from roads where type = 'track'"  dirt-rd_osm.shp roads.shp` 

Same but reprojecting to a different CRS: 
`ogr2ogr -sql "select * from roads where type = 'track'" -t_srs EPSG:2274 dirt-rd_osm_2274.shp roads.shp `

### Query all pedestrian roads / paths / cycleways / etc:
`ogr2ogr -sql "select * from roads where type in ('bridleway', 'cycleway', 'footway', 'path', 'pedestrian', 'steps')"  ped-trail_osm.shp roads.shp`   

Same but reprojecting to a different CRS:
`ogr2ogr -sql "select * from roads where type in ('bridleway', 'cycleway', 'footway', 'path', 'pedestrian', 'steps')" -t_srs EPSG:2274 ped-trail_osm_2274.shp roads.shp`

------------
## Working with Data Outputted From [Skeletron](https://github.com/migurski/Skeletron)
Skeletron is a python script that takes OSM xml data as input and generates output GeoJSON data. It will generalize road by taking roads that contain separate lines but share the same name and making them a single line. This is very useful for cartographically rendering highways and freeways at small scale zoom levels while also reducing the size of your dataset.

From Skeletron's README:

*Skeletron generalizes collections of lines to a specific spherical mercator
zoom level and pixel precision, using a polygon buffer and voronoi diagram as
described in a 1996 paper by Alnoor Ladak and Roberto B. Martinez, ["Automated
Derivation of High Accuracy Road Centrelines Thiessen Polygons Technique"](http://proceedings.esri.com/library/userconf/proc96/TO400/PAP370/P370.HTM).*


### Querying geojson data after skeletroning:
**Note:** I appended `z14` and `w13` to the output shapefile to represent the paramaters I passed to skeletron: generalizing the data at zoom level 14 with a line width of 13.

`ogr2ogr -sql "select * from OGRGeoJSON where highway in ('motorway', 'trunk')" -f "ESRI Shapefile" -t_srs EPSG:2274 superhwy_osm_gen_z14_w13_2274.shp nashville_z14_w13.json`  

`ogr2ogr -sql "select * from OGRGeoJSON where highway in ('primary','secondary')" -f "ESRI Shapefile" -t_srs EPSG:2274 main-rd_osm_gen_z14_w13_2274.shp nashville_z14_w13.json`


*************************************
## Working with Natural Feature Data Extracted From OpenStreetMap

###list type attributes:
`ogrinfo -sql "select distinct type from natural order by type" natural.shp natural`

###create a water polygon layer from natural.shp:
`ogr2ogr -sql "select * from natural where type in ('riverbank', 'water')" -t_srs EPSG:2274 water-poly_osm_2274.shp natural.shp`

###create a parks polygon layer from natural.shp:
`ogr2ogr -sql "select * from natural where type = 'park'" -t_srs EPSG:2274 parks_osm_2274.shp natural.shp` 

*************************************
## Working with Flickr Alpha Shapes
* *TO DO: Query out a substring of the name column and create a new field from it*

### query out all neighborhoods in Nashville, TN, US
`ogr2ogr -sql "select label as name from OGRGeoJSON where (label LIKE '%Nashville, TN, US%')" -f "ESRI Shapefile" -t_srs EPSG:2274 hoods_flickr_nv_test_2274.shp flickr_shapes_neighbourhoods.geojson` 
