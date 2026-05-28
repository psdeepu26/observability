# 📊 3 Pillars of Observability

## 🧒 Like You're 5

Imagine you're a **doctor in an ER** and a patient comes in feeling sick. You need 3 things:

1. **❤️ Vitals (Metrics)** — Heart rate, temperature, BP. Tells you *something* happened.
2. **📋 Patient History (Logs)** — Detailed timestamped records.
3. **🔬 CT Scan (Traces)** — Follows exactly where the problem is.

> **The big idea:** Metrics tell you something is wrong. Logs tell you what happened. Traces tell you *exactly where*. You need ALL THREE.

---

## 🔧 For the Engineer

### Metrics
Numeric data: Counter (total), Gauge (current), Histogram (distribution).
- Tools: Prometheus, Grafana, Thanos, CloudWatch
- PromQL: `rate(http_requests_total[5m])`

### Logs
Timestamped events. Structured JSON > plain text.
- Tools: Loki (k8s-native), ELK, Splunk, Datadog
- LogQL: `{namespace="prod"} |= "error"`

### Traces
Follow a single request across services via trace IDs.
- **Span:** Unit of work ("DB query: 40ms")
- Tools: Jaeger, Tempo, Zipkin, X-Ray

### Grafana LGTM Stack
Loki (logs) + Grafana (visualize) + Tempo (traces) + Mimir (metrics)
- **OpenTelemetry:** Unified standard for collecting all 3 signals

### USE / RED
- **USE (Infra):** Utilization, Saturation, Errors
- **RED (Services):** Rate, Errors, Duration

### Pitfall
Metrics alone is not observability. You need traces + logs to pinpoint the culprit.

---

⚡ **Micro-action:** In your Grafana, find a dashboard with only metrics. Can you trace a 500 error to a specific request and log entry?