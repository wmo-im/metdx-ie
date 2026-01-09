# SG-FIT: Least Disruptive

## Four-point plan

1.  **Clarify recommendations for INFCOM**
    *   Changes to Tech Regs (Manual, Guide)
    *   New "good practice" ("MAY" rather than "SHOULD")
    *   Proposal to invest in open source solutions
    *   Extend pilots to additional domains
2.  **Agree on supporting examples**
3.  **Determine documentation for INFCOM-4**
    *   Main paper or appendix
4.  **Create a plan for deliverables**

***

## Pre-requisites for data sharing within WMO

Data publishers should implement WIS2:  
[Manual on the WMO Information System (WMO-No. 1060), Volume II – WIS 2.0](https://library.wmo.int/idurl/4/68731)

**PART IV. WIS TECHNICAL SPECIFICATIONS**

*   WIS-TECHSPEC-1: MANAGING DISCOVERY METADATA
*   WIS-TECHSPEC-2: PUBLISHING DATA AND DISCOVERY METADATA

**Namely:**

*   Data and metadata are accessible using open protocols (HTTPS).
*   Encode data in open, community-agreed formats (e.g., for WMO World Weather Watch use GRIB2 and BUFR).
*   Provide good discovery metadata (WMO Core Metadata Profile v2, WCMP2).
*   Provide pub/sub notifications (MQTT) about the availability of new or updated data and metadata (WIS2 Notification Message, WNM).

> Broadly speaking, (A), (B) and (C) combined with the WIS2 Global Services and the [WMO Codes Registry](https://codes.wmo.int/) go much of the way to delivering the [FAIR Data Principles](https://www.go-fair.org/fair-principles/).  
> (D) is essential to inform users/agents (humans and software systems) in near-real-time at scale when (meta)data becomes available.

*In this work the non-technical foundations for data sharing, such as licensing, are out of scope.*

***

## Patterns for high-performance data sharing

Data publishers should ensure that **users/agents can process data in proximity to where it is stored**.  
Or said differently: Don’t move all the data.

> “Processing” data can take many forms – from computing pre-defined statistics, to retrieving a sub-set, to rendering data as an image, or bespoke computing such as model training. Large data volumes are difficult, if not impossible, to move in a timely and cost-effective manner. \[This is often referred to as “data gravity”.] Even if one could move large volume data, many users lack the necessary local capacity and capability to manage a copy of the data[^1]. And finally, it’s much harder to confirm the authenticity and veracity of datasets when working with copies (e.g., they may have been subject to local modification).

We offer proposals to support a range of users’ data processing needs:

*   For bespoke computation: **A. Data access optimization for power users**
*   For Web applications retrieving data or rendering imagery of data: **B. Data access optimization for everyday users**
*   For pre-defined computations: **C. Custom data processing**

Proposals (B) and (C) may be implemented by themselves but will often benefit from being layered over proposal (A).

We show how these proposals can be leveraged by low- and middle-income countries with open-source software stacks: **D. Toolkits for local NMHS operations**

***

## A. Data access optimization for power users

### Patterns

> *How do we present these patterns as “good practice” that Members should follow?*

#### Provide proximate compute

*   Data read speed is affected by network bandwidth and geographic distance.
*   Processing data close to where it is stored will improve read speeds.
*   Provision of compute to users requires cloud characteristics: on-demand self-service[^2] and rapid capacity elasticity[^3].
*   Provision of compute capacity is not free. Someone always pays: Either through up-front infrastructure investment in private or community cloud, or pay-as-you-go pricing in public cloud.
*   Where compute capacity is constrained, data publishers must manage the impacts of resource limitations[^4] to maintain sufficient quality of service (QoS). For example, employing demand-based throttling (i.e., limiting the number of concurrent requests).
*   High-performance data sharing depends on a cloud foundation. Users’ query performance expectations should be tempered with clear signposting of any \[QoS limits].

> *more to say on QoS and managing users’ expectations?*

#### Object stores not filesystems

*   Object stores are a superior choice for managing data at unlimited scale[^5].
*   \[Access with HTTP API]
*   *Should use of object-stores for publishing high-volume data be a recommendation to INFCOM? It doesn’t necessarily imply cloud implementation.*

#### Cloud-optimize your data

Cloud-optimized data formats are designed to get the most out of object stores:

*   Cloud-native data is stored as small addressable chunks (files, tiles, or both) which enables data reads to be parallelised into many small requests.
*   Traditional formats pack metadata into the headers of each file. With datasets often split into hundreds or thousands of files, figuring out what data is encoded in each file requires many reads – fine for local storage, but slow for object-stores. A cloud-native format packages the metadata for the dataset so that it can be accessed with a single read.
*   Cloud-native dataset metadata can be used by applications to construct a virtual dataset in memory, only loading bytes of data needed at computation time (“lazy loading”) with reads from different chunks being done concurrently.
*   Cloud-native formats should also support \[HTTP] range-requests – further reducing the number of bytes that need to be read by the application.

Cloud-optimization doesn’t necessarily mean abandoning traditional formats. Libraries such as [VirtualiZarr](https://virtualizarr.readthedocs.io/en/stable/index.html) and [Kerchunk](https://fsspec.github.io/kerchunk/) use indexes over chunked, compressed data formats (e.g., GRIB2 and NetCDF/HDF5) to expose collections of files as cloud-optimized [Zarr](https://zarr.dev/) stores that support byte-range access and lazy-loading.

> *Users developing AI models will revisit the data many times in order train the model. For training performance, users will often reshape the data (i.e., adapt the chunking and data organisation) to optimise for the training. How should we address this kind of access pattern? GPUs are expensive, you don’t want them to be idle waiting for data. But – these patterns can be used to create ML-ready optimised datasets for model training.*

#### Recommendations to INFCOM

> Standards, Profiles or Best practices … TBD

#### New work item (SC-IMT): data hypercube description format

*   Weather and climate simulations (including forward projections and reanalyses) are generated by many Members and their outputs are the most widely used high-volume datasets used by the WMO community.
*   We recommend prioritizing the standardization of cloud-optimized provision of weather and climate simulation outputs.
*   For convenience, we refer to the multidimensional data arrays (a.k.a. N-dimensional arrays, ND-arrays, “tensors”) produced by weather and climate simulations as “data hypercubes”.

The [Zarr Storage Specification](https://www.ogc.org/standards/zarr-storage-specification/) defines an open-source mechanism for describing data hypercubes. [GeoZarr](https://github.com/zarr-developers/geozarr-spec) aims to provide geospatial extensions needed to describe Earth-system data. Zarr is one example. STAC is also in the mix (one model run = several hypercubes).

This work item would establish a common vocabulary for describing data hypercubes generated by weather and climate simulations (including the controlled vocabulary for physical parameter names that is harmonized with the `MetOcean EDR profile`).

> Recommendation to INFCOM for SC-IMT in collaboration with OGC (GeoZarr, DataCube) through the WMO-OGC MoU to define a standard mechanism to define the hypercube, and the vocabulary (over the next intergovernmental period).

***

### Example implementations

#### ECMWF FDB and Polytope

*   <https://github.com/ecmwf/polytope>
*   <https://github.com/ecmwf/earthkit>

#### Met Service Canada Virtual Optimal Forecast (VOF) over Zarr

*   Public beta for Zarr in Canada – in Feb 2026 … OpenShift (on prem cloud) … limited cluster resources … need to manage those resources
*   … Zarr (or Icechunk)
*   … pygeoAPI talking to xarray via Zarr or Icechunk
*   … EDR API = pygeoAPI

#### STAC and VirtualiZarr

*   STAC provides the index on sets of hypercubes – e.g., an archive of model runs.
*   Zarr provides the index on the hypercube.
*   Implementation example: STAC + Zarr + native GRIB / netCDF

#### ECMWF ZFDB (to be included ???)

*   Downloadable FDB implementation for DestinE – including 3-4 PB data for a handful of climate scenarios (30-year scenario)
*   … a virtualZarr over FDB with virtual chunking

***

## B. Data access optimization for everyday users

> How much data is too much data? Web applications are limited by local memory footprint. Need to optimize the query so that that response payload is a manageable size.

### Patterns

*   Avoid \[batch] pre-computing images or data products that may not be needed. Generate them on request from users. Cache if you can.
*   HTTP API
    *   For earth observation / earth prediction data (including discrete sampling from continuous datasets): OGC-API EDR
    *   Visualization (pictures of data): OGC-API Maps
*   Standards are pretty flexible – **restrictive profiles** needed to ensure common provision.
*   Encodings for Web applications: JSON-based.

### Recommendations to INFCOM

*   New work item: Incorporate OGC-API Maps into WIS2 Tech Regs + MetOcean profile
*   New work item: Incorporate OGC-API EDR into WIS2 Tech Regs + MetOcean profile
*   New work item: Guidance on encoding data in Web-friendly formats – CoverageJSON, GeoJSON, JSON-FG.
*   New work item: SC-IMT engage with OGC-API Vocabulary SWG via the WMO-OGC MoU – ensure the specification meets needs of WMO.

### Example implementations

*   ECMWF's polytope/EDR work, Tom's "dataless" pygeoAPI plugin, and Mark's serverless implementations
*   PygeoAPI
*   Reference implementation
*   Polytope \[yes – done] ECMWF\* … earthKit plots as a “Map provider” into pygeoAPI Maps \[to do] Adam
*   x-array (via Zarr or Icechunk) for NWP data \[yes – public beta in Feb] Canada
*   Serverless lambda implementations for EDR \[yes] and WMS Maps \*\* \[to do] Mark

***

## C. Custom data processing

Recommend use of OGC-API Processes Part 1 for data publishers (service providers?) to offer a menu of things that users can ask them to execute on the data in-situ.

> Cost implications for data processing on behalf of a client  
> Our recommendations need to be clear that *SOMEONE ALWAYS PAYS*

*   OGC-API Processes part 1
    *   standard \[done]
    *   PygeoAPI demo \[done]
    *   Suggested MetOcean functions for NWP centres to offer \[Tom]; \[develop a OGC-API Processes Part 1 profile for MetOcean?]
    *   Pub/sub call back << Global Discovery Catalogue implementation \[Tom]
    *   Server side data processing costs to managed!!! Resource utilization
*   OGC-API Process part 2
    *   Highlight it exists – at least so they don’t think we forgot
    *   Identify challenges around security and resource utilization
    *   Allow best practice to emerge

***

## D. Toolkits for local NMHS operations

Recommendations are:

*   "follow the WIS2Box pattern". A containerised, open-source (reference?) implementation that resource-constrained Members can deploy, enabling them to leverage the recommendations in themes (B) and (C).
*   Endorse WMO establishing and maintaining an open-source project to do this.
*   As input we develop an MVP to show what can be done. Working code trumps everything 🙂
*   Scope of MVP? “metdatauser-demo”
    *   docker compose based stack
    *   User interface – vue.js or react
    *   “Local” API or remote API hypercube access
    *   Minio as storage service
    *   Don’t need Grafana etc.
    *   Front with nginx
    *   Subscribe to WIS2 notifications to be aware of new data

> Completing work by April 2026.  
> Who should we ask to deploy and test?  
> … Dwayne / Belize << target Dwayne for Feb  
> … India (end-Jan)  
> … WIS2 Champions <<  
> … get it deployed and get feedback!  
> Recommendation > INFCOM should continue to invest in this MVP because it provides utility for Members … use OpenWIS Association -> WIS2 trust fund  
> ET-Open Source? This MVP work should follow the forthcoming WMO FOSS guide.

***

## Footnotes

[^1]: What do you do with 1PB when it’s downloaded? And how do you keep it synchronised with the source? ERA5 = 5PB, ERA6 predicted to be 30 PB.

[^2]: Users can provision services, such as compute time or storage, as they require them using automated tools without the intervention of humans from the cloud provider.

[^3]: The capacity of services can be expanded and decreased rapidly on-demand by the users, without the intervention of the service provider. For example, additional processing power during peak periods of usage.

[^4]: When ECMWF offered Zarr to a limited pool of users the system ground to a halt due to resource contention.

[^5]: ECMWF abandoned reliance of filesystem access a few years ago. MARS is entirely based on object storage.

***
