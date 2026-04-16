# Intersight OpenTelemetry Monitoring Stack

A Docker Compose stack that collects metrics from Cisco Intersight via OpenTelemetry and visualizes them in Grafana.

## Architecture

```
Intersight API  →  intersight-otel  →  OTel Collector  →  Prometheus  →  Grafana
                                                              ↑
                                                        node-exporter
```

| Service | Description | Port |
|---------|-------------|------|
| **intersight-otel** | Polls Cisco Intersight APIs and emits OTLP metrics | - |
| **otel-collector** | OpenTelemetry Collector (contrib) — receives OTLP, exports to Prometheus | 4317 (gRPC), 4318 (HTTP) |
| **prometheus** | Scrapes metrics from the OTel Collector and node-exporter | 9090 |
| **node-exporter** | Exports host-level system metrics | 9100 |
| **grafana** | Dashboards for Intersight and system metrics | 3000 |

## Metrics Collected

### Intersight Pollers
- **VM count** — total virtual machines
- **NTP policy count**
- **Security advisories** — count and affected objects
- **Non-security advisories** — affected objects
- **Alarms** — critical and warning counts

### Intersight Time-Series Pollers
- **HyperFlex** — read/write IOPS, throughput, and latency
- **UCS Network** — utilization, TX/RX rates
- **UCS Environmentals** — fan speed, host power, host temperature

### Host Metrics (via node-exporter)
- CPU, memory, disk, and network statistics

## Prerequisites

- Docker and Docker Compose
- A Cisco Intersight account with API key credentials

## Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/<your-username>/intersight-otel.git
   cd intersight-otel
   ```

2. **Configure credentials:**
   ```bash
   cp .env.example .env
   ```
   Edit `.env` and set your `INTERSIGHT_KEY_ID`.

3. **Add your Intersight API private key:**
   ```bash
   mkdir -p secrets
   cp /path/to/your/intersight.pem secrets/intersight.pem
   ```

4. **Update the Prometheus volume path** in `docker-compose.yml` if needed:
   ```yaml
   volumes:
     prometheus-data:
       driver_opts:
         device: /your/desired/path
   ```

5. **Start the stack:**
   ```bash
   docker compose up -d
   ```

## Access

- **Grafana:** http://localhost:3000 (default credentials: admin / admin)
- **Prometheus:** http://localhost:9090
- **OTel Collector ZPages:** http://localhost:55679

## Configuration

- `config/intersight-otel.toml` — Intersight poller definitions (API queries, intervals, attributes)
- `config/otel-collector-config.yaml` — OTel Collector pipeline (receivers, processors, exporters)
- `config/prometheus.yml` — Prometheus scrape targets
- `config/grafana/dashboards/` — Pre-provisioned Grafana dashboard JSON files
- `config/grafana/datasources/` — Grafana datasource provisioning

## Credits

Built on [intersight-otel](https://github.com/cgascoig/intersight-otel) by Chris Gascoigne.
