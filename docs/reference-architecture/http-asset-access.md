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
* Offers the best client experience

**CONS:**

* May over-burden the Catalogue to rewrite many URLs for each request - regardless of whether the client will follow those URLs
* May overload the Storage with many requests to generate tokens
* Assumes the Storage supports pre-signed URLs
    * Mitigated by providing a wrapper service for those that don't (maybe implies proxy)
* Requires the Catalogue to integrate with each backend Storage solution
    * Mitigated by adding a 'generate pre-signed URL' capability to the Data Gateway - that the Catalogue can then exploit

## Client-based Approach

TBD

## Proxied Approach

TBD

