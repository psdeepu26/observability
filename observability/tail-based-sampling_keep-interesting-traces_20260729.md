# Tail-Based Sampling: Keep the Interesting Traces

**Topic:** Observability  
**Date:** 2026-07-29  
**Category:** Tracing

## 🧒 Like you're 5

Imagine a teacher watching every child run through an obstacle course. Instead of deciding at the entrance which runs to record, the teacher waits until each run is finished. Then she saves the runs where someone fell, took unusually long, or needed extra help—and keeps only a few ordinary runs. That is tail-based sampling: decide what to keep after seeing the whole trace.

↓ Now let's call it what it really is ↓

## 🔧 For the engineer

Tail-based sampling makes a sampling decision after spans for a trace have arrived, so the collector can preserve high-value evidence such as errors, latency outliers, and selected business events. Head sampling is cheaper and simpler, but it can discard a slow or failed trace before its outcome is known.

### Key design points

- **Decision signal:** Evaluate the complete trace: status code, duration, attributes, span count, or parent service.
- **Collector role:** All spans for a trace must reach the same tail-sampling processor. Scale it with trace-aware routing, not arbitrary load balancing.
- **Trade-off:** More retained signal and better debugging, but higher memory, network, and operational cost while traces wait for a decision.
- **Safe default:** Keep errors and latency outliers at a high rate; sample healthy, fast traces at a lower baseline rate.

```yaml
policies:
  - name: keep-errors
    type: status_code
    status_code: {status_codes: [ERROR]}
  - name: keep-slow
    type: latency
    latency: {threshold_ms: 1000}
  - name: baseline
    type: probabilistic
    probabilistic: {sampling_percentage: 5}
```

- **Ask first:** Can the pipeline preserve trace affinity and tolerate the decision latency?
- **Measure:** Collector queue depth, decision latency, dropped spans, retained traces by policy, and memory pressure.

## ⚡ Micro-action

Pick one critical request path and write down three retention rules: one error condition, one latency threshold, and one low-cost baseline percentage. Then identify the metric that would warn you when the sampler's queue or memory is becoming unsafe.

---

*Observability · 💫 Small steps. Every day.*
