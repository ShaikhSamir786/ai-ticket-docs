# 📊 AI Customer Ticket Triage - Observability Guide

This document outlines how to implement end-to-end observability across the ticket triage microservice ecosystem using the **PLGT stack** (Prometheus, Loki, Grafana, Tempo) + **OpenTelemetry**.

---

## 🧩 Stack Overview

| Component | Purpose | Integration Point |
|-----------|---------|-------------------|
| **Prometheus** | Metrics collection & alerting | Scrapes `/metrics` endpoints from each service |
| **Loki** | Log aggregation | Ingests structured JSON logs via Promtail or Docker driver |
| **Tempo** | Distributed tracing | Receives OTLP traces over HTTP/gRPC |
| **Grafana** | Unified UI & alerting | Visualizes metrics, logs, traces in correlated views |
| **OpenTelemetry SDK** | Auto-instrumentation | Traces HTTP, DB, Redis, LLM calls, propagates context |

---

## 🐳 Local Setup

### 1. Add to `docker-compose.yml`
```yaml
# docker-compose.observability.yml
version: "3.9"
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./observability/prometheus.yml:/etc/prometheus/prometheus.yml
    ports: ["9090:9090"]

  loki:
    image: grafana/loki:latest
    ports: ["3100:3100"]
    command: -config.file=/etc/loki/local-config.yaml

  tempo:
    image: grafana/tempo:latest
    command: ["-config.file=/etc/tempo.yaml"]
    volumes:
      - ./observability/tempo.yaml:/etc/tempo.yaml
      - tempo_data:/tmp/tempo
    ports: ["3200:3200", "4317:4317", "4318:4318"]

  grafana:
    image: grafana/grafana:latest
    environment:
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    volumes:
      - ./observability/grafana/provisioning:/etc/grafana/provisioning
    ports: ["3000:3000"]
    depends_on: [prometheus, loki, tempo]

volumes:
  tempo_
```

### 2. Prometheus Config (`observability/prometheus.yml`)
```yaml
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: "ai-ticket-services"
    static_configs:
      - targets:
          - "host.docker.internal:9091" # main-server
          - "host.docker.internal:9092" # core-server
          - "host.docker.internal:9093" # processor-server
          - "host.docker.internal:9094" # llm-server
          - "host.docker.internal:9095" # scheduler-server
```

### 3. Grafana Datasource Provisioning (`observability/grafana/provisioning/datasources/ds.yml`)
```yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    isDefault: true
  - name: Loki
    type: loki
    url: http://loki:3100
  - name: Tempo
    type: tempo
    url: http://tempo:3200
```

---

## 🔌 Service Instrumentation

### A. Metrics (`prom-client`)
```bash
npm install prom-client
```
```ts
// src/observability/metrics.ts
import client from 'prom-client';
const register = new client.Registry();

client.collectDefaultMetrics({ register });

export const ticketTriageDuration = new client.Histogram({
  name: 'ticket_triage_duration_seconds',
  help: 'Time from ingestion to AI triage completion',
  labelNames: ['service', 'model', 'status'],
  registers: [register],
});

export const llmTokensUsed = new client.Counter({
  name: 'llm_tokens_used_total',
  help: 'Total input + output tokens sent to LLM providers',
  labelNames: ['model', 'provider'],
  registers: [register],
});

// Expose endpoint in Express/NestJS
app.get('/metrics', async (_req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});
```

### B. Distributed Tracing (OpenTelemetry)
```bash
npm install @opentelemetry/api @opentelemetry/sdk-node \
  @opentelemetry/auto-instrumentations-node @opentelemetry/exporter-trace-otlp-http
```
```ts
// src/observability/trace.ts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_TRACES_ENDPOINT || 'http://tempo:4318/v1/traces'
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();
process.on('SIGTERM', () => sdk.shutdown());
```
✅ Auto-instrumentation captures HTTP, PostgreSQL, Redis, and propagates `traceparent` headers across services.

### C. Structured Logging + Trace Correlation
```bash
npm install winston
```
```ts
// src/observability/logger.ts
import winston from 'winston';

export const logger = winston.createLogger({
  level: 'info',
  defaultMeta: { service: process.env.SERVICE_NAME },
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
    winston.format((info) => {
      // Trace correlation (if trace data is attached by OTel)
      if (info.trace) {
        info.trace_id = info.trace.trace_id;
        info.span_id = info.trace.span_id;
      }
      return info;
    })()
  ),
  transports: [new winston.transports.Console()],
});
```
Logs will contain `trace_id` & `span_id`, enabling Grafana's **Logs → Traces** correlation.

