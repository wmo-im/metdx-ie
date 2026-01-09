# SG-FIT: Least Disruptive
## Four-point plan:
Clarify what recommendations we want to make to INFCOM. Maybe this is changes to Tech Regs (Manual, Guide), new "good practice" ("MAY" rather than "SHOULD"), proposal to invest in developing open source solutions, or to extend pilots to additional domains.
Agree what examples we will use to support those recommendations.
Determine what documentation we'll provide to INFCOM-4 - either in the main paper or as an appendix.
Create a plan for the deliverables.

## Pre-requisites for data sharing within WMO:
Data publishers should implement WIS2:
Manual on the WMO Information System (WMO-No. 1060), Volume II – WIS 2.0
PART IV. WIS TECHNICAL SPECIFICATIONS
WIS-TECHSPEC-1: MANAGING DISCOVERY METADATA
WIS-TECHSPEC-2: PUBLISHING DATA AND DISCOVERY METADATA
Namely:
Data and metadata are accessible using open protocols (HTTPS).
Encode data in open, community-agreed formats (e.g., for WMO World Weather Watch use GRIB2 and BUFR).
Provide good discovery metadata (WMO Core Metadata Profile v2, WCMP2).
Provide pub/sub notifications (MQTT) about the availability of new or updated data and metadata (WIS2 Notification Message, WNM).
Broadly speaking, (A), (B) and (C) combined with the WIS2 Global Services and the WMO Codes Registry go much of the way to delivering the FAIR Data Principles.
(D) is essential to inform users/agents (humans and software systems) in near-real-time at scale when (meta)data becomes available. The notifications contain sufficient metadata for the user/agent to determine whether they should access the data at the URL provided in the notification message.
(In this work the non-technical foundations for data sharing, such as licensing, are out of scope.)
## Patterns for high-performance data sharing:
Data publishers should ensure that users/agents can process data in proximity to where it is stored. Or said differently: Don’t move all the data.
“Processing” data can take many forms – from computing pre-defined statistics, to retrieving a sub-set, to rendering data as an image, or bespoke computing such as model training. Large data volumes are difficult, if not impossible, to move in a timely and cost-effective manner. [This is often referred to as “data gravity”.] Even if one could move large volume data, many users lack the necessary local capacity and capability to manage a copy of the data. And finally, it’s much harder to confirm the authenticity and veracity of datasets when working with copies (e.g., they may have been subject to local modification).
We offer proposals to support a range of users’ data processing needs:
For bespoke computation: A. Data access optimization for power users
For Web applications retrieving data or rendering imagery of data: B. Data access optimization for everyday users
For pre-defined computations: C. Custom data processing
Proposals (B) and (C) may be implemented by themselves but will often benefit from being layered over proposal (A).
We show how these proposals can be leveraged by low- and middle-income countries with open-source software stacks: D. Toolkits for local NMHS operations

