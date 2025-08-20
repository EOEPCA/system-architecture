# User Support BB Architecture

> Notes...
> 
> * Client Lib + CLI + EOEPCA-UI
>     * Modular design - i.e. plugin support for each BB
>         * with coherent submodules - auth, catalog, processing, workspace, etc.
> 
> * User client library (similar to terrapi)
>     * Make it easy to work with EOEPCA services for authentication, start/monitor processing jobs, start/monitor data registrations, create > workspaces, upload files to workspace
>     * Support for public/private STAC collections
>     * Device flow for authentication
> 
> * CLI - using client library
> 
> * User Portal

## Overview

The User Support BB helps to drive platform adoption by providing a user-friendly interface and tools for users to interact with EOEPCA services. The BB comprises the following parts:

* Python client library<br>
  _Other languages can be added in the future, as needed._
* Command Line Interface (CLI)<br>
  _Relies upon the Python client library for functionality._
* User Portal (UI)<br>
  _Provides a web-based interface for users to interact with EOEPCA services._

## Python Client Library

TBD

## Command Line Interface (CLI)

TBD

## User Portal (UI)

The User Portal provides a web-based interface for users to interact with EOEPCA services. It is designed to provide a consolidated user-experience that seamlessly combines the capabilities of the EOEPCA building blocks - for example integration of data discovery with data processing.

The User Portal should follow a modular design, that provides a core capability that acts as a framework into which building block UI 'modules' can be integrated. Each module relies upon the user authentication and session management that is handled by the UI core. This allows for a coherent user experience across the different building blocks, while also allowing for flexibility in the design and implementation of each module.

Thus, we envision the following modular capabilitiies:

* [User Portal Core](#user-portal-core)<br>
  _Core user interface and framework for integrating building block UI modules._

* [Catalogue UI](#catalogue-ui)<br>
  _Discovery and exploration of data collections and resources._

* [Processing UI](#processing-ui)<br>
  _Creation and management of processing jobs_

* [Analysis UI](#processing-ui)<br>
  _Platform hosted interactive notebooks_

* [Workspace UI](#workspace-ui)<br>
  _Creation, management and usage of workspaces for users, groups and projects_

* [Machine Learning UI](#machine-learning-ui)<br>
  _Creation, management and usage of machine learning models and training data_

* [Automations UI](#automations-ui)<br>
  _Creation, management and monitoring of automated platform tasks_

### User Portal Core

Provides the umbrella UI through which all other modules are accessed.

* Handles user authentication and session management
* Provide user profile management
* Organises navigation and access to other UI modules
* Included modules should plugin through configuration
* Provides a consistent look-and-feel across all modules

### Catalogue UI

The Catalogue UI should be seen as part of the [Resource Discovery BB](resource-discovery-BB.md). In lieu of this, we will briefly elaborate here.

The Catalogue UI provides a user-friendly interface for users to discover and explore data collections and resources available on the EOEPCA platform. Thus, the UI serve two broad catagories:

  * Data: collections, datasets, etc.
  * Resources: Jupyter notebooks, workflows, etc.

**Data**

The existing STAC Browser is quite technical for new users. A more user-friendly interface is needed that allows users to search, filter, browse and visualise data - with better support for collections:

* Grouping of collections
* Supporting links - e.g. to Jupyter notebooks demonstrating how to use the collections
* Support for public/private STAC collections

The suggestion is to reuse the existing DevelopmentSeed solution, which is already used by the Planetary Computer and EODC:

* **Planetary Computer:**
    * Catalog - [https://planetarycomputer.microsoft.com/catalog](https://planetarycomputer.microsoft.com/catalog)
    * Explore - [https://planetarycomputer.microsoft.com/explore](https://planetarycomputer.microsoft.com/explore)
* **EODC:**
    * Catalog - [https://portal.services.eodc.eu/data_catalogue](https://portal.services.eodc.eu/data_catalogue)
    * Explore - [https://portal.services.eodc.eu/explore](https://portal.services.eodc.eu/explore)

**Resources**

Resources includes other discoverable non-data items, such as Jupyter notebooks, workflows, etc. that can be used with the data collections.

The existing [Open Science Catalogue UI](https://opensciencedata.esa.int/) may provide a good starting point for this.

### Data Management UI

The Data Management UI provides control and visibility over the registration and harvesting of data collections in the platform:

* Registration of data collections
* Starting and monitoring harvesting jobs

> It may be pragmatic to integrate this UI with the Catalogue UI, as it is primarily focused on data collections.

### Processing UI

The Processing UI faciltates the discovery and execution of processing workflows on the platform.

Capabilities include:

* Discovery of processing workflows (search and filter)
* Integration with Catalogue UI to establish a link between data discovery and processing execution
* Start/monitor processing jobs
* Manage and visualise workflow outputs
* Create workflows interactively
* Validate workflow application quality
* Publish and share workflows

### Analysis UI

Provision of interactive notebooks, hosted on the platform, with access to the platform's data and services.

The capabilities of the Analysis UI are provided by the [Application Hub BB](application-hub-BB.md).

### Workspace UI

The Workspace UI provides both Administrative and End-user capabilities for managing and exploiting workspaces.

> Note that some of these capabilities are already provided by the [Workspace BB](workspace-BB.md). The capabilties described here can be regarded as an extention to the UI of the existing Workspace BB.

**Workspace Administration**

* Create and manage workspace lifecycles
* Manage workspace users, authorization and sharing
* Bucket management

**Workspace End-user**

* Access and management of files in workspace buckets
* Sharing of workspace files
* Workspace vCluster management
* Workspace service management
* Exposing workspace services for public access

### Machine Learning UI

The capabilities of the Machine Learning UI are provided by the [MLOps BB](mlops-BB.md).

### Automations UI

The Automations UI should be seen as part of the [Notification & Automation BB](notification-automation-BB.md). In lieu of this, we will briefly elaborate here.

Capabilities include:

* Create and manage automations
* Define triggers<br>
  _e.g. watching a linked data collection_
* Define actions linked to triggers<br>
  _e.g. trigger processing jobs based on data availability, or run scheduled tasks_
* Monitor execution of automations
