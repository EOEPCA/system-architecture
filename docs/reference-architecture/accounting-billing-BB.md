# Accounting and Billing BB Architecture

## Overview

The Accounting and Billing Building Block collects, generates and stores resource use data relevant to billing. Drawing from the EODH implementation, it consists of several related microservices linked by messaging - a central Accounting Service and multiple Collectors.

- **Central Accounting Service**: Manages product and price settings, serves accounting data to users
- **Resource Collectors**: Microservices that collect resource use data
- **Messaging System**: Asynchronous persistent messaging (e.g. Pulsar, Kafka)
- **Database**: Stores billing events, products and prices

### Billing Events
Record resource consumption by a particular workspace over a particular time period (typically 5 minutes, 1 hour or 1 day) and of a particular product/resource. Billing Events have UUIDs and messages with duplicate UUIDs will be ignored. Collectors generate UUIDs based on the time period, workspace and product they're generating events for.

### Resource Consumption Rate Samples
Point-in-time samples of the rate at which a particular product is being consumed by a particular workspace. The Ingester generates Billing Events from Resource Consumption Rate Samples via linear interpolation to the boundaries of one hour intervals for storage use. For all other products, exact Billing Events are generated directly by Collectors.

### Products
Consist of an SKU, a name and the units in which consumption is measured.

### Prices
Give the price of a unit of a particular product which applied between particular dates.

## Central Accounting Service

Consists of two services sharing a database and codebase - the API service and the Ingester.

### API Service
The API service serves endpoints including:
- `/api/accounting/prices` and `/api/accounting/skus` with no authentication requirement
- `/api/workspaces/{workspace}/accounting/` - requires workspace ownership or membership
- `/api/workspaces/{account}/accounting` - requires account ownership

The API Service depends on the Ingester adding data to the database but will still serve existing data without it.

### Ingester
The Ingester:
- Reads consumption data from messaging topics
- Performs UUID-based deduplication
- Generates Billing Events from Resource Consumption Rate Samples via linear interpolation to hourly boundaries (for storage only)
- Stores events in the database

The Ingester depends on Collectors to send data and the Collectors on the Ingester to store it. These are linked by asynchronous persistent messaging.

## Resource Collectors

Microservices that gather resource use data about workspaces:

### Compute Collector
Gathers CPU and memory use data about workspace namespaces. Can restart collection from X hour before its start time, relying on UUID-based deduplication at the Ingester.

### Storage Collectors
Multiple collectors for different storage types:
- **Object Storage Collector**: Samples storage use and processes access logs to track API calls and bandwidth
- **File Storage Collector**: Periodically calculates disk space use for each workspace by traversing mounted filesystems
- **Block Storage Collector**: Monitors persistent volume use

### Data Transfer Collector
Reads and processes logs. Generates bandwidth consumption events for HTTPS downloads from workspace domain names. 

## Messaging Architecture

Messages are defined with schemas to ensure compatibility. The system uses:
- Topics for different event types
- UUID-based deduplication
- Persistent messaging to handle component downtime

Collectors make use of UUID generation to prevent duplicates - generating UUIDs based on the time period, workspace and product.

## Data Flow

1. **Collection**: Collectors gather metrics from various sources
2. **Messaging**: Send Billing Events or Resource Consumption Rate Samples to topics
3. **Ingestion**: Ingester processes messages, applies deduplication
4. **Storage**: Billing Events stored in database
5. **Serving**: API queries database for accounting data

For storage metrics: Collectors send samples -> Ingester interpolates to hour boundaries -> Billing Events generated
For other metrics: Collectors generate exact Billing Events directly

## Configuration

Products and prices configured via configuration files:
```yaml
items:
  - sku: cpu-seconds
    name: CPU use by workspace pods
    unit: "s"
prices:
  - sku: cpu-seconds
    valid_from: "2025-01-02T00:00:00Z"
    price: 0.0000012
```

## Integration Points

The Building Block integrates with:
* **IAM BB/Workspace BB**: Authentication and workspace information
* **Resource Health**: Prometheus or equivalent for metrics collection
* **Log Processing**: Systems like Athena (maybe Loki?) for analysing access logs
* **Storage**: Object storage, file systems, block storage

In federated environments, supports cross-platform billing event exchange and cost reconciliation.