## A. Data access optimization for power users
### Patterns:
[How do we present these patterns as “good practice” that Members should follow?]
Provide proximate compute:
Without going into details, data read speed is affected by network bandwidth and geographic distance. Processing data close to where it is stored will improve read speeds.
Data processing consumes computation resources (“compute”). Provision of compute to users requires cloud characteristics: on-demand self-service and rapid capacity elasticity.
Provision of compute capacity is not free. Someone always pays: Either through up-front infrastructure investment in private or community cloud, or pay-as-you-go pricing in public cloud.
Where compute capacity is constrained (either through infrastructure or financial limitations) data publishers must manage the impacts of resource limitations to maintain sufficient quality of service (QoS). For example, employing demand-based throttling (i.e., limiting the number of concurrent requests).
High-performance data sharing depends on a cloud foundation. Users’ query performance expectations should be tempered with clear signposting of any [QoS limits].
[more to say on QoS and managing users’ expectations?]
Object stores not filesystems:
Object stores are a superior choice for managing data at unlimited scale. [reasoning: support for massively parallel querying, near-infinite scaling, distributed storage across multiple nodes or servers, cost-effectiveness, etc.]
[Access with HTTP API]
[Should use of object-stores for publishing high-volume data be a recommendation to INFCOM? It doesn’t necessarily imply cloud implementation.] << recommendation/good practice … what requirement does this support? DestinE uses block-storage under the hood; stable API at higher level as a contract with the users. “API-first” approach … let implementors make the right choice for them.
Cloud-optimize your data:
Cloud-optimized data formats are designed to get the most out of object stores:
Cloud-native data is stored as small addressable chunks (files, tiles, or both) which enables data reads to be parallelised into many small requests. With traditional formats chunks are often exposed to users as individual files. Cloud-native formats can present entire datasets (e.g., a weather prediction model run) as a single resource with the chunking hidden from users.
Traditional formats pack metadata into the headers of each file. With datasets often split into hundreds or thousands of files, figuring out what data is encoded in each file requires many reads – fine for local storage, but slow for object-stores. A cloud-native format packages the metadata for the dataset (i.e., what data is in each chunk) so that it can be accessed with a single read.
Cloud-native dataset metadata can be used by applications to construct a virtual dataset in memory, only the loading bytes of data needed at computation time (aka “lazy loading”) with reads from different chunks being done concurrently. This is often supported natively by data analysis tools and libraries.
Cloud-native formats should also support [HTTP] range-requests – further reducing the number of bytes that need to be read by the application.
Cloud-optimization doesn’t necessarily mean abandoning traditional formats. Libraries such as VirtualiZarr and Kerchunk use indexes over chunked, compressed data formats (e.g., GRIB2 and NetCDF/HDF5) to expose collections of files as cloud-optimized Zarr stores that support byte-range access and lazy-loading.
[Users developing AI models will revisit the data many times in order train the model. For training performance, users will often reshape the data (i.e., adapt the chunking and data organisation) to optimise for the training. How should we address this kind of access pattern? GPUs are expensive, you don’t want them to be idle waiting for data. But – these patterns can be used to create ML-ready optimised datasets for model training] << Miruna says this is very important! Trade-off download and local management for performance in training
### Recommendations to INFCOM:
New work item (SC-IMT): data hypercube description format
Weather and climate simulations (including forward projections and reanalyses) are generated by many Members and their outputs are the most widely used high-volume datasets used by the WMO community [evidence?]. We recommend prioritizing the standardization of cloud-optimized provision of weather and climate simulation outputs. << priority is considered fair by Miruna
For convenience, we refer to the multidimensional data arrays (a.k.a. N-dimensional arrays, ND-arrays, “tensors”) produced by weather and climate simulations as “data hypercubes”.
The Zarr Storage Specification defines an open-source provides a mechanism for describing data hypercubes. GeoZarr aims to provide geospatial extensions that are needed to describe Earth-system data. Zarr is one example. STAC is also in the mix (one model run = several hypercubes).
This work item would establish a common vocabulary for describing data hypercubes generated by weather and climate simulations [including the controlled vocabulary for physical parameter names that is harmonized with the MetOcean EDR profile – ET-Data mapping exercise between CF and GRIB … “a lot of work” but this harmonization exercise needs to be done so that we can describe physical parameters in a consistent way].
[Recommendation to INFCOM for SC-IMT in collaboration with OGC (GeoZarr, DataCube) through the WMO-OGC MoU to define a standard mechanism to define the hypercube, and the vocabulary (over the next intergovernmental period).]
[Include note on how WMO would manage the relationships with community standards (e.g., snapshot, publicly hosted copy at WMO, etc.). Particularly relevant for lifecycle / change management of STAC extensions.]
[common interface to interact with a hypercube (describing parameters, dimensions, etc.), abstraction layer between files (objects) and the user, not a common implementation.]
[What does this look like in 5 years? Data publishers provide 000s of GRIB files _plus_ a hypercube definition layered over the GRIB.] << this is how it remains “least disruptive”.
Include a note on publishing WIS2 Notification Messages for the hypercube – rather than for all the individual files (or alongside)? [ET-Metadata and TT-NWP-metadata]
[Include a roadmap for the cloud-optimized provision of other datatypes (e.g., Parquet for vector/tabular data; satellite, radar, …)? Identify what should be next (convergence of user benefit and technology readiness).]
[Exclude details on chunking strategies – there are too many options! Identify chunking as a consideration for how the data is organised: think about your query pattern(s). ECMWF resolve query patterns across different services for high performance – requiring data to be replication albeit organised in different ways.] << important to exclude the details – but do get people to think about the implications of how the data is structured.
### Example implementations:
ECMWF FDB and Polytope
GRIB + [open data] indexes; GRIBJump
APIs deconflict resource contention.
Polytope: not a standard API; bespoke definition of a hypercube; but it’s an implementation that delivers on the pattern; user doesn’t interact with the files / objects all.

