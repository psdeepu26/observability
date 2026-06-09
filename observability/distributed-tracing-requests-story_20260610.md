# Distributed Tracing: How Requests Tell Their Story Across Microservices

**Observability · Wednesday, June 10, 2026 · Insight #2**

---

## 🧒 Like You're 5

Imagine you're sending a toy on a relay race through 10 classrooms. Each teacher writes down *"Got the toy at 2:01, passed it on at 2:03"* on a sticky note. At the end, you collect all the sticky notes and lay them out — you can see exactly where the toy got stuck, which teacher was fastest, and who took forever.

That's distributed tracing. Every service a request touches stamps a **trace** with a unique ID, records **spans** (what it did, how long it took, what went wrong), and a central system stitches them into a timeline.

---

## 🔧 For the Engineer

### Trace & Span Anatomy

- **Trace ID** — globally unique, follows the request end-to-end
- **Span** — one operation; has start/end, name, attributes
- **Parent Span ID** — links parent → child (tree structure)
- **Baggage** — key-value pairs propagated across spans

### W3C Trace Context

- `traceparent` header carries trace-id + span-id
- `tracestate` carries vendor-specific data
- Auto-propagation via HTTP, gRPC, message queues
- OpenTelemetry SDK handles this transparently

### Sampling Strategies

- **Head-based** — decide at trace start (fast, but may miss errors)
- **Tail-based** — decide at end (capture only slow/error traces, needs buffer)
- NVCF-scale: use **probabilistic** (e.g., 1%) + always sample errors

### Jaeger vs Tempo vs Commercial

| Tool | Storage | Best for |
|------|---------|----------|
| Jaeger | ES/Cassandra | OSS, k8s-native |
| Grafana Tempo | Object store | Grafana stack, cheap |
| Datadog APM | Managed | Full-stack, NVCF-like |
| Honeycomb | Managed | High-cardinality events |

### Python: Instrumenting a FastAPI Handler

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from fastapi import FastAPI

provider = TracerProvider()
provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter()))
trace.set_tracer_provider(provider)
tracer = trace.get_tracer("nvcf-inference")

app = FastAPI()

@app.post("/v1/inference")
async def inference(request: Request):
    with tracer.start_as_current_span(
        "nvcf-inference-handler",
        attributes={"model": request.model_name}
    ) as span:
        with tracer.start_span("model-load"):
            model = await load_model(request.model_name)

        with tracer.start_span(
            "gpu-inference",
            attributes={"gpu": "A100"}
        ) as gpu_span:
            result = await run_inference(model, request.input)
            gpu_span.set_attribute("tokens", result.token_count)

        span.set_attribute("status", "success")
        return result
```

### Tail-Based Sampling (OTel Collector Config)

```yaml
processors:
  tail_sampling:
    decision_wait: 10s
    policies:
      - name: error-policy
        type: status_code
        status_code: {status_codes: [ERROR]}
      - name: slow-policy
        type: latency
        latency: {threshold_ms: 200}
      - name: probabilistic-policy
        type: probabilistic
        probabilistic: {percentage: 1}
```

### NVCF Tracing: What to Watch

**Critical Span Attributes:**
- `model_name` — which NIM/container
- `gpu_type` — A100/H100/GPU pool
- `replica_id` — which pod handled it
- `queue_depth` — queued requests count
- `batch_size` — dynamic batching width

**SLO Alerts from Traces:**
- P99 e2e latency > model SLA threshold
- Inference span > load + transfer time (GPU issue)
- Auth span p95 regression → key rotation issue
- Trace sampling drops → collector backpressure

---

## ⚡ Micro-Action (5 min)

Check if your OpenTelemetry Collector pipeline is healthy:

```bash
curl -s localhost:8888/metrics | grep accepted_spans
```

Expected: `otelcol_receiver_accepted_spans{ref="otlp",service="nvcf-backend"} 1429`

If zero or missing, your services aren't exporting traces. Add `prometheus:` under `exporters:` in your collector config and restart.
