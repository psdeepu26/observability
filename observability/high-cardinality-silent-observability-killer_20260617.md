# 📊 High Cardinality: The Silent Observability Killer

**Observability** · Day #3 · June 17, 2026

---

## 🧒 Like You're 5

Imagine you have a giant notebook where you write down everything that happens at your
lemonade stand — every customer, every cup, every ice cube. Now imagine you also write
down the customer's name, their shoe size, their dog's name, and what they had for
breakfast. Your notebook explodes. It gets so fat you can't find anything, and it costs
a fortune to keep buying new notebooks.

That's what happens in monitoring when you attach too many unique labels to your metrics.
Each unique combination of labels creates a new "time series" — a separate row in the
database. A single metric like `http_requests_total` can silently spawn *millions* of
time series if you tag it with things like `user_id` or `request_id`. Your monitoring
system chokes, queries slow to a crawl, and your observability bill skyrockets — all
without a single alert firing.

---

## 🔧 For the Engineer

| Concept | What It Is |
|---------|------------|
| **Cardinality** | The number of unique time series a metric produces. Each unique label combination = 1 series. `http_requests{method,status}` with 5 methods x 3 statuses = 15 series. |
| **High Cardinality** | When labels like `user_id`, `request_id`, or `trace_id` create unbounded unique values. A metric with `user_id` across 10M users = 10M series from one metric. |
| **Active Series** | Time series that receive at least one sample in the last ~5 minutes. Prometheus charges by active series. High cardinality = more active series = higher cost + slower queries. |
| **Label vs Sample** | Labels are metadata keys on a metric. Samples are actual data points. High cardinality is about *label explosion*, not sample volume — you can have few samples but millions of series. |
| **Recording Rules** | Pre-aggregate high-cardinality data into lower-cardinality summaries. `sum by (service) (rate(http_requests_total[5m]))` collapses millions of series into dozens. |
| **Native Histograms** | Prometheus Native Histograms encode bucket boundaries once in the metric definition, not as `le` labels. Reduces cardinality of histogram metrics by 10-100x. |
| **NVCF + Prometheus** | In NVCF observability, every inference request could carry `model_name`, `gpu_type`, `tenant_id`, `region`. That's 4 dimensions — manageable. But adding `request_id` or `trace_id` as a label would be catastrophic. Use **exemplars** to link metrics to traces without inflating cardinality. |

---

## 💡 Why It Matters

Cardinality explosions are the #1 silent killer in production monitoring. They don't
trigger alerts — they just make everything slower and more expensive until one day your
dashboards take 30 seconds to load and your Prometheus OOMs at 3am. The insidious part:
the metric *looks* fine in the config. It's only at scale that the combinatorial
explosion hits. Teams have burned through $50K/month in observability bills because
someone added `customer_email` as a label to a high-throughput metric.

---

⚡ **Micro-action:** Run this PromQL query right now:
`count by (__name__)({__name__!=""})` — it shows your top metrics by cardinality
count. Sort by value and check if any metric has unexpectedly high series counts.
If you see a metric with >10K series, investigate its labels.