Implementation example: ECMWF FDB + Polytope + (earthkit SDK)
"Polytope: Feature extraction for datacubes".
High-performance implementation: A timeseries request for 1 location from a 50-member ensemble returns a 10KB extract from an 80TB dataset in 10-seconds.
Polytope _service_ only available to ECMWF Members (limited users; bounded resource); it leverages ECMWF's operational platforms (HPC, MARS).
FDB is a custom key-value object store at ECMWF; objects are GRIB fields.
FDB requests use the MARS language (i.e., the data card “MARS Request”); FDB finds the data-atoms (i.e., fields) that are needed using a catalogue and indexes.
FDB identifies where data is persisted and returns a list of data-atoms with their byte-locations.
Polytope presents a higher-level API, layered on top of FDB object stores (or XArray data sources).
Polytope provides a python-SDK (which is further simplified in ECMWF's earthkit) and an HTTP API.
Polytope requests use MARS language plus additional temporal-geometric components to enable feature-subsetting.
Polytope uses GRIBjump to determine byte-ranges within the GRIB-encoded data-atoms for the "convex polytopes" (i.e., boxes, disks) requested by the user; byte ranges within GRIB fields are predictable. Also works on compressed fields because you can predict where the content is and decompress local sections.
Polytope then extracts the data from the byte-ranges and compiles the resulting data into arrays.
Polytope arrays are serialized as CovJSON - but users never need to work directly with those objects as they can be read into objects (e.g., xarray) for convenience.
Note: Polytope doesn't cache responses - analysis suggests anything cached is always trashed before being re-read.
Implementation:
https://github.com/ecmwf/polytope
https://github.com/ecmwf/earthkit

Met Service Canada Virtual Optimal Forecast (VOF) over Zarr
Public beta for Zarr in Canada - in Feb 2026 … OpenShift (on prem cloud) … limited cluster resources … need to manage those resources
… Zarr (or Icechunk)
… pygeoAPI talking to xarray via Zarr or Icechunk
… EDR API = pygeoAPI
… EDR collection to find the appropriate “model run [set]” - rather than STAC catalog
… abstraction from the underlying files / objects
… the Zarr end-point will remain private for now
… the EDR endpoint will probably be locked down so only the public Website can make requests - all to drive the visualisations on ECCC public website
… Virtual optimal forecast (VOF) stitching together now cast (0-6h) + deterministic hi-res (6-48h) + global (48h+)
… drive the public forecast

STAC and VirtualiZarr
[WCMP2 ->] STAC + Zarr + native
[deep-dive session (all the way to Curl requests and payload responses ) requested to resolve debate about how STAC assets can be efficiently organised to avoid handling 000s]
Find the STAC catalog by searching the WMO Global Discovery Catalogue
STAC provides the index on sets of hypercubes – e.g., an archive of model runs.
Zarr provides the index on the hypercube.
[Zarr store = “cube-let” … so STAC helps us find the cube-let (an Asset at a URL), then Zarr store to request the bytes that are needed.]
Equitable cost sharing on public cloud: Data publishers provide the data and object store; users run the computation in their own tenancies.

<< Miruna can share STAC catalog references for DestinE satellite content.

Implementation example: STAC + Zarr + native GRIB / netCDF
Highly scaleable implementation: "no moving parts" required to provide data access.
Because STAC catalogues, Zarr stores and data files can be hosted without complex servers or infrastructure, hosting data can be as simple as dropping static data files and metadata into a web-accessible folder.
Spatio-Temporal Asset Catalogue (STAC).
[…]
Zarr + indexing.
[…]
Implementation:
STAC Catalogue for an accumulating archive of UKMO NWP data; native CF-netCDF and VirtualliZarr indexing (proof of concept).
STAC Catalogue for an accumulating archive of ECMWF ensemble data; native GRIB and Kerchunk indexing (proof of concept).
ECMWF STAC Catalogue for DestinE, including STAC Extension for better (metadata) descriptions of data holdings << Tiago - to confirm [yes, but it does use custom extensions to STAC, Mark also has needed to create new extensions]
Web-based Data Catalogue leveraging STAC APIs (proof of concept).
Web-based Data Catalogue leveraging STAC APIs (built using Claude LLM).
Python notebooks to retrieve, process and render data exposed through the STAC Catalogue and indexed Zarr.
HTTP APIs (OGC WMS, OGC-API EDR) to demonstrate serverless user-facing applications for data retrieval and data visualisation (proof of concept).

ECMWF ZFDB (to be included ???)
Downloadable FDB implementation for DestinE - including 3-4 PB data for a handful of climate scenarios (30-year scenario)
… a virtualZarr over FDB with virtual chunking
… ZFDB “Zarr over FDB” - currently prototype
… for local usage (not over HTTP)
… xarray.load uses FDB native protocol (not HTTP range requests [yet]) - so it only works inside a data centre
Many users like Zarr as an interface.
So they want Zarr as data
Virtual chunking allows for rapid “refactoring” re-shaping for their model training.
Exploring the data (20-30TB) before they commit to refactoring petabytes into the right shape.
## B. Data access optimization for everyday users
[How much data is too much data? Web applications are limited by local memory footprint. Need to optimize the query so that that response payload is a manageable size.]
[APIs can deconflict resource contention for direct file access (e.g., Zarr)]
[Highly unlikely that anyone with train an ML model on an EDR interface]
### Patterns:
Avoid [batch] pre-computing images or data products that may not be needed. Generate them on request from users. Cache if you can.
HTTP API
… for earth observation / earth prediction data (including discrete sampling from continuous datasets): OGC-API EDR
… visualization (pictures of data): OGC-API Maps
Standards are pretty flexible - restrictive profiles needed to ensure common provision.
Encodings for Web applications: JSON-based.
[Vocabulary services? TBD]
### Recommendations to INFCOM:
New work item: Incorporate OGC-API Maps into WIS2 Tech Regs + MetOcean profile
… to provide a service that renders data into images for use in Web applications.
… anticipate OGC-API Maps MetOcean profile being a OGC MetOcean work item circa Nov 2026; engage with this activity to drive towards suitable outcomes; leverage WMO-OGC MoU - develop the profile in OGC, don’t start a WMO task team
New work item: Incorporate OGC-API EDR into WIS2 Tech Regs + MetOcean profile
… to provide a service that extracts subsets of “coverage” data (expressed in user-centric terms, queried as "polytopes") for use in Web applications.
[at this stage, we won’t recommend OGC-API Features because we expect EDR queries to better match user expectations (e.g., bounding box). EDR provides an item query api for named objects]
… build on EUMETNET MeteoGate EDR profile (intended to become an OGC MetOcean profile).
New work item: Guidance on encoding data in Web-friendly formats – CoverageJSON, GeoJSON, JSON-FG.
… CoverageJSON: see the MeteoGate EDR profile
… GeoJSON - adding metadata properties? Collection level, item level. Have done this for observations in WIS2Box. [Tom to draft a good practice]
… need to agree a schema for GeoJSON when used as an EDR payload
… Technical recommendations on compression; which algorithms? [James]
… possible further extensions for data like time-series, parameter names << work item - collaborate with OGC on this ???
New work item: SC-IMT engage with OGC-API Vocabulary SWG via the WMO-OGC MoU – ensure the specification meets needs of WMO. [Tom]
### Example implementations:
ECMWF's polytope/EDR work, Tom's "dataless" pygeoAPI plugin, and Mark's serverless implementations

PygeoAPI
Reference implementation
Polytope [yes - done] ECMWF* … earthKit plots as a “Map provider” into pygeoAPI Maps [to do] Adam
x-array (via Zarr or Icechunk) for NWP data [yes - public beta in Feb] Canada
Serverless lambda implementations for EDR [yes] and WMS Maps ** [to do] Mark

Implementation:
pygeoAPI backend plugin for STAC + Zarr + indexes (proof of concept). << by Dec-2025?
pygeoAPI backend plugin for ECMWF polytope. << Tiago - to confirm.
Serverless OGC WMS (Lambda on AWS) as proof-of-concept for serverless visualisation of bulk-volume data.
Serverless OGC-API EDR (Lambda on AWS) as proof-of-concept for serverless subsetting retrieval from bulk-volume data.
"dataless OGC-API" pygeoapi plugin interacting with remote Zarr (or Icechunk) datacube. << Tom to complete the "dressing" :)
pygeoAPI: containerised OGC WMS and OGC-API EDR implementation.
MSC/ECCC (Canada) OGC-API EDR implementation for NWP data. << Tom - I think this is what you mentioned in your email on 26-Aug

