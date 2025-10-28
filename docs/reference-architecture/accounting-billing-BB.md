# Accounting and Billing BB Architecture

> This is only a potential architecture of how this could work based on how the Earth Observation Data Hub (EODH) does it.

The Accounting and Billing Building Block collects resource usage data and prepares it for billing. Based on the EODH implementation, it's built around a central Accounting Service and multiple Collectors, all connected via messaging.

## Main components

- **Central Accounting Service** - handles product/price config and serves accounting data
- **Resource Collectors** - microservices that collect usage data from different sources
- **Messaging System** - async persistent messaging (Pulsar, Kafka, etc.)
- **Database** - stores billing events, products and prices

## Terminology

**Billing Events** capture how much of a product a workspace consumed during a time window (usually 5 minutes/1 hour/1 day). Each event gets a UUID and duplicates are ignored. Collectors generate these UUIDs from the time period + workspace + product.

**Resource Consumption Rate Samples** are point-in-time snapshots of consumption rate. The Ingester converts these into Billing Events using linear interpolation to hourly boundaries - but this only happens for storage metrics. Everything else goes straight to exact Billing Events via Collectors.

**Products** have an SKU, name and units.

**Prices** define cost per unit for a date range.

## Central Accounting Service

Two services share a database and codebase: the API service and the Ingester.

### API Service

Exposes endpoints with different auth requirements:
- `/api/accounting/prices` and `/api/accounting/skus` - public, no auth
- `/api/workspaces/{workspace}/accounting/` - need workspace ownership or membership

If the Ingester goes down, the API keeps serving existing data. New data won't get processed until the Ingester comes back.

### Ingester

Does a few things:
- Reads consumption data from message topics
- Deduplicates using UUIDs
- Converts Resource Consumption Rate Samples into Billing Events (storage only - uses linear interpolation to hour boundaries)
- Writes events to the database

The Ingester needs Collectors to send data. Collectors need the Ingester to store it. The async messaging between them handles this circular dependency.

## Resource Collectors

Different microservices pull usage data from different sources:

### Compute Collector
Tracks CPU and memory usage for workspace namespaces. After a restart, it can backfill data from up to 1 hour before it came back online. UUID deduplication at the Ingester prevents double-counting.

### Storage Collectors
We have multiple collectors for different storage types:

* **Object Storage Collector**<br>
  Samples storage usage and parses access logs for API calls and bandwidth  
* **Block Storage Collector**<br>
  Monitors persistent volumes (implementation depends on your setup)

### Data Transfer Collector
Parses logs to generate bandwidth events for HTTPS downloads from workspace domains.

## Messaging Architecture

The messaging layer uses defined schemas for compatibility. Key bits:
- Different topics for different event types
- UUID-based deduplication
- Persistent messaging so downtime doesn't lose data

Collectors generate UUIDs deterministically (time period + workspace + product), so there's no duplicate coordination needed between services.

## Flow

1. **Collection** - Collectors gather metrics
2. **Messaging** - Send Billing Events or Resource Consumption Rate Samples to topics
3. **Ingestion** - Ingester processes messages and deduplicates
4. **Storage** - Billing Events go into the database
5. **Serving** - API queries the database

Storage metrics work slightly differently: Collectors send samples, Ingester interpolates to hour boundaries, then generates Billing Events. Everything else skips the interpolation step.

## Configuration

Products and prices come from YAML config files:

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

The SKU has to match what the billing collectors use. The `valid_from` date controls when new prices. Historical consumption keeps its original pricing even if you haven't generated bills yet.

## Integration Points

This building block connects to:

* **IAM BB/Workspace BB** - auth and workspace info
* **Resource Health** - Prometheus or similar for metrics
* **Log Processing** - systems like Athena for log analysis (Loki may work too?)
* **Storage** - object storage, block storage systems

In federated setups, you could extend this to handle cross-platform billing event exchange and cost reconciliation.