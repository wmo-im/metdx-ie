# SG-FIT least disruptive  
  
[not attempting to provide a Guide or technical documentation]  
[we’re still debating whether to use the term “==hypercube==”]  
  
## PREAMBLE  

[candidate text for the open paragraphs introducing the work of SG-FIT]

In the weather and climate community, the existing data distribution model pushes copies of data to recipients. But with growing data volumes, scaling becomes increasingly complex and cost prohibitive.  
  
Future Data Infrastructure: Don’t move (all) the data.  
  
Data usage or data “processing” can take many forms – from computing pre-defined statistics, to retrieving a part (or sub-set) of a dataset, to rendering data as an image, or bespoke computing such as model training.   
  
The challenges with large data volumes are threefold:  
1. Large data volumes are difficult, if not impossible, to move in a timely and cost-effective manner. This is often referred to as “data gravity”.  
2. Even if one could move large volume data, many users lack the necessary local capacity and capability to manage a copy of the data.   
3. And finally, it’s much harder to confirm the authenticity and veracity of datasets when working with copies (e.g., they may have been subject to local modification).   
  
ECMWF’s Re-analysis (ERA) dataset, the training data for most ML weather models, contains decades of simulated weather that’s fitted to real observations. ERA5 is 12 petabytes. Version 6 is predicted to be ~25 petabytes. What do you do with several petabytes even when it’s downloaded?  
   
Strategies for better (more effective) data sharing:  
1. Let users download only the data they need (Web-based data APIs);  
2. Let users work with data in place (Cloud-hosted, cloud-optimised data and proximate compute);  
3. Give users a latent representation of the data which is highly compressed (AI compression);  
4. Move data generation closer to the users (Machine Learning Weather Prediction - MLWP).  
  
*In this work the non-technical foundations for data sharing, such as licensing, are out of scope.*  
  
## CLOUD  

[candidate text for inclusion in the Cloud section]

### Cloud characteristics  
  
Cloud Platforms offer services for system developers to create, deploy, and operate their systems without needing to be concerned with the details of managing lower levels of the platform services they rely upon.  
  
There is no standard definition for “Cloud” or some of the other related terms. There is, however, broad agreement in the understanding and terms used when discussing Cloud platforms. The distinguishing characteristics of Cloud Platforms, when taken together, are:  
* **On-demand self-service** — Users can provision services, such as compute time or storage, as they require them using automated tools without the intervention of humans from the cloud provider.  
* **Leverage pooled resources** — The services offered by the cloud provider share a pool of resources, such as processors or storage, which are opaque to the consumers. Actual location or specific identity are abstracted away from the consumer. Capacity management, pooling and time-sharing management are the responsibility of the cloud provider to assure the cloud services’ availability and reliability.  
* **Rapid capacity elasticity** — The capacity of services can be expanded and decreased rapidly on-demand by the users, without the intervention of provider’s people. For example, additional processing power during peak periods of Web-site usage.  
* **Support multiple sandboxed systems** — Systems operating on the cloud are securely independent and non-interfering even though they share a pooled resource base.  
* **Unified access to metered services** — There are a common set of credentials for accessing all the services available to a user (one login), and the usage of those services can be monitored in near-real-time as a usage-based metered services  
  
[**Cloud-native**: Cloud-native means that applications are built to exploit the cloud’s elasticity, automation, and distributed nature—not just hosted on virtual machines.]  
  
### Proximate compute  
  
Why does the computation need to happen so close to the data?  
* Applications using or processing data need to read it into memory.  
* Data read speed is affected by network bandwidth and geographic distance.  
* Processing data close to where it is stored will improve application performance.  
* If we don’t move (all) the data – we need to compute where the data is published.  
  
Compute in parallel, scale elastically. Self-service, on-demand, elastic provisioning means that:   
* You can meet demand from a large user-base; and  
* Applications can turn a time-bound problem (1 core x 1000 seconds) into a resource-bound problem (1000 cores x 1 second).  
  
## LEAST DISRUPTIVE  
  
We use the term “least disruptive” for approaches that complement existing WMO data sharing workflows: dataset discovery and file-based data publication via WIS2. The core elements of data sharing via WIS2 are described in the WIS TECHNICAL SPECIFICATIONS [^1]:  
* WIS-TECHSPEC-1: MANAGING DISCOVERY METADATA  
* WIS-TECHSPEC-2: PUBLISHING DATA AND DISCOVERY METADATA  
  
These technical specifications require that:  
1. Data and metadata are accessible using open protocols (HTTP).  
2. Encode data in open, community-agreed formats (e.g., for WMO World Weather Watch use GRIB2 and BUFR4).  
3. Provide suitable discovery metadata (WMO Core Metadata Profile v2 [WCMP2]).  
4. Provide Publish-Subscribe notifications (MQTT) about the availability of new or updated data and metadata (WIS2 Notification Message [WNM]).  
  
(1), (2) and (3) combined with the WIS2 Global Services and the WMO Codes Registry [^2] go much of the way to delivering the FAIR Principles [^3]. (4) notifies users/agents (humans and software systems) in near-real-time at scale when new or updated data and metadata becomes available.   
  
Importantly, these “least disruptive” data sharing approaches do not replace or require changes to existing systems or ways of working.  
  
The approaches outlined below are derived from implementation experience. The Study Group has researched where these approaches have been adopted and built demonstration software components in a test-bed to illustrate how these approaches deliver improved data exchange.   
  
### Web-based APIs  
  
A Web-based Application Programming Interface (API) is a way for one computer program to use standard Web requests (via HTTP) to ask another program hosted elsewhere on the Internet for data or to perform an action. One does not need to download and interpret a set of resources (i.e., data files), instead a request can be sent to the remote computer to perform an operation on those resources.      
  
Web-based APIs:  
* Describe the operation(s) they can perform.   
* Can be configured with access controls.  
* Should use compression schemes to further reduce the volume of data sent, relying on well-adopted schemes (e.g., gzip or brotli).  
  
Three types of operation are particularly relevant where one aims to reduce the volume of data sent to the user:  
1. *Subsetting* — Requesting a small part from a large dataset.  
2. *Visualisation* — Rendering data into pictures or animations.  
3. *Processing* — Executing a function and returning only the result (e.g., calculating statistical summaries).  
  
#### OGC APIs  
  
Since 2009, a Memorandum of Understanding (MoU) has been in place between the Open Geospatial Consortium (OGC) [^4] and the WMO to ensure that the standards adopted in the geospatial world are fit for meteorology.   
  
