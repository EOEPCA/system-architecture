# Http Asset Access

This section provides a discussion of the possible approaches for access to data assets (held in cloud storage) that have been discovered via STAC catalogue.

The approach must satisfy some basic needs expressed by utilisation domains, and we can anticipate being needed by a typical Catalog/Data Provider:

* Core Requirements:
  * Http download service - transparent to underlying storage
  * Controlled (authorized) access
  * Compatible with simple clients, including Authz, such as GDAL
  * Accessible to processing workflows with federated inputs (multiple catalogues and data storages)
* Additional:
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

## Client-based Approach

TBD

## Proxied Approach

TBD

