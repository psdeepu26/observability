# 📊 Metric Temporality: Delta vs Cumulative

**Observability · Metric Semantics** · 29 July 2026 · Wednesday

---

## 🧒 Like you're 5

Imagine tracking rain. One notebook says, “It rained **3 mm during this hour**.” Another says, “It has rained **27 mm since Monday**.” Both describe the same rain, but each number covers a different window.

Metrics work the same way. A **delta** reports only what changed since the previous report. A **cumulative** value reports everything since a fixed starting point.

---

↓ Now let’s call it what it really is ↓

## 🔧 For the engineer

**Temporality** defines the time range represented by each aggregated metric point. OpenTelemetry sums and histograms can use delta or cumulative temporality; the choice changes exporter state, reset handling, and backend compatibility—not the underlying measurements.

| Mode | Meaning and trade-off |
|---|---|
| **Delta** | Each point covers `(previous export, current export]`. Good for systems that naturally consume increments. |
| **Cumulative** | Each point covers `(start time, current export]`. Natural fit for Prometheus counters and `rate()`. |
| **Reset signal** | A changed start timestamp distinguishes a process reset from a lower delta. Losing it can create false spikes. |
| **Conversion** | Delta → cumulative stores per-series totals. Cumulative → delta stores previous values. Restarts or dropped points can break continuity. |

**Measurements → SDK aggregation → temporality → exporter/backend**

### Same traffic, different points

```text
# Requests in three one-minute windows: 10, 15, 8

Delta:       10, 15,  8
Cumulative:  10, 25, 33
```

Prometheus expects cumulative counter samples:

```promql
rate(http_requests_total[5m])
```

### Collector policy example

```yaml
exporters:
  otlphttp/metrics_backend:
    endpoint: https://metrics.example/v1/metrics

# Temporality is usually selected by SDK/exporter capabilities.
# Verify the chosen exporter and backend agree before conversion.
```

> **Failure pattern:** sending delta values to a backend that assumes cumulative counters makes every export look like a reset. Sending cumulative values to a backend that assumes deltas repeatedly counts old work. Match semantics end to end; do not infer them from metric names.

---

## ⚡ Micro-action (5 minutes)

Trace one counter end to end. Pick `http_requests_total`. Check its OpenTelemetry SDK/exporter temporality, inspect three raw backend samples, and confirm they rise cumulatively or represent clean interval deltas as expected. Record the contract beside the exporter config.

---

**Observability · Metric Semantics** · 💫 Small steps. Every day.