Consequently, [OGC APIs](https://ogcapi.ogc.org) are a good match for data-sharing needs within the meteorological community. The key benefits of OGC APIs are:  
  
* *Developer accessibility* — OGC APIs use standard Web addresses and HTTP methods to interact with resources (i.e., REST conventions), and return JSON responses. A Web developer can consume geospatial data using the same tools and mental models they use for any other API, thus widening the pool of people who can work with geospatial data.  
* *Discoverability & self-documentation* — Each OGC API exposes an OpenAPI document [^5], meaning services are machine-readable and can be explored via commons tools. You don't need specialist knowledge just to understand what a service offers.  
* *Modularity* — The suite of operations is realized by reusable building blocks: OGC API – Features, Tiles, Maps, Coverages, Processes, Records, and so on. Implementers adopt only the parts they need, rather than one monolithic standard.  
* *Web-native interoperability* — Like WIS2, OGC APIs are Web-native, making deployment and scaling straightforward.  
  
Data publishers have a standard way to expose geospatial datasets that modern consumers can actually use without friction. Application developers to integrate geospatial data without learning a new stack of GIS-specific tooling. Enterprises benefit from reduced vendor lock-in since the standards are open and broadly implemented.  
  
Three of the OGC APIs are particularly relevant:  
1. *OGC API — Environmental Data Retrieval (EDR)* is a standard Web API specification that provides a simple, consistent way to query and retrieve subsets of environmental data — such as weather observations, forecasts, or climate data — from large datasets using spatial and temporal patterns like points, areas, or trajectories, without needing to download the entire dataset. The specification is published by the Open Geospatial Consortium and is available at [ogcapi.ogc.org/edr](https://ogcapi.ogc.org/edr).  
2. *OGC API — Maps* is a standard Web API specification that allows clients to retrieve geo-referenced map images (rendered visual representations of spatial data) from a server by requesting a defined area, scale, and style over the web. The specification is published by the Open Geospatial Consortium and is available at [ogcapi.ogc.org/maps](https://ogcapi.ogc.org/maps).  
3. *OGC API — Processes Part 1* is a standard web API specification that enables clients to discover, execute, and retrieve the results of geospatial processing operations hosted on a remote server — such as running a spatial analysis, a data transformation, or calculating summary statistics — using standard HTTP requests. The specification is published by the Open Geospatial Consortium and is available at [ogcapi.ogc.org/processes](https://ogcapi.ogc.org/processes).  
  
OGC APIs require a server to interpret and execute requests from users. An advantage of this approach is that complex data storage is not exposed to the users; they only see how the service operator has chosen to organise the data for them. However, a server will consume compute resource - which may be non-trivial if serving popular data to a large community.   
  
Generally, Earth-system datasets are too big to publish as single resource: most data publishers provide datasets as sets of objects or files. In fact, this commonly used pattern is how WMO has been publishing data for decades (e.g., NWP model data as GRIB files). Rather than providing a server to interpret users’ requests, users can download the subset of objects or files that contain the data they need. The challenge is helping users determine which of the objects or files contain the data they need.  
  
ECMWF and NOAA recognise the problem and have independently adopted data models, tools and services which allow users to find the data they need. This is not standardised; ECMWF and NOAA use different tooling and formats.  
  
A standardised approach is adopted by OGC providing a fourth, albeit slightly different, kind of Web API:
<ol start="4">  
<li>STAC — SpatioTemporal Asset Catalog is a community-driven suite of specifications that provides a common, JSON-based structure for describing and cataloguing geospatial assets — such as satellite imagery, aerial photography, or climate datasets — making them easily discoverable, searchable, and accessible on the web through a consistent set of metadata fields and links. Originating from the Earth observation community, STAC has been adopted by the OGC as a community standard, reflecting its broad uptake across government agencies, cloud providers, and the geospatial industry as a de facto approach to publishing and discovering spatio-temporal data at scale. The specification and further resources are available at <a href="https://stacspec.org/">stacspec.org</a>.</li>
</ol>   
  
#### Profiles  
  
Open standards, such as OGC APIs, define how things should work. Yet open standards need to allow degrees of flexibility to be broadly applicable leaving many “degrees of freedom”, for example: which operations to support, which controlled vocabularies to use, how to respond to errors, etc. Developers are left to make their best judgements about how to implement services given the options available.   
  
In a federated system comprising many data publishers and many data consumers, this kind of flexibility impedes interoperability. For example, a service to publish meteorological observations may be implemented differently among a group of National Meteorological Services, meaning that a data consumer will have to work harder to consume meteorological observations from each NMS — maybe juggling different terminology, maybe having to customise how their application works with each data source.  
  
Put another way, open standards are intentionally designed for flexibility, but that flexibility becomes a point of friction as deployments multiply.  
  
Profiles — or more specifically, “restrictive profiles” — are a means to address this friction. A profile removes ambiguity by defining additional rules on top of a standard controlling how it should be implemented. A profile may define things like: use of optional query patterns, parameter naming conventions, required response formats, CRS support.   
  
Essentially, use of a profile means trading flexibility for predictability: services follow tightly specified rules, client applications will know what behaviour to expect from those services. A profiled service distributing a particular kind of data should be interoperable — interoperable in the sense that one client or library should be able to use EDR services from different projects and institutes to query data without having custom code or plugins for each service.  
  
Profiling is a well-known pattern for interoperability. The approach is now being formalised in OGC API EDR Part 3: Service Profile Support [^6]. In Europe, EUMETNET is developing a profile for publishing meteorological data via OGC API EDR [^7] with the objective of harmonising data service provision across the 31 Member National Meteorological Services as part of its MeteoGate federated data platform.  
  
To support interoperable data exchange within WIS2, service profiles are required for OGC API Maps, EDR, and Processes and for STAC for each type of data that is exchanged.  
  
#### Implementation evidence  
  
* pygeoapi an open source Python server Reference Implementation of the suite of OGC APIs. pygeoapi is used extensively by numerous organizations (including Environment and Climate Change Canada and ECMWF).  ECCC's [MSC GeoMet API platform](https://eccc-msc.github.io/open-data/msc-geomet/readme_en) uses pygeoapi in 24/7 production to provide access to NWP, radar, alerts, archive climate and water data, real-time hydrometric data, and surface weather observations. [https://pygeoapi.io](https://pygeoapi.io)   
* WIS2 in a box (wis2box) is a Reference Implementation of a WIS2 Node. It includes a dedicated wis2box-api component providing OGC APIs to discover, access, and visualise notifications, data collections, and configurations (datasets and stations). This implementation is powered by pygeoapi. [https://docs.wis2box.wis.wmo.int/](https://docs.wis2box.wis.wmo.int/)   
* ECMWF Polytope is an open source library for extracting complex data from datacubes. Its API enables any arbitrary n-dimensional polygon (called a polytope) to be extracted from a datacube, allowing for efficient extraction of complex features, such as polygon regions or spatio-temporal paths. Like OGC API EDR, Polytope encodes data in CoverageJSON. Polytope also mirrors the query patterns of OGC API EDR: feature extraction allows users to request standard meteorological features such as time series, vertical profiles, and arbitrary polygons, retrieving only the data they need rather than downloading global fields. Polytope is a data service served by ECMWF for their operational data and Destination Earth data, but can also be used effectively as a backend to plugin to pygeoapi. [https://github.com/ecmwf/polytope](https://github.com/ecmwf/polytope)   
* ECMWF earthkit is an open source library, providing powerful tools for speeding up weather and climate science workflows by simplifying data access, processing, analysis, visualisation and much more. earthkit-data module makes it easy for users to read, inspect, and slice data from a wide range of geospatial input types, with a dedicated component to handle CoverageJSON data served by the Polytope.  [https://github.com/ecmwf/earthkit](https://github.com/ecmwf/earthkit)  
* EUMETNET MeteoGate is a federated data sharing platform developed and operated by EUMETNET that provides a unified technical infrastructure for the discovery and access of meteorological and hydrological data across Europe and beyond. MeteoGate enables European National Meteorological Services to openly share their data through a combination of an API Gateway, a Data Explorer for discovery and browsing, and integration with WIS2. The platform emerged from the EU- and EUMETNET-funded RODEO project and is designed to meet EU obligations under the Open Data Directive and the High Value Datasets regulation, bringing together data assets including land-based surface observations, weather radar composites, climate datasets, and severe weather warnings. OGC API EDR is the mandated standard for data access across MeteoGate, representing one of the most significant real-world deployments of OGC APIs in the European meteorological domain. Data access components within MeteoGate use OGC API EDR as the MeteoGate-compliant standard for providing interactive API access to datasets and collections. The E-SOH (EUMETNET Supplementary Observations Hub) operational system, run by DWD on the European Weather Cloud, uses the OGC API EDR interface for retrieving land-based surface observations. EUMETNET has further invested in the wider OGC API ecosystem by publishing a MetOcean profile of OGC API EDR, defining meteorological community conventions for parameter naming, response encoding using CoverageJSON, and coordinate reference systems. [https://meteogate.eu](https://meteogate.eu)   
* FMI SmartMet Server demonstrates evidence for OGC API adoption at operational scale. SmartMet Server is a high-capacity, high-availability data and product server for MetOcean data, written in C++ and in continuous operational use at FMI since 2008, underpinning FMI's Open Data Portal since 2013. The platform follows a plugin-based architecture in which individual OGC API standards are implemented as discrete, interchangeable plugins. The server is published under the MIT licence and freely available on GitHub, meaning FMI's implementation has become a reusable reference platform that other National Meteorological Services can adopt. Notably, FMI's EDR API and Surface Observations API are cited directly in EUMETNET MeteoGate documentation as reference implementation examples for Data Publishers looking to understand how to publish data in a MeteoGate-compatible, OGC API EDR compliant setup. [https://github.com/fmidev/smartmet-server](https://github.com/fmidev/smartmet-server)    
* The Danish Meteorological Institute (DMI) has made OGC API standards the foundation of its open data platform — notably OGC API Features for observation and climate data, and EDR for forecast data. For bulk forecast file access, DMI additionally offers a Forecast Data STAC API covering the same underlying data, giving users a choice between file-level discovery and download via STAC, or on-demand sub-selection via OGC API EDR depending on their use case. DMI’s open data platform is openly accessible without authentication as of December 2025. [https://www.dmi.dk/friedata/dokumentation/basics](https://www.dmi.dk/friedata/dokumentation/basics)  
* STAC Index is a community maintained registry of STAC catalogues and APIs. The STAC index lists a large and growing number of catalogues spanning satellite imagery, climate data, Earth observation archives, and more — from providers including NASA, ESA/Copernicus, Microsoft Planetary Computer, USGS, and many others. [https://stacindex.org](https://stacindex.org/)  
* Radiant Earth STAC Browser is a UI for browsing and searching STAC catalogues listed on the STAC Index. It is open source and hosted on GitHub by Radiant Earth. [https://radiantearth.github.io/stac-browser/](https://radiantearth.github.io/stac-browser/)   
* DestinE Data Lake Harmonised Data Access (HDA) API provides a STAC interface for data discovery and access. [https://destine-data-lake-docs.data.destination-earth.eu/en/latest/dedl-discovery-and-data-access/Harmonized-Data-Access/API-Architecture/API-Architecture.html](https://destine-data-lake-docs.data.destination-earth.eu/en/latest/dedl-discovery-and-data-access/Harmonized-Data-Access/API-Architecture/API-Architecture.html)   
  
### Objects not files  

[we edited this section to be a bit more accurate but now I'm wondering why we even talk about object stores. I don't think anything we are doing with OGC standards is particularly tied to object storage? Its obviously convenient and a good way to store data that we then serve with OGC APIs, but does it actually matter here?]
  
Three storage interfaces are in common use today, each presenting a different model to applications:

1. Block stores expose storage as a flat array of fixed-size blocks, addressed by number — the lowest-level interface, usually chosen for raw IO performance.
2. Filesystems present a hierarchical namespace of named files with POSIX-style read/write/seek semantics — the dominant interface for general-purpose software.
3. Object stores present a flat keyspace of blobs with rich metadata — designed around large, network-distributed data.

These interfaces are usually layered. ECMWF's exabyte-scale MARS archive, for example, exposes meteorological data as an object store while using filesystems and block storage underneath. For meteorological data at this scale, the object-store abstraction has proven the right choice. Modern cloud-native object stores make it easier than ever to deploy, abstracting both data access and service management, and complement elastic compute with equally elastic IO.
  
Cloud-native Object stores have the following characteristics:  
* *Scalability* — Object stores are designed to scale horizontally to exabytes of data.  
* *Durability and availability* — Data can be replicated across multiple (geographic) zones with very high durability and automated integrity checking (e.g., S3 provides “11-nines” durability).  
* *Cost effective* — Service providers may offer tiered storage (e.g., hot, cool, archive) to optimise cost based on access patterns; essentially, this means offsetting costs by accepting slower read speeds.  
* *Access via HTTP APIs* — Objects (and byte-ranges within objects) are accessed via RESTful APIs making them ideal for distributed processing, serverless workflows (e.g., Spark, Dask, ML pipelines).  
* *Event-driven integration* — Support for triggers on object creation or modification which is useful for automating workflows.   
  
In summary, publishing data as objects enables cost effective yet massively parallel querying and near-infinite scaling of durable storage. Consequently, object storage is a great choice for publishing large data volumes to a big audience. Support for byte-range queries (i.e., requesting just a part, a range of bytes, from an object) means that data users can further reduce the volume of data downloaded.  



[is this needed?]
> [!NOTE]  
> **Examples of Tiered Storage Classes**: AWS S3 storage classes [https://aws.amazon.com/s3/storage-classes/](https://aws.amazon.com/s3/storage-classes/)   
> * S3 Standard: Hot tier for frequently accessed data. General purpose. Amazon S3 provides **99.999999999% durability** (often referred to as "11 nines") for objects stored in its standard storage classes.  
> * S3 Intelligent-Tiering: Automatically moves objects between frequent and infrequent tiers. Unknown or changing access patterns.  
> * S3 Express One Zone. High performance, single Availability Zone. Up to 10x data access speed compared to Standard.   
> * S3 Glacier Instant Retrieval / Flexible Retrieval / Deep Archive: Cold storage for archival data. AWS lifecycle policies automate transitions between these classes. Offset cost against slower retrieval speed.  
>

  
#### Implementation Evidence  
  
[Annex 1](#annex-1.-meteorological-open-datasets-published-on-cloud-platforms) provides a list of open meteorological datasets that are published on cloud-platform object stores. This list is not intended to be exhaustive, only to illustrate that publishing data in this way is common place.  
  
### Analysis-Ready, Cloud-Optimised (ARCO) data  
  
Earth-system datasets are large, are growing, and are increasingly accessed by users and applications that have neither the capacity nor the desire to download them. Two related properties make a dataset useful in this setting:  
  
* *Analysis-Ready* — the data is directly usable for analysis without further transformation. It is findable, accessible, interoperable and reusable (FAIR), and the burden of preparation on the user is as low as practicable. What counts as "ready" is consumer-dependent: different analyses require different preprocessing, and a dataset that is analysis-ready for one community may not be for another.  
* *Cloud-Optimised* — access to the data is optimised for minimal data transfer and minimal cost. Reading is stateless or streamed, does not depend on the user's local filesystem, and allows selective access to discrete segments of the underlying data without retrieving the whole.  
  
Together — Analysis-Ready, Cloud-Optimised, or *ARCO* — these properties allow many users to extract just the data they need from very large datasets, on demand, without copies. ARCO is a property of how data is *accessed*, not of any particular file format on disk. The same primary data can be exposed through several ARCO access paths in parallel, each tuned for a different class of user.  
  
### Cloud-optimised access  
  
Cloud-optimised access has a small set of characteristic properties, regardless of how it is implemented:  
  
* *Selective access to discrete segments* — clients can fetch arbitrary subsets (a region, a time-series at a point, a single parameter at a single step) without reading the whole dataset.  
* *Stateless or streaming reads* — access works over standard HTTP, with no dependence on the client's local filesystem and no large local working copy.  
* *Minimised data transfer* — the bytes returned are close to the bytes the analysis actually needs; metadata and indices are arranged so that the number of reads required to locate and fetch those bytes is small.  
* *Parallelisable* — many small requests can be issued in parallel, allowing applications to turn a time-bound problem into a resource-bound one (see *Proximate compute*, above).  
* *Lazy materialisation* — the dataset is described to the client as a logical n-dimensional structure (a ==hypercube==); concrete bytes are fetched only as the application reads from regions of that structure.  
* *Decoupled from primary storage layout* — the access path is described independently of the underlying storage structure, so the same logical view can be served from different physical layouts, or from copies tuned for different access patterns.  
  
These are properties of an *access pattern*, not of a single on-disk format. There are several complementary ways to deliver them, and a data publisher will typically combine more than one.  
  
#### Approaches to cloud-optimised access  
  
Three complementary approaches have emerged in the weather and climate community. They are not mutually exclusive: the same primary archive can be exposed via more than one, and a publisher will typically pick the combination that fits their data, their users and their infrastructure budget.  
  
1. *Cloud-optimised formats* — write (or rewrite) the data into a chunked, compressed format that is itself designed to be served over HTTP. Each chunk can be fetched independently. This gives the best raw read performance for the chunking dimensions chosen, at the cost of an extra copy of the data.  
2. *Feature extraction over native data* — keep the data in its primary archive format and provide a service that, given a high-level request (a region, a trajectory, a time-series), jumps to and returns only the relevant bytes. This avoids copies but requires a server with knowledge of the underlying data layout.  
3. *Virtual cloud-optimised views* — present a cloud-optimised interface (for example a Zarr-shaped API) that is dynamically backed by the primary archive, with no data duplication. This combines the familiar client-side API of approach (1) with the no-copy property of approach (2).  
  
##### Cloud-optimised formats  
  
A cloud-optimised format stores data as many small, independently addressable chunks, alongside metadata that describes the logical n-dimensional structure of the dataset. Clients read the metadata once to build an empty in-memory representation of the dataset, then issue parallel HTTP (range) requests for just the chunks they need.  
  
The key characteristics of a cloud-optimised format are:  
  
* Resources are described independently of their underlying storage structures. A single parameter spanning an entire model run can be presented as one logical resource, rather than thousands of small files.  
* The number of reads needed to locate the required bytes is minimised — metadata is packaged together so that clients do not need to open many objects to find what they want.  
* Data is stored as small addressable chunks (files, tiles, or both), so reads can be parallelised into many small requests.  
* The chunking is hidden from the user; the format library converts logical reads on the dataset into the corresponding chunk reads.  
* Lazy-loading is supported — the metadata is used to build an empty hypercube in memory, and chunks are fetched only when the application reads from them.  
* HTTP range-requests are supported, so clients can fetch sub-chunk byte ranges where useful.  
  
Cloud-optimised formats are typically paired with client libraries that hide the chunking and IO entirely, so that data analysts can work with familiar n-dimensional array abstractions in their notebooks rather than managing parallel HTTP requests by hand.  
  
> [!NOTE]  
> **Why chunking?**  
> * *Scalability* — Chunking breaks large datasets into smaller, manageable pieces.  
> * *Performance* — Only the needed chunks are read or processed, reducing memory and compute load, increasing application speed; smaller chunks can be read more quickly, avoiding contention on IO.  
> * *Storage Efficiency* — Chunks can be compressed individually.  
> * *Streaming & Access* — Chunks allow partial reads of a dataset; most applications don’t need everything.  
>  
  
> [!NOTE]  
> **Chunking complexity**  
> * Chunk-shape impacts read performance. for example, consider how many chunks need to be read for an area query vs time-series query where coverage data is chunked into 2-d (x,y) layers at a single timestep; the time-series read will have to open *many* chunks, each of which incurs latency  
> * The objective of your chunking strategy should be to enable faster data retrievals by needing fewer object reads to access the bytes   
> * You need to think about how your priority users want to query the data and chunk your data to suit – optimise for the main query pattern, if “outlier” queries take longer then so be it  
> * Sometimes there’s no way to avoid supporting different query patterns – in which case you may need to persist the data twice with each copy chunked differently and exposed through a different end-point  
>   
> *Detailed guidance on chunking strategies is beyond the scope of this report.*   
>  
  
The cost of approach (1) is that the data has to be (re)written into the cloud-optimised format, and a given chunking is only optimal for a particular direction of access (for example, geospatial vs. time-series). Where users want to query the same underlying data along very different axes, multiple chunked copies may be required. For high-volume primary archives, this cost is significant and can dominate the storage budget.  
  
Cloud-optimised formats also tend to assume elastic, non-shared storage that can scale on demand. They typically have no native notion of queueing, throttling or quality-of-service, which can become a problem when serving popular data to a large user base from finite infrastructure.  
  
Cloud-optimised formats are available for point data, vector data, rasters, n-dimensional arrays (datacube / ==hypercube==), and point clouds. For more information see the Cloud-Optimized Geospatial Formats Overview [^8] from the Cloud-Native Geospatial Forum (CNG).  
  
##### Feature extraction over native data  
  
Where the primary data is held in a high-volume archive (for example, a multi-petabyte store of GRIB messages), rewriting it into a cloud-optimised format may be infeasible or undesirable. An alternative is to keep the data in its native layout and provide a service that performs cloud-optimised access on the user's behalf.  
  
Such a service accepts a high-level request describing the feature the user wants — a region, a polygon, a vertical profile, a spatio-temporal trajectory, a time-series at a point — and returns only the bytes corresponding to that feature, packaged in a streamable response format (for example, OGC CoverageJSON). Internally, the service uses indices over the native archive to identify which fields and which byte-ranges within those fields are required, and reads only those bytes. Compressed primary data can be supported provided the compression scheme allows partial decoding.  
  
This approach delivers the cloud-optimised access properties listed above without producing additional copies of the underlying data. It also lends itself naturally to quality-of-service: because every request is mediated by a server, the operator can apply queueing, throttling, authentication and metering. The cost is that the service must understand the structure of the primary archive, and is therefore tightly coupled to it.  
  
For meteorological datacubes, this can deliver substantial savings even compared with bounding-box requests on a chunked store: only the bytes that intersect the requested feature are read from the underlying IO system.  
  
##### Virtual cloud-optimised views  
  
A third approach is to present a cloud-optimised interface — for example, a Zarr-compatible store — that is not backed by a real chunked copy of the data but is instead dynamically synthesised from the primary archive. Client applications interact with it exactly as they would with a native cloud-optimised format; under the covers, each chunk read is translated into the appropriate read against the underlying archive.  
  
Two variants are common:  
  
* *Static virtual stores* — a one-off indexing pass over the legacy files produces a manifest mapping logical chunks to byte-ranges in the original objects. The manifest itself is small and cheap to host. Clients read it like a normal cloud-optimised dataset, while the actual bytes are streamed from the original files. Tools such as Kerchunk and VirtualiZarr implement this pattern over formats including netCDF, HDF5 and GRIB.  
* *Dynamic virtual stores* — the cloud-optimised view is generated on demand from a live archive, with no pre-built manifest. The client requests a chunk; the service queries its primary archive, identifies the corresponding fields and byte-ranges, and returns the chunk as if it had always existed. This allows a single archive to be presented under many different chunkings without storing any of them.  
  
Virtual stores combine the familiar client-side API of cloud-optimised formats with the no-copy property of feature-extraction services. They inherit the strengths of both — and some of the limitations: the underlying archive still needs an index that supports byte-range extraction, and dynamic virtual stores in particular still depend on a service to mediate requests.  
  
#### Choosing among the approaches  
  
The three approaches are complementary, not alternatives. A typical publisher of a high-volume archive will use a mix:  
  
* For "hot" data with a clear dominant access pattern (for example, point time-series for climate analysis, or dense datacubes for ML training), a *cloud-optimised format* copy gives the best raw performance and offloads serving cost from the primary archive. The data can always be rebuilt from the primary if the chunked copy is lost or corrupted, and storage of the copy can be isolated from time-critical primary storage.  
* For arbitrary, ad-hoc access across the full archive — especially extraction of features such as regions, profiles or trajectories — *feature extraction over native data* avoids producing one chunked copy per access pattern, and gives the operator natural levers for QoS over a shared resource.  
* For users who specifically want a cloud-optimised client API but where pre-materialising a chunked copy is not viable, *virtual cloud-optimised views* let the same primary archive be presented under whatever chunking the user asks for, on demand.  
  
In practice, well-designed ARCO offerings publish a small number of high-value chunked copies, expose the rest of the archive via feature extraction, and use virtual views to bridge the two — unified through standard catalogues and APIs (STAC, OGC, WMO, CF) so that consumers see one coherent ARCO offering rather than a set of disconnected services.  
  
#### Implementation evidence  
  
The examples below are grouped by the three approaches.  
  
##### Cloud-optimised formats  
  
###### Zarr  
  
Zarr [^9] is the de facto cloud-native storage format for chunked n-dimensional arrays in the geoscience community. Arrays are persisted as a Zarr store where data is split into chunks stored as individual objects in a directory-like structure, complemented with metadata describing content, shape, chunking and encoding. Zarr is well-supported by client tooling: for example, Xarray reads and writes Zarr directly and supports lazy-loading. Several extensions and complementary specifications (notably GeoZarr) address Zarr's relatively weak built-in metadata model for geospatial data.  
  
###### Google ARCO-ERA5  
  
A canonical example of a cloud-optimised format copy of a high-value dataset is the Analysis-Ready, Cloud-Optimized ERA5 dataset, hosted by Google as a public dataset on Google Cloud Storage. The underlying ERA5 reanalysis is distributed in GRIB and netCDF; for ARCO-ERA5, a subset has been re-encoded and re-chunked into Zarr stores that are directly loadable by Xarray over HTTP. The Zarr layout is chosen to favour the intended workloads (in particular ML training and time-series analysis at points). The original ERA5 data continues to be distributed in its native formats; the Zarr copy is an additional, purpose-built ARCO surface over it. [https://console.cloud.google.com/marketplace/product/bigquery-public-data/arco-era5](https://console.cloud.google.com/marketplace/product/bigquery-public-data/arco-era5)  
  
###### Environment and Climate Change Canada / Met Service Canada Virtual Optimal Forecast (VOF) over Zarr  
As part of their next generation forecasting data dissemination, MSC are using cloud-optimised data stores (Zarr hosted on OpenShift private cloud infrastructure) to serve all visualisations for the ECCC public website. Data from nowcast (0-6h), deterministic hi-res (6-48h), and  global (48h+) is stitched together to create a single “virtual” forecast and exposed through OGC API services using pygeoapi. To manage load on the private cloud infrastructure, the OGC API services and Zarr stores are not publicly accessible; the data may only be accessed via the public website.  
  
##### Feature extraction over native data  
  
###### ECMWF Polytope  
  
Polytope [^12] is a data extraction service that provides both access to full-field global data and feature extraction capabilities. It uses concepts of computational geometry to extract n-dimensional polygons (also known as polytopes) from datacubes, returning the result in OGC CoverageJSON.  
  
Polytope is deployed by ECMWF as a user-facing service over its petabyte-scale Fields DataBase (FDB) and over Destination Earth data stored in a distributed network of data bridges located at EuroHPC sites where digital twin simulations are run. The hosted service is fast: a time-series request for one location from a 50-member ensemble returns a 10 kB extract from an 80 TB dataset in around 10 seconds.  
  
ECMWF produces around 120 TiB of raw weather data each day, represented as a six-dimensional dataset. As model resolutions increase, daily production will rise into the petabytes — making distribution of full products and archived data impossible without in-situ, on-the-fly extraction of the kind Polytope provides.  
  
The core Polytope library and the client are both open source under the Apache 2.0 licence and available on GitHub at [ecmwf/polytope](https://github.com/ecmwf/polytope) and [ecmwf/polytope-client](https://github.com/ecmwf/polytope-client).  
  
##### Virtual cloud-optimised views  
  
###### VirtualiZarr and Kerchunk  
  
VirtualiZarr [^10] and its precursor Kerchunk [^11] are reference implementations of static virtual Zarr stores. They scan legacy files (netCDF, HDF5, GRIB — including compressed variants) and produce a manifest describing where each logical chunk lives as a byte-range within the original files. Clients then open the manifest as a normal Zarr dataset; the actual bytes are read from the legacy objects. This allows existing archives to be exposed as cloud-optimised resources in parallel with their continued use in legacy workflows, at the cost of generating and maintaining the indices.  
  
###### NASA evaluation of Earthmover Icechunk  
[Solving NASA’s Cloud Data Dilemma: How Icechunk Revolutionises Earth Data Access](https://www.earthmover.io/blog/nasa-icechunk/)  
NASA has been migrating over 100 petabytes of data from on-premises systems to the cloud over several years, with the volume expected to surpass 320 petabytes by 2030. The vast majority is multidimensional scientific data stored in archival formats like NetCDF and HDF.  
  
NASA partnered with Earthmover and Development Seed to pilot an approach based on Earthmover's open-source Icechunk tensor storage engine. The pilot applied Icechunk to the GPM IMERG precipitation dataset stored in S3, enabling high-performance cloud-native access without transforming the data to a new format. Icechunk's virtualization capabilities were used to present the entire collection of over a million files as a single analysis-ready Zarr data cube. VirtualiZarr was used to scan the files, producing an index describing the precise location of every chunk of data within each file, and these "chunk manifests" were stored in Icechunk's optimised format.  
  
This approach enabled a 100x speedup for extracting time series data compared to existing approaches, while also simplifying the end-user experience. The implications are significant when scaled to NASA's 300 PB archive — duplicating archival files into a cloud-native format would effectively double storage costs, while the Icechunk approach can deliver tens of millions of dollars of savings in storage costs alone.  
  
###### ECMWF zfdb  
  
[Partly Cloudy with a Chance of Zarr — A virtualised approach to Zarr stores from ECMWF's Fields Database, FOSDEM 2026](https://fosdem.org/2026/events/attachments/YNQRA7-partly-cloudy-with-a-chance-of-zarr/slides/267412/fosdem_26_dccuy9k.pdf)  
  
zfdb [^14] is an experimental open-source Python library that implements a Zarr store using ECMWF's Fields DataBase (FDB) as its backend, allowing users to access GRIB meteorological data stored in the FDB as if it were a native Zarr dataset. Rather than converting or duplicating data, zfdb creates virtual Zarr v3 views by translating Zarr access patterns into MARS-language requests — the same domain-specific vocabulary used to query ECMWF's meteorological archive — and leveraging the FDB's indexing of GRIB fields by semantic metadata such as parameter, level, date, and step. Unlike VirtualiZarr or Kerchunk, which operate on static file references, zfdb is dynamic: views are defined programmatically using MARS keywords and chunked along temporal or step axes, enabling integration with standard Python scientific tools such as xarray and zarr. Developed as part of the WarmWorld Easier project — which aims to improve interoperability of climate and weather data across European HPC centres — the library is described as experimental rather than a production system, but represents a significant step toward bridging GRIB-based data infrastructure with the cloud-native Zarr ecosystem increasingly adopted for machine learning and big-data analytics workflows.  
  
  
### Testbed  
  
The Study Group has created a testbed to illustrate how Web-based APIs and cloud-optimised data formats can be used to enable effective data sharing. The testbed deployment target is a technically competent low- to middle-income country. Focus is enabling effective use of weather prediction data.  
  
GitHub repository: [wmo-im/metdx-demo](https://github.com/wmo-im/metdx-demo/tree/main).  
  
Components include ([diagram](https://github.com/wmo-im/metdx-demo/blob/main/docs/architecture/c4.container.png)):  
1. Web-based user interface (discover datasets, select data from OGC API services and visualise).  
2. [TBD] QGIS client.  
3. Data discovery catalogue (Global Discovery Catalogue test instance; to host WCMP2 records for the test datasets used in the testbed).   
4. Data cube indexer (create virtual Zarr stores over legacy data files).  
5. Raw data cube index server (Garage object store — an S3 clone; for serving native Zarrs and Zarr indexes).  
6. Data subsetting server (OGC API EDR).  
7. Data visualisation server (OGC API Maps).  
8. Data processing and charting server (OGC API Processes).  
  
Components are deployable as Docker containers.  
  
OGC API implementations:  
* pygeoapi — with native support for Xarray resources.  
* Zarr plugin for pygeoapi — exposing native Zarr objects published by ECCC/MSC; model run instance identification based on hard-coded pattern matching on the object name.   
* earthkit plugin for pygeoapi — exposing polytope library with local Zarr store _AND_ polytope service with ECMWF FDB.  
* earthkit plots — OGC API Maps exposing Destination Earth digital twin data using polytope service and FDB.  
* earthkit workflows — OGC API Processes (simple chart/timeseries renderer).  
* Serverless EDR implementation — AWS lambda exposing Met Office / ECMWF indexed CF-NetCDF model data.  
* Serverless Maps implementation — AWS lambda (with matplotlib) exposing Met Office / ECMWF indexed CF-NetCDF model data.      
  
Virtual Zarr over indexed GRIB:  
* Data sources: NOAA GFS; ECMWF AIFS ensemble.  
* Kerchunk indexer (with cfgrib codec) plugin for pygeoapi — exposing remote GRIB datasets on Amazon Open Data Registry. [TBD: pygeoapi plugin configured from STAC catalog (e.g., model run instances)]  
* Custom STAC catalog to describe model run instances; templated STAC collection/item, instances generated on (cron) trigger.  
* Indexer configurable for given data source based on file organisation, naming conventions etc. (object-naming, structure and organisation is a sovereign choice that we shouldn’t attempt to standardise); indexes generated on (cron) trigger.  
* Dimensions (e.g., z = abc) identified in the pygeoapi xarray-core config, so pygeoapi knows which label relates to x, y, z, t dimension (& ensemble).  
* “Dataless” OGC API server:   
    * OGC API server only has the indexes locally; data is lazy-loaded from remote location.  
    * Client applications don’t need to know about the back-end GRIB objects.  
    * OGC API server doesn’t need to know about the back-end GRIB objects — only that the remote data is encoded in GRIB so it needs the cfgrib library.  
* Parameter names and axes from source data propagate through to the indexes. [TBD: Mapping added to Zarr store .zattrs to explain mapping to standard names.]   
* Opinionated: one Kerchunk index per model run, single parameter.  
  
Example Kerchunk index:  
```JSON
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
  
Example usage in Xarray:  
```Bash
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
  
> [!NOTE]  
> **Note on xarray operations:**  
> * `open_dataset()` is the “Swiss Army knife” - but you need to tell it which dataset engine to use.  
> * `open_zarr()` is a convenience wrapper that only works with Zarr.  
> * `open-mfdataset()` opens multiple files and combines them into a single (in-memory) virtual datacube.  
>   
  
OCG API Maps profile:  
* [GitHub](https://github.com/opengeospatial/metocean-ogcapi-maps-profile), [draft spec](https://docs.ogc.org/DRAFTS/26-002.html)  
* Collection instance: single model run, all parameters with consistent domain (note: models output data on several vertical reference systems — pressure levels, surface, whole-earth, etc.; consequently a model run may map to several Collections).  
* Extend Maps to add custom parameter-name queryable (Maps operation is tied to Collection; a collection will likely include many parameters; we need to instruct the Map service which parameter to visualise). (discussion: [Issue opengeospatial/ogcapi-maps#142](https://github.com/opengeospatial/ogcapi-maps/issues/142)).  
* Use subset queryable for subsetting on enumerated dimensions (e.g., pressure level, ensemble member).  
* Dimensions and parameter names defined in the `/collections/{collection-id}/schema` endpoint (see [OGC API - Common - Part 3: Common: Schemas](https://docs.ogc.org/DRAFTS/23-058r1.html)).  
* Servers shall not interpolate between enumerated dimensions (instance, time-step, vertical level). On error, return a list of the valid values for the dimension.  
* Servers shall return a 2-d image in horizontal x-y plane (more sophisticated cases may be added in future, e.g., vertical slices, Hovmöller diagram). If the query isn’t constrained (e.g., z dimension is not specified) the server will choose how to reduce the data to a 2-d horizontal plane. This ensures that standard ogcapi-maps queries will work, albeit with limited dimensionality.  
  
Example maps query:  
```
https://example.com/collections/ca-eccc-msc-nwp-gdps/map?f=png&width=500&height=400&promperties=AirTemp2&datetime=2025-10-28T01:00:00Z&subset=vertical(0),instance_id("2026-01-16T00:00:00Z") 

```
  
OGC API Environmental Data Retrieval (EDR) profile:   
* [GitHub](https://github.com/EUMETNET/metocean-edr-profile), [draft spec](https://eumetnet.github.io/metocean-edr-profile/standard/metocean-edr-profile-DRAFT.html) ==pending merge of [PR#75](https://github.com/EUMETNET/metocean-edr-profile/pull/75)==  
* Collection instance: single model run, all parameters with consistent domain (note: models output data on several vertical reference systems — pressure levels, surface, whole-earth, etc.; consequently a model run may map to several Collections).  
* Parameter names should be consistent, concise and human readable. Propose a standard list of names for common parameters (the 16 parameters defined in Manual on WIPPS for global deterministic weather prediction data) based on ECMWF eccodes short-names.  
* Unified additional dimensions used to define enumerated dimensions (e.g., pressure levels, ensemble members).  
* Servers shall not interpolate between enumerated dimensions (instance, time-step, vertical level). On error, return a list of the valid values for the dimension.  
* Servers shall return CoverageJSON (MIME type: `application/vnd.cov+json`) as a minimum.     
  
Example position query: `coords=POINT(-75 45)&parameter-name=AirTemp`  
```
https://example.com/collections/ca-eccc-msc-nwp-gdps/position?f=json&coords=POINT%28%20-75%2045%29&parameter-name=AirTemp

```
  
OGC API Processes Part 1 profile:  
* None.   
* Propose to define which operations a modelling centre should provide to process their data. TBD.  
  
STAC profile:  
* None.  
* Suggest to use existing `datacube` and `projection` extensions.  
* Propose new extension `link_temporal_extent` that allows a catalog-browse client to determine the date-range of resources (Catalog/Collection or Asset) without needing to pre-fetch those resources, thus speeding up browsing a resource tree published using static JSON objects. temporal_extent can be used in a link-object for a Catalog/Collection, or in a asset-object for an Item. The value can be a time-instant, time-range, or repeating interval, expressed in ISO8601 (RFC3339) notation.   
  
N-dimensional array dataset (==hypercube==) description:  
* Based on Zarr store metadata.   
* JSON encoded.  
* ==Cube== instance: single model run, one variable (if ensemble: all members).   
* Where no STAC metadata provided, pattern match on filenames to identify model run instance (testbed shortcut; see ECCC/MSC pygeoapi plugin for reading WAF).  
  
Web-based UI:  
* Browse Global Discovery Catalogue to find datasets.  
* If a dataset has a link to a collection resource, then it is probably interactive (i.e., can be queried as an OGC API or STAC Collection). Don’t pre-fetch the collection resources yet — this could be expensive/slow. Add an icon to the search results indicating that it’s an interactive resource.  
* Example snippet from WCMP2 record:  
```JSON
        {
            "href":"https://example.com/collections/ca-eccc-msc-nwp-gdps",
            "type":"application/json",
            "rel":"collection",
            "title":"GDPS"
        }

```
* When viewing a record for a specific dataset, fetch the (JSON-encoded) collection resource.  
* If the collection resource includes an id property, this confirms it’s probably OGC API or STAC. Investigate further.  
* A collection may contain other collections; for example a collection for a weather prediction model will list as sub-collections all of the instances of that model run that are available (link-relation collection).  
* The collection resource for an EDR end-point describes the data_queries it offers, one or more of: position, radius, area, cube, trajectory, corridor, items, locations, or instances. There should be enough metadata available to build a UI component that constructs valid queries for each query-type.   
* Example collection resource fragment for query-type position:  
```JSON
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
* The collection resource for a Maps end-point includes a link object with link-relation `http://www.opengis.net/def/rel/ogc/1.0/map`. Fetch the `/collections/{collection-id}/schema` to get the metadata needed to construct valid queries.  
* The collection resource for a STAC resource will always declare a stac_version, for example:  
```JSON
  "stac_version": "1.1.0"

```
* A STAC resource may be of type Catalog, Collection or Item. Links to Catalog or Collection resources contained by a STAC Catalog or Collection are provided in link-objects with link-relation `child`. Links to Item resources are provided in link-objects with link-relation `item`. Links to the data resources are provided in an assets dictionary within a STAC Item. One can browse through the Catalog/Collection(s) to find Items; depending on the MIME-type of the asset, the WebUI may be able to add the data resource to the user’s session, using the metadata in the Item to provide necessary context.  
  
### Future Work  
  
The Least Disruptive work-stream of the Study Group has identified several elements that could be incorporated into WMO Technical Regulation. During the next intercessional period (2027-2028), the Standing Committee on Information Management and Technology (SC-IMT) is expected to pursue the following objectives under the work item “WIS 2.0 Evolution in Support of the Future Data Infrastructure and AI applications”:  
  
1. Develop a standard framework for the use of Web-based data APIs in WIS2.  
2. Collaborate with the OGC MetOcean Domain Working Group to develop OGC-API profiles and reference implementations for the use of OGC API Maps ([GitHub](https://github.com/opengeospatial/metocean-ogcapi-maps-profile), [draft spec](https://docs.ogc.org/DRAFTS/26-002.html)), EDR ([GitHub](https://github.com/EUMETNET/metocean-edr-profile), [draft spec](https://eumetnet.github.io/metocean-edr-profile/standard/metocean-edr-profile-DRAFT.html) ==pending merge of [PR#75](https://github.com/EUMETNET/metocean-edr-profile/pull/75)==), and Processes (Part 1) in the WMO context.  
3. Develop a standard, technology-agnostic mechanism for describing n-dimensional array datasets (i.e., ==hypercubes==) plus reference implementations.  
4. Develop guidance for provision of “==Analysis-Ready==, Cloud-Optimized” (ARCO) weather and climate data that is ready for use with AI applications.  
5. Develop guidance for the use of STAC to describe weather and climate datasets.   
  
The “WMO context” is broad, covering 7 Earth system domains: weather, climate, water (hydrology), atmospheric composition, oceans, cryosphere, and space weather. The Study Group identified the most impactful data-sharing use case to be the distribution of weather prediction model output. This type of data is produced by many centres and is provides an essential input into every Member’s public weather service and early warning system. Yet still, this data remains difficult for many least developed countries and small island developing states to leverage. Consequently, the work of SC-IMT will initially focus on driving effective sharing of weather prediction data. In parallel, SC-IMT will develop a roadmap for the application of these approaches into other application areas.  
  
Recognising the continued increase in data volumes from space-based observation platforms, and the need to ensure equitable access to such data products for all Members, SC-IMT welcomes the opportunity to collaborate with satellite operators to progress solutions relevant to their needs.     
  
+++  
  
[^1]: Manual on the WMO Information System (WMO-No. 1060), Volume II - WIS 2.0, PART IV WIS TECHNICAL SPECIFICATIONS [https://library.wmo.int/idurl/4/68731](https://library.wmo.int/idurl/4/68731)  
[^2]: WMO Codes Registry - Web-based publication of code tables from WMO Technical Regulation [https://codes.wmo.int/](https://codes.wmo.int/)   
[^3]: FAIR Guiding Principles for scientific data management and stewardship [https://www.go-fair.org/fair-principles/](https://www.go-fair.org/fair-principles/)   
[^4]: Open Geospatial Consortium, a member-lead organisation that defines open geospatial standards [https://www.ogc.org](https://www.ogc.org)   
[^5]: Open API Specification [https://swagger.io/specification/](https://swagger.io/specification/)   
[^6]: OGC API EDR — Part 3: Service Profile Support (Working Group draft) [https://github.com/opengeospatial/ogcapi-environmental-data-retrieval/blob/master/extensions/service_profiles/standard/25-014.adoc](https://github.com/opengeospatial/ogcapi-environmental-data-retrieval/blob/master/extensions/service_profiles/standard/25-014.adoc)  
[^7]: EUMETNET MetOcean OGC API EDR Profile (draft) [https://eumetnet.github.io/metocean-edr-profile/standard/metocean-edr-profile-DRAFT.html](https://eumetnet.github.io/metocean-edr-profile/standard/metocean-edr-profile-DRAFT.html)  
[^8]: Cloud-Optimized Geospatial Formats Overview [https://guide.cloudnativegeo.org/overview.html](https://guide.cloudnativegeo.org/overview.html)  
[^9]: Zarr specification [https://zarr.dev/](https://zarr.dev/)   
[^10]: VirtualiZarr [https://github.com/zarr-developers/VirtualiZarr](https://github.com/zarr-developers/VirtualiZarr)   
[^11]: Kerchunk [https://fsspec.github.io/kerchunk/](https://fsspec.github.io/kerchunk/)   
[^12]: ECMWF Polytope — An open-source library for extracting complex data from datacubes [https://github.com/ecmwf/polytope](https://github.com/ecmwf/polytope)   
[^13]: ECMWF earthkit — An open-source project providing powerful tools for speeding up weather and climate science workflows [https://github.com/ecmwf/earthkit](https://github.com/ecmwf/earthkit)   
[^14]: zfdb — A python-zarr v3 store implementation that provides a virtual Zarr store from FDB [https://github.com/ecmwf/fdb#z3fdb](https://github.com/ecmwf/fdb#z3fdb)  
  
  
+++  
  
# ANNEX 1. Meteorological open datasets published on cloud platforms  
Datasets published by meteorological services mapped against availability on three major open cloud data platforms.  
* **AWS** = [Registry of Open Data on AWS](https://registry.opendata.aws/)  
* **GEE** = [Google Earth Engine Data Catalog](https://developers.google.com/earth-engine/datasets/catalog)  
* **MPC** = [Microsoft Planetary Computer](https://planetarycomputer.microsoft.com/catalog)  
  
## ECMWF — European Centre for Medium-Range Weather Forecasts  

| Organisation | Dataset | Description | AWS | GEE | MPC |
| ------------ | -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | --- | --- | --- |
| ECMWF | ERA5 Reanalysis | Global atmospheric reanalysis at 31 km, 137 levels, hourly from 1940 to present. Temperature, wind, precipitation, pressure and more. | [↗](https://registry.opendata.aws/ecmwf-era5/) | [↗](https://developers.google.com/earth-engine/datasets/catalog/ECMWF_ERA5_HOURLY) | — |
| ECMWF | ERA5-Land Reanalysis | High-resolution (9 km) land-surface replay of ERA5. Soil moisture, snow depth, runoff, and 50 variables from 1950 to present. | — | [↗](https://developers.google.com/earth-engine/datasets/catalog/ECMWF_ERA5_LAND_HOURLY) | — |
| ECMWF | IFS Open Data (Real-Time Forecasts) | Operational medium-range IFS forecasts at 0.25°. Atmospheric, wave and ensemble products. Updated 4× daily. CC-BY-4.0. | [↗](https://registry.opendata.aws/ecmwf-forecasts/) | — | [↗](https://planetarycomputer.microsoft.com/dataset/ecmwf-forecast) |
| ECMWF | AIFS Open Data (AI Forecasting System) | Deterministic and ensemble AI-based forecasts, released from the same real-time feed as IFS open data. Added to open catalogue in 2024. | [↗](https://registry.opendata.aws/ecmwf-forecasts/) | — | [↗](https://planetarycomputer.microsoft.com/dataset/ecmwf-forecast) |
  
## EUMETSAT — European Organisation for the Exploitation of Meteorological Satellites  

| Organisation | Dataset | Description | AWS | GEE | MPC |
| --------------------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | --- | --- | --- |
| EUMETSAT / Copernicus | Sentinel-3 OLCI / SLSTR | Ocean colour, land surface and sea surface temperature from ESA/EUMETSAT Sentinel-3. NRT and offline products via Copernicus Data Space. | [↗](https://registry.opendata.aws/sentinel-3/) | — | — |
| EUMETSAT / ESA | Sentinel-5P (Tropomi) — Atmospheric Products | Daily global coverage of ozone, NO₂, SO₂, CO, CH₄, aerosol and cloud properties from the Copernicus Sentinel-5 Precursor. | — | [↗](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S5P_OFFL_L3_NO2) | [↗](https://planetarycomputer.microsoft.com/dataset/sentinel-5p-l2-netcdf) |
  
## NOAA — US National Oceanic and Atmospheric Administration  

| Organisation | Dataset | Description | AWS | GEE | MPC |
| ------------- | ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- | --- | --- | --- |
| NOAA / NESDIS | GOES-16 / 18 / 19 (ABI Level 1b & Level 2) | Full-disc and CONUS geostationary imagery every 10–15 min. Radiance, cloud height, fire, aerosol, rainfall products. Real-time + archive. | [↗](https://registry.opendata.aws/noaa-goes/) | [↗](https://developers.google.com/earth-engine/datasets/catalog/NOAA_GOES_16_MCMIPC) | [↗](https://planetarycomputer.microsoft.com/catalog?filter=goes) |
| NOAA / NWS | NEXRAD Level II & III Radar | Real-time and archive data from the US Next Generation Weather Radar network. Reflectivity, radial velocity, and derived products. | [↗](https://registry.opendata.aws/noaa-nexrad/) | — | — |
| NOAA / NWS | Global Forecast System (GFS) | Operational global NWP model at ~13 km resolution. 4 runs per day, 16-day forecast. Atmosphere, ocean and land parameters. | [↗](https://registry.opendata.aws/noaa-gfs-bdp-pds/) | [↗](https://developers.google.com/earth-engine/datasets/catalog/NOAA_GFS0P25) | — |
| NOAA / NESDIS | Joint Polar Satellite System (JPSS / Suomi-NPP / NOAA-20/21) | Polar-orbiting constellation measuring SST, vegetation, clouds, rainfall, snow/ice, fire, atmospheric temperature, water vapour, ozone. | [↗](https://registry.opendata.aws/noaa-jpss/) | — | — |
| NOAA / NWS | National Digital Forecast Database (NDFD) | Gridded sensible weather forecasts (temperature, cloud cover, precipitation type) from NWS field offices; seamless CONUS mosaic. | [↗](https://registry.opendata.aws/noaa-ndfd/) | — | — |
  
## Met Office — UK National Meteorological Service  

| Organisation | Dataset | Description | AWS | GEE | MPC |
| ------------ | ----------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | --- | --- | --- |
| Met Office | UKV Deterministic (2 km) — 2-Year Rolling Archive | High-resolution UK and Ireland NWP. Temperature, wind, humidity and pressure on a 2 km grid. NetCDF, updated ~3–6 h after each model run. | [↗](https://registry.opendata.aws/met-office-uk-deterministic/) | — | [↗](https://planetarycomputer.microsoft.com/dataset/group/met-office-uk-deterministic) |
| Met Office | Global Deterministic (10 km) — 2-Year Rolling Archive | Global NWP on a ~0.09° grid. Forecast parameters across pressure levels and surface. NetCDF; 600 TB+ archive. | [↗](https://registry.opendata.aws/met-office-global-deterministic/) | — | [↗](https://planetarycomputer.microsoft.com/dataset/group/met-office-global-deterministic) |
| Met Office | MOGREPS-UK Ensemble (2.2 km) — 30-Day Rolling Archive | UK ensemble forecast system. 5-day horizon, ~2.2 km resolution, multiple ensemble members. NetCDF via AWS. | [↗](https://registry.opendata.aws/met-office-uk-ensemble/) | — | — |
| Met Office | MOGREPS-G Global Ensemble — 30-Day Rolling Archive | Global ensemble at 20 km, 4 runs/day, out to 198 h. Run as part of the Unified Model operational NWP suite. | [↗](https://registry.opendata.aws/met-office-global-ensemble/) | — | — |
| Met Office | Global Wave Model — 2-Year Rolling Archive | Significant wave height, period and direction for open ocean and coastal waters. WAVEWATCH III configuration forced by Met Office global winds. | [↗](https://registry.opendata.aws/met-office-global-wave/) | — | — |
| Met Office | NWS Wave model — 2-Year Rolling Archive | Northwest European continental shelf regional wave model predicting sea-state and various sea and swell wave characteristics for waters surrounding the UK. | [↗](https://registry.opendata.aws/met-office-nws-wave/) | — | — |
| Met Office | NWS Ocean model — 2-Year Rolling Archive | The Northwest European continental shelf physical ocean model predicts temperature, salinity and circulation for waters surrounding the UK. | [↗](https://registry.opendata.aws/met-office-nws-ocean/) | — | — |
| Met Office | UK Radar Observations — 2-Year Rolling Archive | The United Kingdom Composite, radar reflectivity derived, surface rain rate estimate product in HDF5. | [↗](https://registry.opendata.aws/met-office-uk-radar-observations/) | — | — |
| Met Office | UK Land Surface Observations — 7-Day Rolling Period | Land surface weather observations for 31 parameters from over 250 locations across the Met Office UK land observation network. | [↗](https://registry.opendata.aws/met-office-uk-land-observations/) | — | — |
| Met Office | UK Marine Observations — 7-Day Rolling Period | Marine surface weather observations for 32 parameters from 69 locations across the Met Office marine observation network. | [↗](https://registry.opendata.aws/met-office-uk-marine-observations/) | — | — |
  
## JMA — Japan Meteorological Agency  

| Organisation | Dataset | Description | AWS | GEE | MPC |
| ------------ | --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | --- | --- | --- |
| JMA | Himawari-8/9 (AHI Level 1b) | Geostationary full-disk imagery of East Asia and West Pacific, every 10 min. 16 spectral bands. Archive from July 2015. Distributed by NOAA/NESDIS. | [↗](https://registry.opendata.aws/noaa-himawari/) | — | — |
  
## NOAA / NASA Joint — Reanalysis & Climate Datasets  

| Organisation | Dataset | Description | AWS | GEE | MPC |
| ---------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- | --- | --- | --- |
| NOAA / NCEP–NCAR | NCEP/NCAR Reanalysis (R1) | Atmospheric reanalysis from 1948 to present. Sea-level pressure, surface temperature, water vapour at ~2.5° resolution. | — | [↗](https://developers.google.com/earth-engine/datasets/catalog/NCEP_RE_surface_temp) | — |
| NOAA / NCEP | NCEP Climate Forecast System v2 (CFSv2) | Fully coupled atmosphere–ocean–land–sea-ice model. Seasonal forecasts and reanalysis. Hourly products from 1979 onwards. | — | [↗](https://developers.google.com/earth-engine/datasets/catalog/NOAA_CFSV2_FOR6H) | — |
| NOAA / NCEI | PERSIANN-CDR Precipitation | Long-term global daily precipitation estimates from satellite IR imagery using neural networks. 0.25° resolution from 1983 to present. | — | [↗](https://developers.google.com/earth-engine/datasets/catalog/NOAA_PERSIANN-CDR) | — |
| NOAA / NCEI | Optimum Interpolation SST v2.1 (OISST) | Daily global 0.25° sea surface temperature analysis blending AVHRR satellite and in-situ data. 1981–present. | — | [↗](https://developers.google.com/earth-engine/datasets/catalog/NOAA_CDR_OISST_V2_1) | — |
  
*Created: April 2026. This list is illustrative of open datasets available on cloud-platforms. It is not an exhaustive list.*  
  
  
  
