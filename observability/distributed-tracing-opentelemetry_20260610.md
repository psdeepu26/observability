# 🔍 Distributed Tracing with OpenTelemetry

**Observability** · Day #2 · 10 Jun 2026

---

## 🧒 Like You're 5

Imagine you order a pizza. The delivery doesn't just magically appear — it goes through steps: the restaurant gets the order, the chef makes it, the driver picks it up, and brings it to you. Now imagine each step writes a tiny note: *"Order received at 6:01"*, *"Pizza in oven at 6:05"*, *"Out for delivery at 6:15"*.

If your pizza is late, you can look at all the notes and find out exactly where the delay happened — was the chef slow? Was the driver stuck in traffic?

**Distributed tracing** does this for software. When a user request travels through 10 different microservices, tracing records a "span" at each stop — with a timestamp, duration, and context — so you can see the entire journey and pinpoint exactly where things slow down or break.

---

## 🔧 For the Engineer

### Trace → Span → SpanContext

A `Trace` is the full journey (like the pizza delivery). Each service creates a `Span` — a named, timed operation. Spans link to parents via `SpanContext` (trace_id + span_id), forming a tree.

### Context Propagation (the magic)

How does Service A tell Service B which trace it's in? Via **propagation** — the trace_id and span_id are injected into HTTP headers (e.g., `traceparent` in W3C Trace Context) and extracted on the other side. Like passing a tracking number between delivery depots.

### Sampling: You Can't Trace Everything

At 1M requests/sec, tracing all of them would cost more than the infra itself. **Head-based sampling** decides at the trace root (e.g., sample 1%). **Tail-based sampling** keeps all traces but only stores the interesting ones (errors, high latency) — this is what Tempo + Grafana excels at.

### OpenTelemetry: The Standard

OTel is the CNCF project that unifies traces, metrics, and logs under one SDK. It's vendor-neutral — instrument once, send to Jaeger, Tempo, Datadog, or Honeycomb. For Nvidia's NVCF platform, OTel spans can track GPU inference requests end-to-end.

### Example: A request through 3 services

```
├─ POST /api/inference [trace: abc123] ━━━━━━━━━━━━━━━━ 340ms
│  ├─ auth.validate_token [span: def456] ━━ 12ms
│  ├─ model.predict [span: ghi789] ━━━━━━━━━━━━ 280ms ⚠️ slow
│  │  ├─ gpu.memcpy [span: jkl012] ━━ 15ms
│  │  └─ gpu.kernel_exec [span: mno345] ━━━━━━━━ 255ms 🔥 bottleneck
│  └─ response.serialize [span: pqr678] ━━ 8ms
```

### Python: Instrumenting a span with OTel

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

# Configure the tracer
provider = TracerProvider()
otlp_exporter = OTLPSpanExporter(endpoint="tempo-distributor:4317")
provider.add_span_processor(BatchSpanProcessor(otlp_exporter))
trace.set_tracer_provider(provider)

tracer = trace.get_tracer("nvcf.inference")

def handle_inference(request):
    with tracer.start_as_current_span("model.predict") as span:
        span.set_attribute("model.name", "llama-70b")
        span.set_attribute("gpu.device", 0)
        result = run_model(request)
        span.set_attribute("inference.tokens", result.tokens)
        return result
        # Child spans (gpu.memcpy, kernel_exec) nest automatically
        # via context propagation through gRPC/HTTP
```

---

## ⚡ Micro-Action

**Today's 5-minute task:** Pick one service in your stack that has zero tracing. Add a single `tracer.start_as_current_span()` call around its main handler. Export to Jaeger or Tempo (even locally via Docker). Make one request and see the span appear. That's it — you've just taken the first step from "blind" to "observable."
