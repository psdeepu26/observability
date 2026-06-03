# 📊 The Three Pillars of Observability

**Observability** · Day #1 · 03 Jun 2026

---

## 🧒 Like You're 5

Imagine you're a doctor trying to figure out why someone feels sick. You'd want three things: **what they tell you** ("my head hurts at 3pm"), **vital signs** (temperature, heart rate on a monitor), and **the story of their day** (what they ate, where they went, who they met).

Observability in software works the same way. **Logs** are what the system tells you ("error at line 42"). **Metrics** are the vital signs (CPU at 95%, latency spiking). **Traces** are the story — they follow a single request from start to finish, through every service, so you can see exactly where it went wrong.

---

## 🔧 For the Engineer

| Pillar | What It Is | Tools |
|--------|------------|-------|
| **📝 Logs** | Timestamped, immutable records of discrete events. Structured (JSON) > unstructured. | Loki, ELK, CloudWatch |
| **📈 Metrics** | Numerical measurements over time. Counters, gauges, histograms. Cheap to store, great for alerting. | Prometheus, Grafana, Datadog |
| **🔗 Traces** | End-to-end request flow across services. Spans form a trace tree. Essential for latency debugging in microservices. | Jaeger, Zipkin, Tempo |
| **🎯 Key Insight** | No single pillar is enough. Logs without metrics = no alerting. Metrics without traces = no root cause. Traces without logs = no context. | You need all three |
| **⚙️ Your Stack** | In NVCF-land: Prometheus for metrics, Grafana for dashboards, OpenTelemetry for traces, structured JSON logs to your log aggregator. The magic is **correlation** — linking a metric spike → trace → exact log line. | NVCF observability stack |

---

## 💡 Why This Matters

When an NVCF endpoint starts returning 500s at 2am, here's how the three pillars work together: **Metrics** alert you something's wrong (error rate > 5%). **Traces** show you the failing request path — maybe it's timing out at the model inference step. **Logs** from that specific span reveal the actual error: `CUDA out of memory`. Without all three, you're guessing.

---

⚡ **Micro-action:** Open your Grafana dashboard right now. Pick any panel showing a metric (e.g., p99 latency). Ask yourself: "If this spiked right now, could I trace a single request and find the log line that explains it?" If not, that's your observability gap.
