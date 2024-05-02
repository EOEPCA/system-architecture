# Data Gateway Building Block

## Overview

The Data Gateway building block provides a consolidated and consistent capability for accessing the data offering of an extensible set of data providers and datasets.

Within the architecture there are a number of points of interaction with data sources for discovery and access to datasets and data products. Each data source has its own interface requirements, including authentication, metadata search, data retrieval, etc.

Rather than implement the logic required for specific data source interaction in each BB where it is required - instead the Data Gateway provides a consolidated implementation that all platform services can exploit as they require. In this way, the interface with each data provider is only implemented once, and there is only one implementation that requires maintenenance in the case that the data provider interface evolves.

### Capabilities

To support the needs of the dependent building blocks, the Data Gateway provides the following capabilities:

* Discover data providers
* Discover product types
* Search products - by type and temporal/spatial search criteria
* Retrieve products (raw assets)
* Retrieve xarray.DataArray representation

See section [Dependent Building Blocks][dependent-building-blocks] - which elaborates the usage of the Data Gateway by other building blocks.

### Extensible

To support an ever growing set of data sources, the Data Gateway must be extensible for:

* Data providers and datasets
* Authentication approach
* Search interface protocol/standard
* Data retrieval approach/standard

## Approach

The Data Gateway compiles an extensible capability of integrated data source offerings.

### Embeddable

This capability is designed to be embeddable/invocable from a number of other buildings, including:

* Resource Registration Harvester component<br>
  *For harvesting and registration of data products*

* Processing Engine<br>
  *For preparation and access to input data for processing workflows.*

* Data Access<br>
  *For retrieval of dataset assets into the services for data visualisation and access.*

* Data Cube Access<br>
  *For discovery and access to multi-dimensional data, including for consumption as xarray.DataArray.*

* Federated Orchestrator<br>
  *For discovery and access to datasets across a federated set of platforms and data providers.*

To support inclusion in the capabilities these BBs, the Data Gateway shall provide a programmtic library (at least python) to support inline integration.

### Dataset Representation

#### STAC

The Data Gateway presents the offering of each dataset as STAC items - regardless of underlying technology capability of the data provider itself.

Thus, the Data Gateway is able to present a standard STAC interface for each dataset - which provides a simplified approach for the other building blocks, that can use STAC semantics regardless of the data source.

#### Data Cube

The Data Gateway should also provide a data cube semantic, by offering a dataset representation as an `xarray.DataArray`.

This facilitates, for example, integration of the data layer with pangeo stack.

## Dependent Building Blocks

This section introduces the other building blocks that are expected to utilsie the Data Gateway building block.