## C. Custom data processing (proximate compute)
Recommend use of OGC-API Processes Part 1 for data publishers (service providers?) to offer a menu of things that users can ask them to execute on the data in-situ.
>> cost implications for data processing on behalf of a client
>> our recommendations need to be clear that _SOMEONE ALWAYS PAYS_

We need a list of things that would be sensible for, say, an NWP centre to offer. I think NOAA mooted providing such a list.
Mention (or not?) using OGC-API pub/sub to push notifications as a call back? Tom has developed a pattern for the WIS2 Global Replay service which does this. I don't know how mature this approach is - maybe we just talk about this as a "consideration" rather than a good practice?

<< Miruna suggests an alternative chaining microservices together; processing near the data, but a different philosophy. EU-EPCOA ? << Tom says this is implementing OGC-API Processes part 2.

OGC-API Processes part 1
standard [done]
PygeoAPI demo [done]
Suggested MetOcean functions for NWP centres to offer [Tom]; [develop a OGC-API Processes Part 1 profile for MetOcean?]
Pub/sub call back << Global Discovery Catalogue implementation [Tom]

Server side data processing costs to managed!!!! Resource utilization

OGC-API Process part 2
Highlight it exists - at least so the don’t think we forgot
Identify challenges around security and resource utilization
Allow best practice to emerge


## D. Toolkits for local NMHS operations

Recommendations are:
"follow the WIS2Box pattern". A containerised, open-source (reference?) implementation that resource-constrained Members can deploy, enabling them to leverage the recommendations in themes (B) and (C).
Endorse WMO establishing and maintaining an open-source project to do this.

As input we develop an MVP to show what can be done. Working code trumps everything 🙂

Scope of MVP? “metdatauser-demo”
docker compose based stack
User interface - vue.js or react
“Local” API or remote API hypercube access
Minio as storage service
Don’t need Grafana etc.
Front with nginx
Subscribe to WIS2 notifications to be aware of new data

Tom happy to scaffold this.
W/c 12-Jan … hackathon for at least a couple of days that work; scaffold in place and we fill in the gaps
Completing work by April 2026.
Who should we ask to deploy and test?
… Dwayne / Belize << target Dwayne for Feb
… India (end-Jan)
… WIS2 Champions <<
… get it deployed and get feedback!

Recommendation > INFCOM should continue to invest in this MVP because it provides utility for Members … use OpenWIS Association -> WIS2 trust fund
ET-Open Source? This MVP work should follow the forthcoming WMO FOSS guide.



