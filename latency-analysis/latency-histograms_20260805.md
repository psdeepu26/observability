# Latency Histograms: Preserve the Shape, Not Just the Average

**Topic:** Observability  
**Category:** Latency Analysis  
**Date:** 2026-08-05

## 🧒 Like you're 5

Imagine ten children race. Nine finish in 10 seconds; one gets stuck and finishes in 100 seconds. Their average is 19 seconds. But nobody actually ran in 19 seconds.

An average crushes the whole race into one misleading number. A histogram keeps little buckets—how many finished by 10, 20, 50, or 100 seconds—so you can still see the race’s shape and spot the child who got stuck.

↓

## 🔧 For the engineer

A latency histogram counts observations in ordered buckets. From the cumulative counts, your backend can estimate percentiles and aggregate results across replicas—something client-calculated summaries usually cannot do safely.

| Instrument | Strength | Limitation |
|---|---|---|
| Average | Cheap and aggregatable | Hides tails and multimodal behavior |
| Histogram | Aggregatable distribution | Accuracy depends on bucket boundaries |
| Summary | Accurate local quantiles | Usually not aggregatable across instances |

### Bucket design is an SLO decision

Put dense boundaries around thresholds where decisions change. If your API SLO is “99% under 300 ms,” a bucket ending at `0.3` seconds gives an exact count of compliant requests. A percentile interpolated across a wide `0.25–0.5` bucket is only an estimate.

```text
# Prometheus-style cumulative histogram
http_request_duration_seconds_bucket{le="0.1"}  8200
http_request_duration_seconds_bucket{le="0.2"}  9600
http_request_duration_seconds_bucket{le="0.3"}  9905  # SLO boundary
http_request_duration_seconds_bucket{le="0.5"}  9970
http_request_duration_seconds_bucket{le="1.0"}  9990
http_request_duration_seconds_bucket{le="+Inf"} 10000
```

**Direct SLO result:** `9905 / 10000 = 99.05%` of requests met 300 ms. No percentile interpolation needed.

### Useful queries

```promql
# Approximate fleet-wide p99 over 5 minutes
histogram_quantile(
  0.99,
  sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
)

# Exact fraction satisfying the 300 ms SLO
sum(rate(http_request_duration_seconds_bucket{le="0.3"}[5m]))
/
sum(rate(http_request_duration_seconds_count[5m]))
```

| Failure | Why it hurts | Better choice |
|---|---|---|
| Only average latency | Tail failures disappear | Histogram + count + sum |
| Very wide buckets | Quantile error grows | Dense buckets near SLO |
| Every route/user as a label | Series count explodes | Bounded route templates; logs for IDs |
| Averaging p99 values | Percentiles are not additive | Aggregate buckets, then calculate p99 |

> Core rule: aggregate distributions first, calculate percentiles second.

## ⚡ Micro-action (5 minutes)

Open one production latency histogram. Find its SLO threshold, then check whether a bucket boundary lands exactly there. If not, propose one boundary at the SLO and two nearby boundaries. Also confirm no unbounded labels—request ID, user ID, raw URL—exist on the metric.
