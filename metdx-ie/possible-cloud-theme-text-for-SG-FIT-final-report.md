# SG-FIT cloud  
  
[candidate donor text for the Cloud section of the report]

## CLOUD  

### Cloud characteristics  
  
Cloud Platforms offer services for system developers to create, deploy, and operate their systems without needing to be concerned with the details of managing lower levels of the platform services they rely upon.  
  
There is no standard definition for “Cloud” or some of the other related terms. There is, however, broad agreement in the understanding and terms used when discussing Cloud platforms. The distinguishing characteristics of Cloud Platforms, when taken together, are:  
* **On-demand self-service** — Users can provision services, such as compute time or storage, as they require them using automated tools without the intervention of humans from the cloud provider.  
* **Leverage pooled resources** — The services offered by the cloud provider share a pool of resources, such as processors or storage, which are opaque to the consumers. Actual location or specific identity are abstracted away from the consumer. Capacity management, pooling and time-sharing management are the responsibility of the cloud provider to assure the cloud services’ availability and reliability.  
* **Rapid capacity elasticity** — The capacity of services can be expanded and decreased rapidly on-demand by the users, without the intervention of provider’s people. For example, additional processing power during peak periods of Web-site usage.  
* **Support multiple sandboxed systems** — Systems operating on the cloud are securely independent and non-interfering even though they share a pooled resource base.  
* **Unified access to metered services** — There are a common set of credentials for accessing all the services available to a user (one login), and the usage of those services can be monitored in near-real-time as a usage-based metered services.  
  
[**Cloud-native**: Cloud-native means that applications are built to exploit the cloud’s elasticity, automation, and distributed nature—not just hosted on virtual machines.]  

### Objects not files  
  
Three storage interfaces are in common use today, each presenting a different model to applications:

1. Block stores expose storage as a flat array of fixed-size blocks, addressed by number — the lowest-level interface, usually chosen for raw IO performance.
2. Filesystems present a hierarchical namespace of named files with POSIX-style read/write/seek semantics — the dominant interface for general-purpose software.
3. Object stores present a flat keyspace of blobs with rich metadata — designed around large, network-distributed data.

These interfaces are usually layered. ECMWF's exabyte-scale MARS archive, for example, exposes meteorological data as an object store while using filesystems and block storage underneath. For meteorological data at this scale, the object-store abstraction has proven the right choice. Modern cloud-native object stores make it easier than ever to deploy, abstracting both data access and service management, and complement elastic compute with equally elastic IO.
  
Cloud-native Object stores have the following characteristics:  
* *Scalability* — Object stores are designed to scale horizontally to exabytes of data.  
* *Durability and availability* — Data can be replicated across multiple (geographic) zones with very high reliability and durability with automated integrity checking (e.g., S3 provides “11-nines” durability).  
* *Cost effective* — Service providers may offer tiered storage (e.g., hot, cool, archive) to optimise cost based on access patterns; essentially, this means offsetting costs by accepting slower read speeds.  
* *Access via HTTP APIs* — Objects (and byte-ranges within objects) are accessed via RESTful APIs making them ideal for distributed processing, serverless workflows (e.g., Spark, Dask, ML pipelines).  
* *Event-driven integration* — Support for triggers on object creation or modification which is useful for automating workflows.   
  
In summary, publishing data as objects enables cost effective yet massively parallel querying and near-infinite scaling of durable storage. Consequently, object storage is a great choice for publishing large data volumes to a big audience. Support for byte-range queries (i.e., requesting just a part, a range of bytes, from an object) means that data users can further reduce the volume of data downloaded.  

#### Implementation Evidence  
  
[Annex 1](#annex-1.-meteorological-open-datasets-published-on-cloud-platforms) provides a list of open meteorological datasets that are published on cloud-platform object stores. This list is not intended to be exhaustive, only to illustrate that publishing data in this way is commonplace.  
  
### Proximate compute  
  
Why does the computation need to happen so close to the data?  
* Applications using or processing data need to read it into memory.  
* Data read speed is affected by network bandwidth and geographic distance.  
* Processing data close to where it is stored will improve application performance.  
* If we don’t move (all) the data – we need to compute where the data is published.  
  
### Compute in parallel, scale elastically. 

Self-service, on-demand, elastic provisioning means that:   
* You can meet demand from a large user-base; and  
* Applications can turn a time-bound problem (1 core x 1000 seconds) into a resource-bound problem (1000 cores x 1 second).  
  

  
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
  
  
  