# celestia-da-monitoring
Repo related to node monitoring setups.
# Architecture

## A Bird's Eye View

```mermaid
graph TD
    subgraph "Celestia Consensus Node(s) (Full / Validator)"
        C[Consensus Node] -->|Node / Prometheus Metrics| D{ Sidecar Collector}
        TT[Telegraf] -->|Host Metrics / Line Protocol| D
    end
    subgraph "Celestia (DA) Node(s)"
        A[Bridge Node] -->|Node / OTLP Metrics| B{Sidecar Collector}
        T[Telegraf] -->|Host Metrics / Line Protocol| B
    end
    
    subgraph " "
        I((Central Collector))
    end
    subgraph " "
        F((Foundation's Collector))
    end
    
    subgraph "Backends"
        PP[(Prometheus)]
        J[(Victoria Metrics)]
        K[(Clickhouse)]
        L[(Influx)]
    end
    subgraph "Monitoring"
        U>Grafana]
        AA>Alert Manager]
    end
    D --> |Node + Host / OTLP Metrics| I
    B --> |Node + Host / OTLP Metrics| I
    B --> |Node / OTLP Metrics| F
    I -.-> |Prometheus Metrics| PP
    I --> |Prometheus Metrics| J
    I -.-> |OTLP Metrics| K
    I -.-> |Line Protocol| L
    J --> U
    J --> AA
    
```
## High-Level Components

### OpenTelemetry Collector
- Streamlines the collection and delivery of metrics.

### Telegraf Agent
- Collect's host-level metrics and run additional scripts to derive specific metrics.

### Sidecar Collector
- Run's the OpenTelemetry collector on the same machine as the node.

### Central Collector
- Handle's the aggregation of metrics from multiple sidecar collectors.
- Ideally deployed behind a load balancer to distribute the load and for HA.

## Protocols

### OTLP (OpenTelemetry Protocol)
- Native protocol for OpenTelemetry metrics.

### Line Protocol
- Native protocol for Telegraf Agent, also used by InfluxDB.

### Prometheus Format
- Standard protocol for Prometheus metrics.
- Metrics are typically exposed via the `/metrics` endpoint
