# Metrics & Alerting: RED vs USE Method

> **Observability · Nightly Insight** · June 17, 2026 (Wednesday)

---

## 🧒 Like you're 5

Imagine you run a pizza shop. **RED** is like watching the customers: *How many orders came in? How fast did we deliver? How many pizzas came back?* **USE** is like watching the kitchen: *Is the oven too hot? Is the fridge full? Is the dough-roller jammed?*

You need *both* views. Customers complaining (RED) tells you **something is wrong**. Kitchen metrics (USE) tells you **what is wrong**. Together, they're your observability superpower.

---

## 🔧 For the engineer

### Two complementary frameworks for full-stack observability

#### 🔴 RED Method — Services

Designed for microservices & APIs. Top-down, user-centric.

- **R**ate — Requests per second
- **E**rrors — Failed requests / sec (HTTP 5xx)
- **D**uration — Latency percentiles (p50, p95, p99)

```promql
# Prometheus RED metrics example
http_requests_total{service="api-gateway", status="500"}
http_request_duration_seconds{quantile="0.99"}
```

#### 🟢 USE Method — Resources

Designed for infrastructure & hardware. Bottom-up, resource-centric.

- **U**tilization — % of time resource is busy
- **S**aturation — Queue length / wait time
- **E**rrors — Error count (correctable & uncorrectable)

```promql
# Node Exporter USE metrics
node_cpu_seconds_total{mode="iowait"}
node_memory_MemAvailable_bytes
node_disk_io_time_weighted_seconds_total
```

### When to use which?

| Scenario | RED | USE |
|---|---|---|
| API latency spike | ✅ Duration metric | |
| Database connection pool exhausted | ✅ Errors going up | ✅ Saturation on pool |
| Disk I/O bottleneck | | ✅ Utilization + Saturation |
| NVCF function cold start | ✅ Duration (p99) | ✅ GPU memory saturation |
| K8s node under pressure | | ✅ CPU/Mem/Disk USE |
| User-facing error rate climbing | ✅ Rate + Errors | |

### Alerting on RED + USE Together

```yaml
# Example: Alert when API errors spike AND DB saturation is high
# This is more actionable than either alone

# Prometheus alert rule
- alert: HighErrorRateWithDBSaturation
  expr: |
    (
      rate(http_requests_total{status=~"5.."}[5m])
      / rate(http_requests_total[5m])
    ) > 0.05
    and on(instance)
    node_disk_io_time_weighted_seconds_total / node_disk_io_time_seconds_total > 0.8
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "API errors + disk saturation on {{ $labels.instance }}"
```

---

## ⚡ Micro-action

**Audit your alerts: RED-only or USE-only?**

Open your alerting rules today. For every RED alert (errors/duration), check if there's a corresponding USE metric on the same resource. If not, add one. The combination cuts MTTR dramatically — you'll know *what's broken* AND *why it's broken* in a single alert.

---

*Tags: #observability #metrics #alerting #red-method #use-method #prometheus #sre*

*🦉 OWL · Nightly Learning Insight · psdeepu26/observability*
