# Http Asset Access

## Introduction

This section provides a discussion of the possible approaches for access to data assets (held in cloud storage) that have been discovered via STAC catalogue.

The approach must satisfy some basic needs expressed by utilisation domains, and we can anticipate being needed by a typical Catalog/Data Provider:

* **_Core Requirements:_**
    * Http download service - transparent to underlying storage
    * Controlled (authorized) access
    * Compatible with simple clients, including Authz, such as GDAL
    * Accessible to processing workflows with federated inputs (multiple catalogues and data storages)
* **_Additional:_**
    * Per-user rate limiting, to ensure that all users have equal access

For the core requirements some broad approaches have been identified:

* Catalogue-based
* Client-based
* Proxied

These approaches are discussed in the following sections.

## Catalog-based Approach

The Catalog-based approach provides a search result in which the asset hrefs are rewritten with pre-signed URLs that are tailored to the access of the calling user.

```plantuml source="docs/reference-architecture/puml/http-asset-access/catalogue-approach.puml"
```

**Steps:**

* User authenticates to obtain an access token (JWT)
* User queries Catalogue (STAC API), supplying their JWT
* Catalogue establishes query results, and uses JWT to provision each item asset as a pre-signed URL
* Provided asset URL can be used to download directly, or used with simple clients, such as GDAL/curl etc.
* Provided asset URL can be used by processing as input data, without additional concerns

**PROS:**

* Asset URLs transparently provide access to the data, without additional client behaviour
* Asset URLs direct to storage which avoids intermediate services with associated inefficiency
* Offers the best client experience

**CONS:**

* May over-burden the Catalogue to rewrite many URLs for each request - regardless of whether the client will follow those URLs
* May overload the Storage with many requests to generate tokens
* Assumes the Storage supports pre-signed URLs
    * Mitigated by providing a wrapper service for those that don't (maybe implies proxy)
* Requires the Catalogue to integrate with each backend Storage solution
    * Mitigated by adding a 'generate pre-signed URL' capability to the Data Gateway - that the Catalogue can then exploit

## Client-based Approach

The Client-based approach provides a search result that requires the client to follow the authorization approach for the data provider.

In the simple case the client may receive http asset URLs that can simple be consumed using the user's associated JWT. More typically, the client will have to perform more specific technique such as...

* Request pre-signed URLs for assets (client-side) on demand - which can then be used as per the _Catalog-based Approach_
* Request dedicated short-lived credentials from the storage STS (Security Token Service) service

For example, using storage STS service...

```plantuml source="docs/reference-architecture/puml/http-asset-access/client-approach.puml"
```

**Steps:**

* User authenticates to obtain an access token (JWT)
* User queries Catalogue (STAC API)
* Catalogue establishes query results - the asset URLs have **not** been personalised for the calling user
* For assets provided with http URLS...
    * Either, the client can use the URL with the JWT to consume the asset data
    * Or, the client must itself re-write the URLs to convert them to pre-signed URLs
* For assets provided with s3 URLs, the client must request short-lived credentials from the storage STS, and must act as an s3 client to access the asset data

There is an additional dimension in the context of processing workflow execution. There are two possibilities regarding the interface with the storage:

* Performed by the Processing Engine, on behalf of the running workflow - as per flow above
* Performed by the workflow job itself - as per flow below

```plantuml source="docs/reference-architecture/puml/http-asset-access/client-workflow-approach.puml"
```

**PROS:**

* Removes demand from the Catalogue

**CONS:**

* Places heavy demand on the client which must integrate with each different storage backend
* In the case that the workflow itself is the client...
    * Places a heavy integration burden for the workflow to carry
    * Hinders the capability to easily move a workflow form one platform to another

The client demand can be mitigated by specific client implementations. For example, the Microsoft Planetary Computer relies upon url-rewriting on the client - support for which has been added to `pystac` to facilitate this. Also, this solves the problem only for a specific provider and not as a general solution.

## Client-based Data Gateway Library Approach

Many of the 'cons' of the Client-based approach can be mitigated by use of the Data Gateway library as a consolidated client that handles the complexities of interfacing with the storage backend.

In particular the Data Gateway can provide abstract interfaces...

* **High-level abstraction**<br>
  High-level interfaces that provide a full abstraction to retrieve data from the STAC asset URLs - i.e. providing a 'get' operation that transparently performs any flows such as requesting pre-signed URLs or obtaining short-lived credentials from storage STS - in accordance with the approach of the specific storage backend.
* **Low-level abstraction**<br>
  Lower-level interfaces that provide an abstraction of the flows to (for example) request pre-signed URLs or STS credentials - transparent to the underlying storage backend.

**PROS:**

* Use of the Data Gateway library offers the client a generic asset access method regardless of the URL type (http/s3) and storage backend characteristics (JWT/pre-signed URL/storage STS)

**CONS:**

* Places an additional dependency on the client to use the Data Dataway - e.g. the ADES or the processing workflow
* Does not provide a solution for a browser-based client + other types of client that don't fit this approach
* Similarly, GDAL client would require development of a specific `/vsiXXX` virtual file system plugin to utilise the Data Gateway

## Proxied Approach

The Proxied approach builds upon the [`Client-based Data Gateway Library Approach`](#client-based-data-gateway-library-approach). Instead the Data Gateway provides a 'proxy' service which provides a retrieval service API that offers the high-level abstractions identified for the Data Gateway library.

Thus, the Data Gateway runs as a service proxy which transparently handles the different access protocol and authorization approaches of the storage backend.

In fact this approach is the same as that already identified in the System Architecture for the [Storage Controller](https://eoepca.readthedocs.io/projects/architecture/en/latest/reference-architecture/workspace-BB/#storage-controller) - which is derived from the requirement...

* **BR076:** The Storage Controller shall provide direct HTTP access to files in the object storage buckets – such that the assets can be retrieved without need of an S3 client – including support for HTTP Range requests.<br>
_This abstracts the underlying object storage, simplifies access and allows direct access to cloud-optimised data, such as COG/Zarr formats._

```plantuml source="docs/reference-architecture/puml/http-asset-access/proxy-approach.puml"
```

**Steps:**

* User authenticates to obtain an access token (JWT)
* User queries Data Gateway service (STAC API), supplying their JWT<br>
  _Note that this could equally be the Resource Catalogue if its records have been populated with appropriate asset links to the Data Gateway proxy_
* Data Gateway federates the offereing of one or more data providers, from which it establishes the query results - using the supplied user JWT as required
* The returned STAC asset URLs are links to the Data Gateway service
* Clients consume the http URLs to 'directly' consume the data
    * Provided asset URL can be used to download directly, or used with simple clients, such as GDAL/curl etc.
    * Provided asset URL can be used by processing as input data, without additional concerns
* The Data Gateway proxies the data requests to the provider - transparently satisfying the interfaces with the data providers

**PROS:**

* Asset URLs transparently provide access to the data, without additional client behaviour
* Offers a seamless client experience that supports all types of client, including browser, GDAL, curl, etc.

**CONS:**

* Asset URLs to storage are proxied which may introduce data transfer inefficiencies
