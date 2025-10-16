# Accounting and Billing BB Architecture

The Accounting and Billing Building Block collects, generates and stores resource use data relevant to billing. Based on the EODH implementation, it consists of several microservices connected by messaging - a central Accounting Service and multiple Collectors.

The system includes:
- **Central Accounting Service**: Manages product and price settings, serves accounting data to users
- **Resource Collectors**: Microservices that collect resource use data
- **Messaging System**: Asynchronous persistent messaging (e.g., Pulsar, Kafka)
- **Database**: Stores billing events, products and prices

**Billing Events** record resource consumption by a workspace over a time period (typically 5 minutes, 1 hour, or 1 day) for a product. Each event has a UUID and duplicate UUIDs are ignored. Collectors generate UUIDs from the time period, workspace and product to prevent duplication.

**Resource Consumption Rate Samples** are point-in-time samples of the rate at which a workspace consumes a product. The Ingester generates Billing Events from these samples via linear interpolation to hourly boundaries - but only for storage. Other products generate exact Billing Events directly through Collectors.

**Products** consist of an SKU, a name and units for measuring consumption. **Prices** specify the cost per unit of a product between dates.

## Central Accounting Service

This consists of two services sharing a database and codebase - the API service and the Ingester.

### API Service

The API service exposes endpoints with different authentication requirements:
- `/api/accounting/prices` and `/api/accounting/skus` - no authentication required
- `/api/workspaces/{workspace}/accounting/` - requires workspace ownership or membership

The API Service continues to serve existing data if the Ingester is unavailable, though new data won't be processed until it's restored.

### Ingester

The Ingester performs several functions:
- Reads consumption data from messaging topics
- Performs UUID-based deduplication
- Generates Billing Events from Resource Consumption Rate Samples through linear interpolation to hourly boundaries (storage metrics only)
- Stores events in the database

The Ingester depends on Collectors to send data, whilst Collectors depend on the Ingester to store it. Asynchronous persistent messaging.

## Resource Collectors

Various microservices gather resource use data about workspaces:

### Compute Collector
Gathers CPU and memory use data about workspace namespaces. When restarted, it can recover from 1 hour before its start time, relying on UUID-based deduplication at the Ingester to prevent double-counting.

### Storage Collectors
Multiple collectors handle different storage types:

**Object Storage Collector** - Samples storage use and processes access logs to track API calls and bandwidth  
**Block Storage Collector** - Monitors persistent volume use (implementation varies)

### Data Transfer Collector
Reads and processes logs to generate bandwidth consumption events for HTTPS downloads from workspace domain names.

## Messaging Architecture

The system uses messaging with defined schemas to ensure compatibility. Key features include:
- Separate topics for different event types
- UUID-based deduplication
- Persistent messaging to handle component downtime

Collectors generate UUIDs deterministically based on the time period, workspace and product, preventing duplicates without requiring complex coordination between services.

## Flow

The typical flow through the system:

1. **Collection**: Collectors gather metrics from various sources
2. **Messaging**: Send Billing Events or Resource Consumption Rate Samples to appropriate topics
3. **Ingestion**: Ingester processes messages and applies deduplication
4. **Storage**: Billing Events stored in database
5. **Serving**: API queries database for accounting data

Storage metrics follow a slightly different path - Collectors send samples, which the Ingester interpolates to hour boundaries before generating Billing Events. Other metrics generate exact Billing Events directly.

## Configuration

Products and prices are configured via YAML configuration files:

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

The SKU must match the identifier used by billing collectors. The `valid_from` field determines when new prices take effect - historical consumption retains its original pricing even if bills haven't been issued yet.

## Integration Points

The Building Block integrates with several systems:

* **IAM BB/Workspace BB**: Authentication and workspace information
* **Resource Health**: Prometheus or equivalent for metrics collection  
* **Log Processing**: Systems like Athena for analysing access logs (Loki could be an alternative)
* **Storage**: Object storage, block storage

In federated environments, the system could support cross-platform billing event exchange and cost reconciliation.