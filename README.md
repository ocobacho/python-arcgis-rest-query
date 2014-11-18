[![PyPI version](https://badge.fury.io/py/arcgis-rest-query.svg)](http://badge.fury.io/py/arcgis-rest-query) ![travis-ci status](https://travis-ci.org/Schwanksta/python-arcgis-rest-query.svg?branch=master)
# ArcGIS REST Query 

A simple library that can download a layer from a map in an 
ArcGIS web service and convert it to something useful,
like GeoJSON.

## Usage

```python
>>> import arcgis
>>> source = "http://services.arcgis.com/P3ePLMYs2RVChkJx/ArcGIS/rest/services/USA_Congressional_Districts/FeatureServer"
>>> arc = arcgis.ArcGIS(source)
>>> layer_id = 0
>>> shapes = arc.get(layer_id, "STATE_ABBR='IN'")
```

This assumes you've inspected your ArcGIS services endpoint to know what to look for.
ArcGIS DOES publish json files enumerating the endpoints you can query, so autodiscovery
could be possible further down the line.

## Installation

The easiest way:
```bash
pip install arcgis-rest-query
```
From source:

```bash
# Create a virtual environment (pip install virtualenv if you don't have it already)
virtualenv python-arcgis-rest-query
cd python-arcgis-rest-query
. bin/activate
git clone git@github.com:Schwanksta/python-arcgis-rest-query repo
cd repo
pip install -r requirements.txt
```

## From the command line

You can also use the included arcgis-get utility, like so:

*Note: this query will take a long time, so maybe try one of the other examples below?*
```bash
arcgis-get http://tigerweb.geo.census.gov/arcgis/rest/services/Basemaps/CommunityTIGER/MapServer 9 > ~/Desktop/railroads.geojson
```

This will download a Railroads layer from the US Census' TIGER dataset.

The size of that file brings up a good point: you should run `--count_only` before downloading an entire dataset, so you can see what you're in store for. 

```bash
$ arcgis-get http://tigerweb.geo.census.gov/arcgis/rest/services/Basemaps/CommunityTIGER/MapServer 9 --count_only
182149
```
Since we go in batches of 1,000, you're in for over 180 queries to the API.

# API
## Constructor
The ArcGIS() constructor takes only one argument, the URL to the web services endpoint you wish to query.
```python
>>> from arcgis import ArcGIS
>>> service = ArcGIS("http://tigerweb.geo.census.gov/arcgis/rest/services/Basemaps/CommunityTIGER/MapServer")
```
## ArcGIS.get(layer[,where="1 = 1", fields=[], count_only=False, srid='4326'])

Gets a single layer from the web service.

```python
>>> geojson = service.get(28)
>>> only_florida = service.get(28, where="NAME = 'Florida'")
>>> # Specifying the fields means we get only those fields in return
>>> only_florida_shape = service.get(28, where="NAME = 'Florida'", fields=['OBJECTID'])
```

If count_only is specified, we return a simple count of the number of features in the layer you're querying. This is useful for determining how big of a query you're about to make, or if your WHERE filter is correct.

```python
>>> states_count = service.get(28, count_only=True)
56
>>> southeast_count = service.get(28, where="NAME IN ('Florida', 'Georgia', 'Alabama', 'South Carolina')", count_only=True)
4
```

## ArcGIS.getMultiple(layers[, where="1 = 1", fields=[], srid='4326', layer_name_field=None])

Concatenate multiple layers into one geojson. This is useful if you have a map with layers for, say, every year, named foo_2014, foo_2013, foo_2012, etc. Setting layer_name_field adds a field to every returned object specifying which layer it came from.

```python
>>> service = ArcGIS("http://tigerweb.geo.census.gov/arcgis/rest/services/Census2010/Transportation/MapServer")
>>> # Get any primary or secondary roads named after MLK Jr. and combine them.
>>> mlk_roads = service.getMultiple([0,1], where="NAME LIKE '%Martin Luther King%'", layer_name_field="src_layer")
>>> # Inspect the src_layer field in the first returned feature.
>>> mlk_roads.get('features')[0].get('properties').get('src_layer')
u'Primary Roads'
```

## ArcGIS.get_json(layer[, where="1 = 1", fields=[], count_only=False, srid='4326'])

Returns the raw JSON from ArcGIS web services for the layer. This is not GeoJSON.

```python
>>> raw_json = service.get_json(0)
```

## ArcGIS.get_descriptor_for_layer(layer)

Returns the JSON descriptor for the layer. This tells you things like what fields are in the layer, what sort of geometry it contains, etc. The response of this function is cached, so repeated calls to the same layer will not hit the ArcGIS web service.

```python
>>> descriptor = service.get_descriptor(0)
```

### ArcGIS.enumerate_layer_fields(layer)

Returns a list of the field names in the layer. Useful for determining what you want to request in a .get() call.

```python
>>> field_list = service.enumerate_layer_fields(0)
```

## Piping to geojsonio

If you install [geojsonio-cli](https://github.com/mapbox/geojsonio-cli/), you can pipe output directly to a viewable map.

```bash
npm install -g geojsonio-cli
```

Then, we could re-do the previous query:

```bash
arcgis-get http://tigerweb.geo.census.gov/arcgis/rest/services/Basemaps/CommunityTIGER/MapServer 9 | geojsonio
```

And get some glorious mapped output: 
![geojsonio-example](https://cloud.githubusercontent.com/assets/20067/4998565/6be2e4f8-69a7-11e4-8aa1-d735bd1a7dac.png)

You can also do WHERE filtering from the command line. For example, if you want to get the Census' state shape for just Florida
and display it on geojson.io, you could do:

```bash
arcgis-get --where="NAME = 'Florida'" http://tigerweb.geo.census.gov/arcgis/rest/services/Basemaps/CommunityTIGER/MapServer 28 | geojsonio
```
![florida](https://cloud.githubusercontent.com/assets/20067/5001808/ee233ff6-69c7-11e4-9c3e-245aba847bb5.png)

## Potential pitfalls

Since you can only query in batches of 1,000, and sometimes these are millions of records, these operations could take a long time. Currently there's no status indicator on the CLI, so run `--count_only` first to see how long you might wait.
