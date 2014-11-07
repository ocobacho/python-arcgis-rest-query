# ArcGIS REST Query 

A simple library that can download a layer from a map in an 
ArcGIS web service and convert it to something useful,
like GeoJSON.

TODO: Everything.
- Download all data (loop through with offsets until we get all the data)
- Output to a useful format (right now I'm just returnin the raw json)

Usage:

```python
>>> import arcgis
>>> arc = arcgis.ArcGIS("http://www.pathtomapserver:6080/", "FOLDER_OF_PROJECTS", "MAP_DATA_I_WANT")
>>> layer_id = 1
>>> arc.get(layer_id)
```
This assumes you've inspected your ArcGIS services endpoint to know what to look for.
ArcGIS DOES publish json files enumerating the endpoints you can query, so autodiscovery
could be possible further down the line.