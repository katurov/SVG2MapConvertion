# SVG to Map Convertion
A hobby project to make Serbian wine Map from different sources

## A converter

I've used [MyGeodata Converter](https://mygeodata.cloud/converter/) (Online GIS / CAD Data Conversion and Transformation Tool) which can consume SVG tags `<polygon />` and `<path />` and convert into KML (for Google Maps) or GPKG (sor QGIS or geopandas) and offers a useful preview. Sometimes it shows an [error of conversion](https://gis.stackexchange.com/questions/388638/converting-svg-files-to-geojson/483476#483476), I found two common reasons (ourside the XML-error):
* excess code (like `<style />` block)
* file too big

## How to add coordinates to SVG

This section have to be child of `<svg>` tag (on a first level):
```xml
<MetaInfo xmlns="http://www.prognoz.ru">
    <Geo>
        <GeoItem
            X="308.65"
            Y="553.10"
            Latitude="45.13230239388232"
            Longitude="19.796716975125058" />
        <GeoItem
            X="1003.40"
            Y="2039.52"
            Latitude="42.23265905178628"
            Longitude="21.69736866778565" />
    </Geo>
</MetaInfo>
```

Good practice: take two point from different corners of the picture (e.g. left-up and right-down).

## Solutions

**The best tool to edit SVG** is [Incscape](https://inkscape.org) - it is free and usefull. You can **find X/Y points** of the key point of the picture in it. Alse you can use its easy interface to select unneeded object to delete before conversion (it can help make file smaller). 

Use this post to [**convert paths to polygon**](https://alpha.inkscape.org/vectors/www.inkscapeforum.com/viewtopic8ac5.html?t=9597), paths are not the same as polygons for the maps. 

**You can edit curves** right on Google Map (in "My maps" application) or use QGIS for that (if you are brave enough).

**You can combine polygons from SVG** to multypolygon in python like this:
```python
import pandas
import geopandas 
import shapely

from shapely.geometry import Polygon
from shapely.ops import cascaded_union

serbia_regions = geopandas.read_file("Wine_regions_of_Serbia_from_SVG.gpkg")

Data = [
    ('West Morava', '', cascaded_union([serbia_regions.loc[2,"geometry"], serbia_regions.loc[3,"geometry"]])),
    ('Banat', '', cascaded_union([serbia_regions.loc[7,"geometry"], serbia_regions.loc[8,"geometry"], serbia_regions.loc[9,"geometry"]])),
    ('Subotica-Horgoš', '', cascaded_union([serbia_regions.loc[10,"geometry"], serbia_regions.loc[11,"geometry"], serbia_regions.loc[12,"geometry"]]))
]

data_df = pandas.DataFrame.from_records(Data, columns=["Title", "Description","geometry"])
geo_df  = geopandas.GeoDataFrame(data_df, geometry='geometry', crs='epsg:4326')

geo_df.to_file("Wine_regions_of_Serbia_multypoligons.gpkg", driver='GPKG')
```
