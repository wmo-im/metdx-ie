# SG-FIT Least Disruptive update  
  
## Draft document for INFCOM-4: 
- [https://github.com/wmo-im/metdx-ie/blob/main/metdx-ie/sg-fit-least-disruptive-oct-2025-share.md](https://github.com/wmo-im/metdx-ie/blob/main/metdx-ie/sg-fit-least-disruptive-oct-2025-share.md)  
- Probably the basis of an “information paper” - recommendations need to be more concise :)  
- Still work in progress  
  
## Hackathon participants:  
- Met Office: Mark Burgoyne and Jeremy Tandy  
- Environment Canada: Tom Kralidis  
- ECMWF: Tiago Quintino, Adam Warde, James Hawkes  
  
## Terminology:  
- Granule - a persisted object  
- Datacube - a hypercube of data from a single model run  
  
## Demo target:  
1. Web UI to discover remote datasets via WIS2 Global Discovery Catalogue  
2. Deploy data-servers over local data - and add those endpoints to the Web UI   
3. Add a resource from a data service end-point - browse a OGC-API Collection or a STAC Catalogue  
4. Display a 2-d coverage on a map (either image from ogcapi-maps or visualised data from ogcapi-edr)  << “layer manager”  
    1. With time-animation?  
    2. At vertical level (or just surface for demo)?  
5. Display a timeseries for a point << “time-series manager”  
6. Diff two resources (coverage, timeseries) to compare << via ogcapi-processes  
7. Select point value from map view or timeseries  
8. Average (or other statistics) over spatial polygon (or duration interval)   
  
## Hackathon outcomes:  
- Focus on weather prediction model output (NWP and MLWP) as first use case  

### Hypercube specification (a _profile_ for a Zarr datacube) ==to be documented==:

- Hypercube object  
        - Aim: implementation agnostic description of an n-dimensional array for environmental data … with weather prediction model output (NWP) as the first pass  
        - Specification will need to describe instances (i.e., what a single hypercube is), dimensions, etc.   
        - Describes data from a single model run  
        - Opinionated: one hypercube for 1 model run, single variable  
        - Must have consistent _domain_ - model run output often provided on multiple vertical reference systems (e.g., pressure levels, surface, whole-earth, etc.); implies several hypercube objects for each model run   
        - Hypercube described with [==standard?==] JSON  
            - Kerchunk: Zarr version 2  
            - VirtualiZarr: Zarr version 3  
        - Need mechanism to determine which hypercube object relates to which model run / vertical domain  
            - Deployment specific solution: pattern match on file-names (see ECCC/MSC pygeoapi plugin for reading the WAF)  
            - Standard solution: use STAC   

### OGC-API EDR profile for weather prediction model output: building on existing “MeteoGate” profile in EUMETNET space  