---

## 📈 Domain-Specific Metrics to Track

| Service | Key Metrics |
|---------|-------------|
| `main-server` | Request rate, `http_request_duration_seconds`, 4xx/5xx %, auth failures |
| `core-server` | DB connection pool usage, query latency, ticket CRUD ops/sec |
| `processor-server` | Triage success/fail rate, confidence score histogram, override rate |
| `llm-server` | `llm_tokens_used_total`, cost per request, model fallback triggers, LLM latency |
| `scheduler-server` | BullMQ queue depth, job retry count, DLQ size, SLA breach rate |

---

## 🌐 Tracing & Business Context Propagation

Use OTel Baggage to inject business metadata into traces:
```ts
import { propagation, trace } from '@opentelemetry/api';

const ctx = propagation.setBaggage(propagation.getBaggage(activeSpan.spanContext()), 
  new Baggage().setEntry('ticket_id', { value: ticketId })
                 .setEntry('priority', { value: priority })
                 .setEntry('agent_id', { value: agentId })
);

trace.setSpanContext(ctx, activeSpan.spanContext());
```
This lets you filter traces in Tempo by `ticket_id` or `priority` without touching payload data.

---

## 🖥️ Grafana Dashboards

1. **Import Community Dashboards**
   - Node.js App: `11159`
   - PostgreSQL: `9628`
   - Redis: `763`
   - BullMQ Monitor: Search Grafana dashboard hub for `BullMQ`

2. **Custom Triage Dashboard Queries**
   ```promql
   # Average triage latency by model
   rate(ticket_triage_duration_seconds_sum[5m]) / rate(ticket_triage_duration_seconds_count[5m])

   # LLM token burn rate
   sum(rate(llm_tokens_used_total[1m])) by (model)

   # Queue backlog + failures
   bullmq_active_jobs + bullmq_failed_jobs

   # SLA breach rate (tickets > 15m untriaged)
   count(sla_breached_total > 0)
   ```

3. **Correlate Logs & Traces**
   - Use Grafana's **Explore** → select Loki → filter by `trace_id` → click **View in Tempo**

---

## 🚨 Alerting & SLOs (Prometheus/Grafana Rules)

```yaml
# observability/alerts.yml
groups:
  - name: ticket-triage-slos
    rules:
      - alert: HighTriageLatency
        expr: rate(ticket_triage_duration_seconds_sum[5m]) / rate(ticket_triage_duration_seconds_count[5m]) > 5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "AI triage latency > 5s for 5m"

      - alert: LLMFallbackTriggered
        expr: increase(llm_fallback_total[10m]) > 3
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "LLM fallback triggered > 3 times in 10m"
```

---

## ⚠️ Best Practices & Pitfalls

| ✅ Do | ❌ Don't |
|------|----------|
| Use correlation IDs from the gateway and pass them downstream | Hardcode service names, IPs, or ports in configs |
| Sample traces in prod (`0.05–0.1`) to avoid storage bloat | Log PII, customer emails, or raw ticket content |
| Alert on SLOs & business impact (latency, queue depth, SLA breaches) | Alert on raw CPU/memory without context |
| Keep metric cardinality low (avoid `user_id`, `ticket_id` as labels) | Store metrics in DB/Redis (use Prometheus) |

---

## 🛠️ Quick Start Commands

```bash
# Start observability stack
docker compose -f docker-compose.observability.yml up -d

# View Grafana
open http://localhost:3000

# Check Prometheus targets
open http://localhost:9090/targets

# Verify traces in Tempo UI (or via Grafana Explore)
open http://localhost:3200
```

---

## 📦 Next Steps
- [ ] Generate `observability/` folder with all config files
- [ ] Scaffold instrumentation boilerplate for `ai-ticket-llm-server`
- [ ] Create a pre-built Grafana JSON dashboard for ticket triage KPIs
- [ ] Add Docker logging driver config for automatic Loki ingestion

Reply with the next item you want, or let me know if you'd like this exported as a ready-to-commit `docs/observability.md` file.