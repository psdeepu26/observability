# 📊 Exemplars: Jump from a Metric Spike to the Exact Trace

**Observability · Exemplars** · 22 July 2026 · Wednesday

---

## 🧒 Like you're 5

Imagine a teacher sees the class average suddenly fall. The average proves
something went wrong, but not whose test explains it. An **exemplar** is a
sticky note on the chart saying, “Open this exact test.”

In observability, the chart is a metric and the test is a trace. One click
takes you from “p99 latency spiked” to one real slow request.

---

↓ Now let’s call it what it really is ↓

## 🔧 For the engineer

An exemplar attaches a representative event—usually a `trace_id` and
`span_id`—to a histogram bucket or counter sample. Aggregated metrics stay
cheap; selected requests retain a path into high-cardinality trace detail.

**Metric alert → histogram bucket → exemplar trace_id → Tempo trace**

| Layer | What it answers |
|---|---|
| **Metric** | Is impact broad? When did it start? Which service, route, or status class changed? |
| **Exemplar** | Which concrete request represents that bucket, without adding trace IDs as metric labels? |
| **Trace** | Which span consumed time? Where did retries, queueing, or downstream errors occur? |
| **Sampling** | Will the referenced trace survive? Favor errors and slow requests with tail sampling. |

### Minimal OpenTelemetry path

```python
# Application records histogram with current trace context
request_duration.record(
    duration_seconds,
    attributes={"http.route": "/checkout"},
)
```

Prometheus exposition may attach exemplar metadata to a bucket:

```text
http_server_duration_seconds_bucket{le="1.0"} 4821
# {trace_id="4bf92f3577b34da6a3ce929d0e0e4736"} 0.842
```

### Grafana correlation

```yaml
# Prometheus data source → Exemplars
Internal link: Tempo
URL label: trace_id
```

> Keep trace IDs out of normal metric labels. Otherwise every request creates
> a new time series. Metrics locate the neighborhood; exemplars provide an
> address; traces show what happened inside.

---

## ⚡ Micro-action (5 minutes)

Open one latency histogram in Grafana. Confirm its Prometheus data source has
an exemplar link targeting your trace backend with `trace_id`. If no exemplar
dots appear, verify trace context reaches metric recording and sampled traces
exist for the same time range.

---

**Observability · Exemplars** · 💫 Small steps. Every day.