- GitHub repo: [https://github.com/EUMETNET/metocean-edr-profile](https://github.com/EUMETNET/metocean-edr-profile)   
        - Pull Request: [EUMETNET/metocean-edr-profile#75](https://github.com/EUMETNET/metocean-edr-profile/pull/75) “add Requirements and Conformance for NWP” (see branch [nwp](https://github.com/EUMETNET/metocean-edr-profile/tree/nwp))  
- Draft spec: [https://eumetnet.github.io/metocean-edr-profile/](https://eumetnet.github.io/metocean-edr-profile/)  
- Proposal:  
        - Focus on weather prediction model output  
        - Collection maps to a single model run (`instance_id`); one model run may maps several collections, each with consistent domain (model run outputs data on several vertical reference systems - pressure levels, surface, whole-earth, etc.)   
        - Parameter names should be consistent, concise, and human-readable (i.e., not numeric [GRIB] codes); define these for a small set of common parameters  
        - Profile includes suggested list of standardised “shortnames” that are used for parameters; initial parameter set is the 16 parameters defined in Manual on WIPPS for global deterministic weather prediction data; suggesting use of ECMWF shortnames (these are widely adopted); _shortnames are used in the `parameter_names.{parameter}.observedProperty.label.id`_; this way, client applications that support the profile can easily distinguish identical parameters published from different EDR end-points even when those end-points choose different ‘native’ parameter ids (`parameter_names.{parameter}.id`) … note: the so-called _native_ parameter ids are used in the API queries  
            - Example parameter sets (without shortnames) from EDR Collection: [GRIB](https://raw.githubusercontent.com/opengeospatial/ogcapi-environmental-data-retrieval/refs/heads/master/core/standard/examples/grib_4_2_parameters.json), [CF](https://raw.githubusercontent.com/opengeospatial/ogcapi-environmental-data-retrieval/refs/heads/master/core/standard/examples/cf_parameters.json)  
            - Parameter catalogue resources from ECMWF with candidate shortnames: [WMO Core](https://www.ecmwf.int/en/forecasts/dataset/wmo-core), [Open data](https://www.ecmwf.int/en/forecasts/datasets/open-data), [Parameter database (ec-codes)](https://codes.ecmwf.int/grib/param-db/)  
        - Collection declares which EDR query types it supports, e.g., `position` query  
```
    "data_queries":{
        "position":{
            "link":{
                "href":"https://example.com/collections/ca-eccc-msc-nwp-gdps/position",
                "rel":"data",
                "variables":{
                    "query_type":"position"
                }
            }
        }
    }
```
- _(note that URLs provided for the `data_queries` breaks the hypermedia model because a client application needs to know how to construct a valid request - see below for example with `coords` and `parameter-name` queryables)_  
- _Which EDR query-types must be supported by the end-point_?  
- Servers don’t interpolate between enumerated dimensions (instance, time-step, vertical level). On error, return a list valid list of values that can be used.   
- Servers should return CoverageJSON _as a minimum?_ - confirm MIME type for CoverageJSON (`vnd.cov+json`)  
- Use “unified additional dimensions” for defining enumerated dimensions (e.g., pressure levels, ensemble members)  
- Use built-in `extent.spatial` and `extent.temporal` for spatial x-y and time dimensions   
- Queryable: `parameter-name` list of values (enumeration from `parameter_names.[?].id`) _note: pygeoapi returns all parameters if you don’t specify_
- Queryable: `coords` spatial extent of data request, defined in WKT (clipped to limit defined in the `extent.spatial.bbox`)  
- _How does EDR service interpret queries_? (e.g., should a position query return a time-series `PointSeries` response?)   
- Example position query - coords=POINT( -75 45):  
```
https://example.com/collections/ca-eccc-msc-nwp-gdps/position?f=json&coords=POINT%28%20-75%2045%29&parameter-name=AirTemp

```

### OGC-API Maps profile for weather prediction model output: new profile in OGC space [new work item approved]  
- GitHub repo [opengeospatial/metocean-ogcapi-maps-profile](https://github.com/opengeospatial/metocean-ogcapi-maps-profile)  
- Draft spec: [https://docs.ogc.org/DRAFTS/26-002.html](https://docs.ogc.org/DRAFTS/26-002.html) [==currently not building, currently only a `scaffold`==]  
- Proposal:  
        - Focus on weather prediction model output  
        - Collection maps to a single model run (`instance_id`); one model run may maps several collections, each with consistent domain (model run outputs data on several vertical reference systems - pressure levels, surface, whole-earth, etc.)   
        - Keep as close to ogcapi-maps as possible  
        - Maps operation tied to collection (i.e., a collection may offer a maps end-point to visualise the data), defined using link-object with link relation `http://www.opengis.net/def/rel/ogc/1.0/map`   
        - Use built-in maps queryables `datetime`, `bbox` (or `bbox-crs`), `width` and `height`  
        - Use `subset` queryable for subsetting on dimensions; example dimensions include `instance_id` (the model run) and `z` (the vertical level)   
        - Add custom `parameter-name` queryable (_proposal to change to `properties`_) - a collection will often contain data for multiple parameters (i.e., physical quantities) and we need to be able to tell the maps end-point which parameter to visualise   
        - Dimensions and parameters for each collection are defined in the `/collections/{collection-id}/schema` end-point (see [OGC API - Common - Part 3: Common: Schemas](https://docs.ogc.org/DRAFTS/23-058r1.html))  
            - Note: if a collection supports both ogcapi-maps and ocgapi-edr the server needs to duplicate publication of some metadata in both the collection and schema resources - it’s grit, but it keeps us close to the spec and thus usable by tools that don’t know about the MetOcean profile   
        - Temporal extent:  
            - Collection relates to a dataset (a set of model runs) - temporal extent covers from beginning of first instance to end of last instance  
            - Individual instances (model runs) are described in the /collections/{collection-id}/schema resource   
        - Servers don’t interpolate between enumerated dimensions (instance, time-step, vertical level). On error, return a list valid list of values that can be used.  
        - Servers only return a 2-d image (horizontal x-y plane) - more sophisticated cases (e.g., vertical slices, Hovmöller diagram) may be added in future. If the query isn’t constrained, the server will choose how to reduce the data to a 2-d horizontal plane (i.e., standard ogcapi-maps queries will work, albeit with limited dimensionality)     
        - Example:  
```
https://example.com/collections/ca-eccc-msc-nwp-gdps/map?f=png&width=500&height=400&promperties=AirTemp2&datetime=2025-10-28T01:00:00Z&subset=vertical(0),instance_id("2026-01-16T00:00:00Z") 

```
- Discussion: [https://github.com/opengeospatial/ogcapi-maps/issues/142](https://github.com/opengeospatial/ogcapi-maps/issues/142)  

### Demonstrate use of OGC-API Processes (part 1) to render chart (time-series) from CoverageJSON input  
- TODO: Describe  

### Use of STAC Catalog/Collection for weather prediction model output  
- TODO: More work required here   
- A STAC Catalog provides a standard, machine-readable mechanism to browse through a collection of data resources  
- STAC Item may point to native GRIB assets and/or [Zarr] hypercubes  
- Example STAC catalog: [html](https://labs.metoffice.gov.uk/asdi/demo/browser/#/?.language=en), [json](https://labs.metoffice.gov.uk/asdi/demo/catalog/catalog.json)   
- At ECMWF, each field is described with a STAC Item [_example?_]  
- _Are there any extensions needed in the STAC resources for weather model prediction datasets, e.g., `datacube` and `projection`, or Mark’s (as yet unpublished) “link_temparal_extent” custom extension that allows a catalog-browse client to determine the date-range of resources (Catalog/Collection or Asset) without needing to pre-fetch those resources (this speeds up browsing a resource tree published using static JSON objects). `temporal_extent` can be used in a link-object for a Catalog/Collection, or in a asset-object for an Item. The value can be a time-instant, time-range, or repeating interval, expressed in ISO8601 (RFC3339) notation. Example below shows a single item with assets for each model-run time-step:_  

#### Example STAC Item:
```
{
  "stac_version": "1.0.0",
  "type": "Feature",
  "stac_extensions": [
    "https://stac-extensions.github.io/datacube/v2.2.0/schema.json",
    "https://stac-extensions.github.io/projection/v2.0.0/schema.json",
    "https://stac-extensions.github.io/link_temporal_extent/v2.0.0/schema.json"
  ],
  "id": "global-deterministic-10km:global_precipitation-2025-12-11-06",
  "title": "Global precipitation data for the 2025-12-11T06:00Z model run",
  "description": "Global model precipitation data for the 2025-12-11T06:00Z model run",
  "bbox": [-180, -90, 180, 90],
  "geometry": {
    "type": "Polygon",
    "coordinates": [
      [
        [-180, -90],
        [180, -90],
        [180, 90],
        [-180, 90],
        [-180, -90]
      ]
    ]
  },
  "collection": "global_precipitation-2025-12-11",
  "properties": {
    "start_datetime": "2025-12-11T06:00:00Z",
    "end_datetime": "2025-12-14T00:00:00Z",
    "datetime": null,
    "cube:dimensions": {
      "projection_x_coordinate": {
        "type": "spatial",
        "axis": "x",
        "extent": [-180, 180],
        "reference_system": "GEOGCS[\"unknown\",DATUM[\"unnamed\",ELLIPSOID[\"Sphere\",6371229,0,LENGTHUNIT[\"metre\",1,ID[\"EPSG\",9001]]]],PRIMEM[\"Greenwich\",0,ANGLEUNIT[\"degree\",0.0174532925199433,ID[\"EPSG\",9122]]],CS[ellipsoidal,2],AXIS[\"latitude\",north,ORDER[1],ANGLEUNIT[\"degree\",0.0174532925199433,ID[\"EPSG\",9122]]],AXIS[\"longitude\",east,ORDER[2],ANGLEUNIT[\"degree\",0.0174532925199433,ID[\"EPSG\",9122]]]]"
      },
      ...
    },
    "cube:variables": {
      ...
      "precipitation_rate": {
        "type": "data",
        "standard_name": "lwe_precipitation_rate",
        "description": "Precipitation rate",
        "unit": "m s-1",
        "dimensions": [
          "time",
          "longitude",
          "latitude"
        ]
      },
      ...
    },
    ...
  },
  "links": [
    ...
  ],
  "assets": {
    "PT0H step of the 2025-12-11T06:00Z run for precipitation_rate": {
      "href": "https://example.com/global-deterministic-10km/20251211T0600Z/20251211T0600Z-PT0000H00M-precipitation_rate.nc",
      "type": "application/x-netcdf",
      "title": "PT0H step of the 2025-12-11T06:00Z run for precipitation_rate",
      "temporal_extent": "2025-12-11T06:00Z",
      "roles": [
        "data"
      ]
    },
    ...
    "PT1H step of the 2025-12-11T06:00Z run for precipitation_rate": {
      "href": "https://example.com/global-deterministic-10km/20251211T0600Z/20251211T0700Z-PT0001H00M-precipitation_rate.nc",
      "type": "application/x-netcdf",
      "title": "PT1H step of the 2025-12-11T06:00Z run for precipitation_rate",
      "temporal_extent": "2025-12-11T07:00Z",
      "roles": [
        "data"
      ]
    },
    ...
  }
}

```

#### Other STAC considerations
- ECMWF proposal for Q3: _Quick Querying of Qubes_  
        - Single templated query into multiple STAC items - avoids latency associated with making many separate object requests  
        - Similar to Coalesced Chunk Retrieval Protocol CCRP [https://cloudnativegeo.org/blog/2025/09/redefining-cloud-native-with-the-coalesced-chunk-retrieval-protocol/](https://cloudnativegeo.org/blog/2025/09/redefining-cloud-native-with-the-coalesced-chunk-retrieval-protocol/)  
        - _Is this something we want to recommend_?  

## WCMP2 extension to support “add to” workflows that enable an application (e.g., WebUI) to visualise data (or maps)       
- Fully data-driven: start from a WIS2 Global Discovery Catalogue to discover datasets  
- Once an “interesting” dataset has been found, use the WCMP2 record (ogcapi-records) to drive the WebUI  
  - WCMP2 Extension: ==Clarify what properties must be included in the WCMP2 metadata record so that Web-applications can use the ogcapi end-points==  
### Collection  
  - WCMP2 record includes a collection resource, example:  
```
        {
            "href":"https://example.com/collections/ca-eccc-msc-nwp-gdps",
            "type":"application/json",
            "rel":"collection",
            "title":"GDPS"
        }

```
  - If the WCMP2 record contains a link-object with link relation `collection` it is probably pointing to an ogcapi end-point or a STAC catalog. Eitherway, it’s an interactive resource that can be queried or browsed [==need to confirm the link relation types used, notes say `data` is also an option?==]  
  - In a [GDC] search results page, add an icon indicating that the dataset is (somehow) interactive  
  - Don’t pre-fetch the collection resource(s) at this stage - that could be expensive (and/or slow)  
  - When viewing a specific WCMP2 record (i.e., the description of a single dataset), fetch and parse the collection resource. Example (add queryable `f=json` to force the server to return JSON content):  
```
https://example.com/collections/ca-eccc-msc-nwp-gdps?f=json
```
  - If the collection resource has an `id` property, that confirms it’s probably from the OGC stable  
     - (Note: ogcapis provide a {root}/conformance resource that describes what features (or operations) the _server_ supports. Unfortunately, this information is provided for the entire server, not the individual collection. There’s no way of determining conformance on a per-coverage basis, so best to rely on parsing the collection resource itself. That said, the {root}/conformance resource would tell you which version of the ogcapi-maps or ogcapi-edr spec it supports. Example below.)  
```
{
    "conformsTo":[
        "http://www.opengis.net/spec/ogcapi-common-1/1.0/conf/core",
        "http://www.opengis.net/spec/ogcapi-common-1/1.0/conf/html",
        "http://www.opengis.net/spec/ogcapi-common-1/1.0/conf/json",
        "http://www.opengis.net/spec/ogcapi-common-1/1.0/conf/landing-page",
        "http://www.opengis.net/spec/ogcapi-common-1/1.0/conf/oas30",
        "http://www.opengis.net/spec/ogcapi-common-2/1.0/conf/collections",
        "http://www.opengis.net/spec/ogcapi-edr-1/1.0/conf/core"
    ]
}
```
### EDR  
  - An EDR end-point describes the query-types it supports (one or more)
  - EDR `data_queries` types include: position, radius, area, cube, trajectory, corridor, items, locations, instances  
  - Example collection resource fragment for query-type `position`:  
```
    "data_queries":{
        "position":{
            "link":{
                "href":"https://2d688578eb75.ngrok-free.app/collections/ca-eccc-msc-nwp-gdps/position",
                "rel":"data",
                "variables":{
                    "query_type":"position"
                }
            }
        }
    }

```            
   - A collection may contain collections - for example, the collection resource for a weather prediction model (e.g., `aifs_surface`) will list all the instances of that model as a sub-collection (link-relation `collection`). This relationship is supported using the `instances` query-type  
     - _Should the collection always include an instance with id `latest`_?  
   - The collection resource should have enough information to construct a valid query for the query-type(s) offered by that collection. The WebUI may have to iterate through (i.e., fetching) several collections to get to a collection containing data  
     - Example OGC-API EDR end-point: [collections](https://labs.metoffice.gov.uk/edr/collections) , [aifs_surface collection](https://labs.metoffice.gov.uk/edr/collections/aifs_surface) , [instances (i.e., model runs) of the aifs_surface collection](https://labs.metoffice.gov.uk/edr/collections/aifs_surface/instances/), [instance of aifs_surface collection at 2026012700z](https://labs.metoffice.gov.uk/edr/collections/aifs_surface/instances/aifs_surface_2026012700z)

### Maps:  
- If the collection resource has a link object with link relation `http://www.opengis.net/def/rel/ogc/1.0/map` then it provides the ogcapi-maps operations   
- _Clarify whether maps end-point is available at the top-level collection (i.e., model), the instances (i.e., the model run at a specific time), or both_  
- Fetch the `/collections/{collection-id}/schema` resource to get the metadata needed to construct valid queries  
  
### STAC  
- A STAC resource will always declare a the `stac_version`  
```
  "stac_version": "1.1.0"

```
- A STAC resource may be of type `Catalog`, `Collection` or `Item`. Collection is a sub-type of Catalog providing some additional metadata. Item is a geojson Feature that points to actual data resources termed `assets`.  
- Links to Catalog or Collection resources contained by a STAC Catalog or Collection are provided in link-objects with link-relation `child`. Links to Item resources are provided in link-objects with link-relation `item`.  
- Links to the data resources are provided in an `assets` dictionary within a STAC Item.  
- One can browse through the Catalog/Collection(s) to find Items; depending on the MIME-type of the asset, the WebUI may be able to add the data resource to the user’s session, using the metadata in the Item to provide necessary context.   
- Example STAC catalog: [html](https://labs.metoffice.gov.uk/asdi/demo/browser/#/?.language=en), [json](https://labs.metoffice.gov.uk/asdi/demo/catalog/catalog.json)   
- Community tools such as [Radiant Earth’s STAC Browser](https://radiantearth.github.io/stac-browser/#/?.language=en) can be used to browse a STAC Catalog or Collection, e.g., [example](https://radiantearth.github.io/stac-browser/#/external/labs.metoffice.gov.uk/asdi/demo/catalog/catalog.json)  
- note: pygeoapi automatically builds a STAC Catalog over persisted files/objects; dynamically generated, single STAC Item per file/object; enable this function in the YAML config   

### And ...    
- A collection may support zero or more of the above options. The WebUI should provide a mechanism to interact with the Maps, EDR or STAC end-points (e.g., a button for each that then launches a dialogue box that helps a user build a valid ogcapi query or browse through a STAC catalog)  
    - See [Mark Burgoyne’s Example EDR Query Tool](https://labs.metoffice.gov.uk/edr/static/html/query.html) for interactions required to help a user build a valid EDR query  

## POC software  
- Code: [https://github.com/wmo-im/metdx-demo](https://github.com/wmo-im/metdx-demo)   
- C4 diagram: [https://github.com/wmo-im/metdx-demo/blob/main/docs/architecture/c4.container.png](https://github.com/wmo-im/metdx-demo/blob/main/docs/architecture/c4.container.png)  
- Containerised: Docker Compose stack  

### pygeoapi ogcapi reference implementation   
  - native support for xarray   
  - updates supporting the profiles now merged into core (main branch) [https://github.com/geopython/pygeoapi/pull/2214](https://github.com/geopython/pygeoapi/pull/2214)   
  - provides html response (to EDR query) rendered with leaflet (leaflet understands the CoverageJSON PointSeries response)   
```
{
    "type":"Coverage",
    "domain":{
        "type":"Domain",
        "domainType":"PointSeries",
        …
```
  - Example ogcapi collection from ==pygeoapi demo==: [html](https://demo.pygeoapi.io/master/collections), [json](https://demo.pygeoapi.io/master/collections?f=json)   
  - … collection with map operation: [https://demo.pygeoapi.io/master/collections/mapserver_world_map?f=json](https://demo.pygeoapi.io/master/collections/mapserver_world_map?f=json)  
  - … and map response: [https://demo.pygeoapi.io/master/collections/mapserver_world_map/map?f=png](https://demo.pygeoapi.io/master/collections/mapserver_world_map/map?f=png)

### earthkit plugin for pygeoapi  
  - Implementation: \[pygeoapi\] + \[plugin: earthkit\] + \[earthkit\] + \[polytope-service (remote)\] + \[FDB\]   
  - Implementation: \[pygeoapi\] + \[plugin: earthkit\] + \[earthkit\] + \[polytope-library (local)\] + \[xarray\] + \[Zarr\] << soon  
  - ogcapi-maps - using earthkit plots as a map provider  
    - DestinE data map-server (_currently not working_): [https://polytope-edr.lumi.apps.dte.destination-earth.eu/collections/mapserver_world_map/map?f=png&subset=instance_id(202601140000)&datetime=202601150000&parameter-name=165](https://polytope-edr.lumi.apps.dte.destination-earth.eu/collections/mapserver_world_map/map?f=png&subset=instance_id(202601140000)&datetime=202601150000&parameter-name=165)  
  - ogcapi-edr - using earthkit+polytope/xarray   
  - ogcapi-processes chart (time-series) renderer [using “earthkit workflows”? - same as Forecast Graph Builder in ECMWF’s Forecast-in-a-box]   
  - _Exposing which ECMWF datasets?_

### Indexed-GRIB-kerchunk plugin for pygeoapi  
  - Original version working with ECCC/MSC Canadian Global 15km NWP data; GRIB files indexed in-situ (on-prem at ECCC)  
  - Update(d) to work with NOAA GFS data on ASDI  
  - Implementation: \[pygeoapi\] + \[plugin: Kerchunk-indexer\] + \[xarray\] + \[kerchunk indexes\] + \[cfgrib\] + \[remote GRIB\]  
  - Indexes pointing to Amazon/ASDI-hosted GRIB datasets (NOAA GFS); index objects stored in “garage” object-store (S3 clone)  
  - Hand-crafted STAC Catalog to find model-run instances  
  - Abstracts ‘hypercube’ from how GRIB files are chunked, the indexes can be consistent even when the original GRIBs are organised differently; different approaches from each centre -  
    - UKMO: 1 file = single parameter, one time-step, all levels  
    - GFS: 1 file = all parameters, all levels, one time-step  
    - MSC: 1 file = single parameter, one time-step, one level  
    - ECMWF: 1 file = all parameters, all levels, one time-step  
  - Indexer needs to be configurable for a given data source based on file organisation, naming conventions etc. - object-naming, structure and organisation is a sovereign choice that we shouldn’t attempt to standardise  
  - Can explicitly identify the dimensions (e.g., z = abc) in the pygeoapi xarray-core config, so pygeoapi knows which label relates to x, y, z, t dimension (& ensemble)   
  - “Dataless” ogcapi server:   
    - The data doesn’t need to be colocated with the server (it’s lazy-loading the data from wherever), only the indexes need to be pre-loaded  
    - Client applications don’t need to know about the back-end GRIB objects  
    - The ogcapi server itself doesn’t even need to know about the back-end GRIB objects (only that it’s using GRIB somewhere, so needs the cfgrib library)  
  - Parameter names:  
    - Currently, the parameter names and axes from the source data propagate through to the indexes - because Kerchunk (etc.) just does a pass-through; for GRIB this isn’t too problematic because it has “opinionated” naming, netCDF is more flexible though  
    - _option_: add a mapping in the .zattrs that explains how (local) param X maps to standard param Y.  
      - parameter information _should_ be described in the `theme` concepts in the WCMP2 discovery metadata record; here, locally used parameter names are mapped back to authoritative definitions in controlled vocabularies  
      - … the WCMP2 record plus the `.zattrs` metadata should be sufficient for a power user (e.g., a data engineer) to interpret the parameter in a useful way  
      - this metadata can be used to inform upstream applications - like pygeoapi, to include this mapping as additional parameter keys (e.g., EDR collection still includes param X, but the metadata for param X would indicate a mapping to Y) (_see discussion elsewhere about shortnames_).  
      - then client applications (that know about the profile) could look up the _real_ param (e.g., X) that maps to the standard param (e.g., Y)  
  - Opinionated hypercube structure: one kerchunk index (JSON object) for 1 model run, single variable  

#### Example index create with kerchunk: \[local\] [file:///Users/jeremy/Documents/Scratch/t2m_2026011400.json](file:///Users/jeremy/Documents/Scratch/t2m_2026011400.json)  
```
{
  "version": 1,
  "refs": {
    ".zgroup": "{\"zarr_format\":2}",
    ".zattrs": "{\"GRIB_edition\":2,\"GRIB_centre\":\"cwao\",\"GRIB_centreDescription\":\"Canadian Meteorological Service - Montreal\",\"GRIB_subCentre\":0,\"institution\":\"Canadian Meteorological Service - Montreal\",\"coordinates\":\"heightAboveGround latitude longitude step time valid_time\"}",
    "valid_time/.zarray": "{\n  \"shape\": [\n    30\n  ],\n  \"chunks\": [\n    30\n  ],\n  \"dtype\": \"\u003Ci8\",\n  \"fill_value\": null,\n  \"order\": \"C\",\n  \"filters\": null,\n  \"dimension_separator\": \".\",\n  \"compressor\": null,\n  \"zarr_format\": 2\n}",
    "valid_time/.zattrs": "{\n  \"units\": \"seconds since 1970-01-01T00:00:00\",\n  \"calendar\": \"proleptic_gregorian\",\n  \"standard_name\": \"time\",\n  \"long_name\": \"time\",\n  \"_ARRAY_DIMENSIONS\": [\n    \"valid_time\"\n  ]\n}",
    "valid_time/0": "base64:gNxmaQAA...",
    "t2m/.zarray": "{\"shape\":[30,1201,2400],\"chunks\":[1,1201,2400],\"dtype\":\"\u003Cf8\",\"fill_value\":null,\"order\":\"C\",\"filters\":[{\"id\":\"grib\",\"var\":\"t2m\",\"dtype\":\"float64\"}],\"dimension_separator\":\".\",\"compressor\":null,\"zarr_format\":2}",
    "t2m/.zattrs": "{\"GRIB_paramId\":167,\"GRIB_dataType\":\"af\",\"GRIB_numberOfPoints\":2882400,\"GRIB_typeOfLevel\":\"heightAboveGround\",\"GRIB_stepUnits\":1,\"GRIB_stepType\":\"instant\",\"GRIB_gridType\":\"regular_ll\",\"GRIB_uvRelativeToGrid\":0,\"GRIB_shortName\":\"2t\",\"GRIB_units\":\"K\",\"GRIB_name\":\"2 metre temperature\",\"GRIB_cfName\":\"air_temperature\",\"GRIB_cfVarName\":\"t2m\",\"GRIB_missingValue\":3.4028234663852886e+38,\"GRIB_NV\":0,\"GRIB_gridDefinitionDescription\":\"Latitude\\/longitude\",\"GRIB_Nx\":2400,\"GRIB_iDirectionIncrementInDegrees\":0.15,\"GRIB_iScansNegatively\":0,\"GRIB_longitudeOfFirstGridPointInDegrees\":180.0,\"GRIB_longitudeOfLastGridPointInDegrees\":179.85,\"GRIB_Ny\":1201,\"GRIB_jDirectionIncrementInDegrees\":0.15,\"GRIB_jPointsAreConsecutive\":0,\"GRIB_jScansPositively\":1,\"GRIB_latitudeOfFirstGridPointInDegrees\":-90.0,\"GRIB_latitudeOfLastGridPointInDegrees\":90.0,\"long_name\":\"2 metre temperature\",\"units\":\"K\",\"standard_name\":\"air_temperature\",\"_ARRAY_DIMENSIONS\":[\"valid_time\",\"latitude\",\"longitude\"]}",
    "t2m/0.0.0": [
      "https://dd.weather.gc.ca/20260114/WXO-DD/model_gem_global/15km/grib2/lat_lon/00/000/CMC_glb_TMP_TGL_2_latlon.15x.15_2026011400_P000.grib2",
      0, 1351673],
    "heightAboveGround/.zarray": "{\"shape\":[30],\"chunks\":[1],\"dtype\":\"\u003Cf8\",\"fill_value\":null,\"order\":\"C\",\"filters\":null,\"dimension_separator\":\".\",\"compressor\":null,\"zarr_format\":2}",
    "heightAboveGround/.zattrs": "{\"units\":\"m\",\"positive\":\"up\",\"long_name\":\"height above the surface\",\"standard_name\":\"height\",\"_ARRAY_DIMENSIONS\":[\"valid_time\"]}",
    "heightAboveGround/0": "\u0000\u0000\u0000\u0000\u0000\u0000\u0000@",
    "latitude/0": "base64:AAAAAACA...",
    "latitude/.zarray": "{\"shape\":[1201],\"chunks\":[1201],\"dtype\":\"\u003Cf8\",\"fill_value\":null,\"order\":\"C\",\"filters\":null,\"dimension_separator\":\".\",\"compressor\":null,\"zarr_format\":2}",
    "latitude/.zattrs": "{\"units\":\"degrees_north\",\"standard_name\":\"latitude\",\"long_name\":\"latitude\",\"_ARRAY_DIMENSIONS\":[\"latitude\"]}",
    "longitude/0": "base64:AAAAAACA...",
    "longitude/.zarray": "{\"shape\":[2400],\"chunks\":[2400],\"dtype\":\"\u003Cf8\",\"fill_value\":null,\"order\":\"C\",\"filters\":null,\"dimension_separator\":\".\",\"compressor\":null,\"zarr_format\":2}",
    "longitude/.zattrs": "{\"units\":\"degrees_east\",\"standard_name\":\"longitude\",\"long_name\":\"longitude\",\"_ARRAY_DIMENSIONS\":[\"longitude\"]}",
    "step/.zarray": "{\"shape\":[30],\"chunks\":[1],\"dtype\":\"\u003Ci8\",\"fill_value\":null,\"order\":\"C\",\"filters\":null,\"dimension_separator\":\".\",\"compressor\":null,\"zarr_format\":2}",
    "step/.zattrs": "{\"units\":\"hours\",\"standard_name\":\"forecast_period\",\"long_name\":\"time since forecast_reference_time\",\"dtype\":\"timedelta64[ns]\",\"_ARRAY_DIMENSIONS\":[\"valid_time\"]}",
    "step/0": "\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000",
    "time/.zarray": "{\"shape\":[30],\"chunks\":[1],\"dtype\":\"\u003Ci8\",\"fill_value\":null,\"order\":\"C\",\"filters\":null,\"dimension_separator\":\".\",\"compressor\":null,\"zarr_format\":2}",
    "time/.zattrs": "{\"units\":\"seconds since 1970-01-01T00:00:00\",\"calendar\":\"proleptic_gregorian\",\"standard_name\":\"forecast_reference_time\",\"long_name\":\"initial time of forecast\",\"_ARRAY_DIMENSIONS\":[\"valid_time\"]}",
    "time/0": "base64:gNxmaQAAAAA=",
    "t2m/1.0.0": [
      "https://dd.weather.gc.ca/20260114/WXO-DD/model_gem_global/15km/grib2/lat_lon/00/003/CMC_glb_TMP_TGL_2_latlon.15x.15_2026011400_P003.grib2",
      0, 1021508],
    "heightAboveGround/1": "\u0000\u0000\u0000\u0000\u0000\u0000\u0000@",
    "step/1": "\u0003\u0000\u0000\u0000\u0000\u0000\u0000\u0000",
    "time/1": "base64:gNxmaQAAAAA=",
    ...
    "t2m/29.0.0": [
      "https://dd.weather.gc.ca/20260114/WXO-DD/model_gem_global/15km/grib2/lat_lon/00/087/CMC_glb_TMP_TGL_2_latlon.15x.15_2026011400_P087.grib2",
      0, 1280654],
    "heightAboveGround/29": "\u0000\u0000\u0000\u0000\u0000\u0000\u0000@",
    "step/29": "W\u0000\u0000\u0000\u0000\u0000\u0000\u0000",
    "time/29": "base64:gNxmaQAAAAA="
  }
}
```

#### Example usage in xarray:  
```
>>> import xarray as xr
>>> d = xr.open_dataset('/{path}/t2m_2026011400.json', engine='kerchunk').squeeze()
>>> d
<xarray.Dataset> Size: 692MB
Dimensions:            (valid_time: 30, latitude: 1201, longitude: 2400)
Coordinates:
  * valid_time         (valid_time) datetime64[ns] 240B 2026-01-14 ... 2026-0...
  * latitude           (latitude) float64 10kB -90.0 -89.85 -89.7 ... 89.85 90.0
  * longitude          (longitude) float64 19kB -180.0 -179.8 ... 179.7 179.8
    heightAboveGround  (valid_time) float64 240B ...
    step               (valid_time) timedelta64[ns] 240B ...
    time               (valid_time) datetime64[ns] 240B ...
Data variables:
    t2m                (valid_time, latitude, longitude) float64 692MB ...
Attributes:
    GRIB_edition:            2
    GRIB_centre:             cwao
    GRIB_centreDescription:  Canadian Meteorological Service - Montreal
    GRIB_subCentre:          0
    institution:             Canadian Meteorological Service - Montreal
>>> d. sel(latitude=45.0, longitude=-75.0, method='nearest').t2m.values
array([277.17514648, 276.23178711, 277.2572998 , 277.56884766,
       277.11738281, 277.50239258, 277.80026855, 277.39003906,
       275.70522461, 274.75771484, 271.4262207 , 270.07958984,
       268.74667969, 267.91367188, 267.01542969, 265.5671875 ,
       263.97358398, 263.02460938, 262.24772949, 261.5934082 ,
       260.98117676, 260.40864258, 261.83337402, 262.27705078,
       258.79003906, 259.26916504, 260.7435791 , 262.09145508,
       263.4121582 , 267.79743652])

```

- note on xarray operations:  
  - `open_dataset()` is the “Swiss Army knife” - but you need to tell it which dataset engine to use  
  - `open_zarr()` is a convenience wrapper that only works with Zarr  
  - `open-mf()` opens multiple files and combines them into a single (in-memory) virtual datacube  

### Native-Zarr plugin for pygeoapi  
- Implementation: [pygeoapi] + [plugin: MSC-Zarr] + [xarray] + [local Zarr stores]   
- Identify Zarr objects with structured filename (based on ECCC/MSC data organisation and naming conventions)  
- Reads directory (bucket) with Zarr objects to identify the instances (model-runs); instance ID is hard-coded pattern matching on the object name (store-specific configuration must be adaptable to sovereign choices)  
- Deploy pygeoapi services over local data; Zarr objects stored in “garage” object-store (S3 clone)   

### And ...    
- GRIB hypercube indexer using Kerchunk   
- Dummy WIS2 Global Discovery Catalogue (so we can add WCMP2 records pertinent to the demo): [https://example.com/collections/wis2-discovery-metadata](https://example.com/collections/wis2-discovery-metadata) (note: WCMP2 metadata editor is out of scope)  
- WebUI:   
  - [https://wmo-im.github.io/metdx-demo/metdx-demo-ui/index.html](https://wmo-im.github.io/metdx-demo/metdx-demo-ui/index.html) (_simple search (text box only), search results, and record summary pages only_) - deployed from GitHub Pages, no servers required  
    - Developed using [Flet](https://flet.dev) (“easily build realtime web, mobile and desktop apps in pure Python”) (_and MatPlotLib?_)  
    - \[configuration\] point to a WIS2 Global Discovery Catalogue to discover data service end-points  
    - \[configuration\] point to locally deployed data service end-points and STAC Catalogs (these have not been published to WIS2)  
    - \[discovery\] simple text search to find resources (_combine search across GDC and local resources?_)  
    - \[interactive\] data-driven “add-to” application workflows for adding data and maps to the application   
  
# Next (short-term):  
- Document “opinionated” hypercube pattern for weather prediction model output  
- “Dockerise” GRIB indexer  
- (AW) Automate addition of model run instances to ECMWF (_earthkit?_) plugin in for pygeoapi so that new data arriving is visible in the Collection, Instance, and Schema end-points (overriding default configuration)  
- (AW) OGC-API Maps temporal extent for ECMWF (_earthkit?_) plugin in for pygeoapi   
  
# Next (medium term):  
- Continue to develop the Web-based UI  
    - “Add to” application workflows for maps (imagery), coverages (q=polygon, q=radius, q=location), and time-series (q=radius, q=location)  
    - Diff two data-sources?  
- Implement pygeoapi plugin for ECMWF’s Polytope-over-xarray providing all types of EDR data queries (“if you can open a resource in xarray, you can run polytope with it”)  
- (MB) Configure GRIB indexer to use GFS data on ASDI (retention period is 2+ years so the indexes will still be valid in Nov-2026 for INFCOM-4)  
- (MB) Convert the GRIB indexer to a ogcapi-processes (part 1) deployment with asynchronous execution  
- Update GRIB indexer to use VirtualiZarr/Zarr.v3   
- Develop proposal on describing model-run output with STAC   
- Enumerate data visualisation use cases in [opengeospatial/metocean-ogcapi-maps-profile ](https://github.com/opengeospatial/metocean-ogcapi-maps-profile)(as per Issue #1)  
- (MB) Implementation: \[lambda: custom ogcapi-edr implementation\] + \[xarray\] + \[indexes\] + \[remote open data\]  
- (MB) Implementation: \[lambda: custom ogcapi-maps implementation + matplotlib\] + \[xarray\] + \[indexes\] + \[remote open data\]  
    - AWS lambda implementations (which incur running costs) could be deployed by 3rd parties, thus enabling sharing of the costs associated with data sharing  
- (JT/Mikko Rauhala (FMI)) Implementation: \[FMI SmartMet Server\]  
    - Illustrate another implementation of ogcapi-maps/ogcapi-edr MetOcean profiles for weather prediction model output data  
    - [https://github.com/fmidev/smartmet-server](https://github.com/fmidev/smartmet-server)   
- Fund implementation of MetOcean profile for ogcapi-maps in QGIS  
  
# For INFCOM-4:  
- Recommendations:  
  - Develop [community standard?] for hypercube description  
        - Begin with weather prediction model output  
        - Define roadmap for other applications (climate models, radar, satellite)  
        - Engage/harmonize with OGC GeoZarr Standards Working Group  
  - Develop standardised framework in TechRegs for use of Web-based data APIs in WIS2: increasing “prescriptiveness” to drive machine-level interoperability at domain level  
        1. APIs must be self-describing 
        2. Use OpenAI to describe your API  
        3. Use an open-standard API definition if a suitable one exists for your application (see no. 4)  
        4. Use OGC-APIs: Maps for visualisation, EDR for data, Processes (part 1) for in-situ data-processing  
        5. Use specific OGC-API profile where one has been defined for your domain (e.g., NWP)    
  - Collaboration to develop OGC MetOcean profile for OGC-API Maps  
  - Collaboration to develop OGC MetOcean profile for OGC-API EDR  
  - Collaboration to develop OGC MetOcean profile for OGC-API Processes (part 1) for in-situ calculation on datasets  
- Draft standard  definition for hypercubes of weather prediction model output   
- Well considered drafts of profiles for OGC-API Maps and EDR for weather prediction model output _to illustrate how a profile should be defined_ 
    - OGC-API Maps profile: GitHub Repo [opengeospatial/metocean-ogcapi-maps-profile](https://github.com/opengeospatial/metocean-ogcapi-maps-profile), Draft spec: [https://docs.ogc.org/DRAFTS/26-002.html](https://docs.ogc.org/DRAFTS/26-002.html)   
- Recommended set of processes that weather prediction modelling centre may offer for processing their data in-situ _based on recommendations from NOAA_  
- Deployable software on GitHub (Docker Compose deployment)  
```
docker compose up 
```
  - pygeoapi reference implementation  
  - WebUI  
- Deployed software stack plus data for target user in Belize  
    - Hypercube index over NOAA’s GFS data on ASDI _with manually hacked STAC catalog?_
    - ECCC/MSC forecast model output as local (native) Zarr  hypercubes _using filename pattern matching on MSC’s WAF to determine content of each hypercube_
- Feedback from Dwayne Scott (Belize) as representative user  
- Live demo in booth  
- Video of demo  
  
# Pathways from POC to operations:  
- ECMWF DestinE: improve data sharing and interoperability through open standards  
- ECMWF Copernicus: TBD - but intent to provide better tooling to use the ERA5 dataset  
- OGC-API EDR and Maps profiles: “add to” work-flows for GeoWeb/DataExplorer (starting from discovery with WCMP2 records in WIS2 GDC)  
