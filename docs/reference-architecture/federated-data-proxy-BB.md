# Federated Data Proxy Architecture

## Overview

The Federated Data Proxy allows a platform to serve data from multiple (external) providers, in such a way that is transparent to the service end-users.

The platform offers a STAC Catalogue to its users that enumerates all the available (online and offline) datasets, including those that are hosted in external data providers. In this case there are a range of approaches for handling access to the external data, that must be taken into account:

1. Data is routinely harvested and re-hosted
2. Selected data is routinely harvested according to a criteria<br>
   _e.g. rolling last N months data over a geographic region_
3. Data is retrieved on-demand
4. Data is ordered for subsequent asynchronous access<br>
   _This includes requests for data from Long-term Archive_
5. On-demand data is maintained in a online cache

The Federated Data Proxy offers a single point of access that transparanerly satisfies each of these data management approaches.

## Approach

The Federated Data Proxy provides an 'access middleware' endpoint that transparently interfaces with each upstream data provider - including both internal and external data sources.

The STAC Calalogue maintains asset `href` that resolve to the Federated Data Proxy endpoint - rather than to the external data provider. 

The user searches the STAC Catalogue to discover data of interest, and then follows the provided assest urls to request data retrieval. These requests go via the Federated Data Proxy which integrates with the data provider to satisfy the request.

The endpoints exposed by the Federated Data Proxy identify the datasets and associated assets, such that the request can be marshalled to the appropriate upstream provider.

The Federated Data Proxy allows the data management approach of each dataset to be independently configured, including:

* Harvesting approach and cadence
* Caching policy

## Cache

The Federated Data Proxy maintains an online cache to minimise the overhead of retrieving data from the upstream provider. Requests are delivered from the cache. In the case of a cache miss, then the requested data will be retrieved by the Federated Data Proxy into the cache, from where it can be delivered to the caller.

The caching strategy should be configurable - such that the cache can be tuned both gloablly and per dataset:

* TBD - see TRUTHS

## Asynchronous Retrieval

> Use of `Retry-After` HTTP header.
>
> Clients must be able to handle this - e.g. GDAL - check this is supported

## Data Gateway - Possible Usage

The Federated Data Proxy may be able to use the _Data Access Gateway_ building-block to support its implementation, including:

* Use of EODAG python library to interface with each data provider
* Evolution of the EODAG server to meet the 'access middleware' requirements
