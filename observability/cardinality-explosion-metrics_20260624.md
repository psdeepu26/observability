# Cardinality Explosion: When Your Metrics Database Eats Itself

> **Topic:** Observability | **Date:** 2026-06-24 | **Category:** observability

## 🧒 Like you're 5

Imagine you have a jar where you keep marbles — one marble for every cat in your neighborhood. Cute, right? Now imagine someone says, "Let's track every *color* of fur, in every *lighting condition*, from every *angle*, for every cat." Suddenly you need a million marbles and the jar explodes. That's what happens to your monitoring system when you let too many unique labels pile up — the database that stores all those marbles gets so full and slow that it stops working for *anyone*.

↓

## 🔧 For the Engineer

High cardinality in time-series databases (Thanos, Cortex, Prometheus, VictoriaMetrics) is the #1 silent killer of observability stacks. It happens when a label has an unbounded set of values — think `user_id`, `request_id`, `container_id`, or `gpu_uuid` as metric labels.

### Why It's Especially Dangerous in GPU/NVCF Environments

In Nvidia SRE contexts, the temptation is to tag metrics with `gpu_id`, `pod_name`, `node_name`, and `nvcf_deployment_id` — each potentially thousands of unique values. Multiply them together and you get combinatorial explosion:

```promql
# BAD: Each GPU gets its own time series
dcgmi_power_usage{gpu_id="gpu-0", node="node-a1", pod="inference-xyz", deployment="nvcf-prod"}
# ... × 8 GPUs × 50 nodes × 200 pods × 10 deployments = 800,000 series
```

### The Fix: Label Discipline + Recording Rules

```yaml
# recording-rule.yml
groups:
  - name: gpu_cluster_aggregates
    interval: 30s
    rules:
      - record: dcgm:power_usage:sum_by_cluster
        expr: sum by (cluster, deployment) (dcgmi_power_usage{Watts})
      - record: dcgm:gpu_utilization:avg_by_node
        expr: avg by (node, cluster) (dcgmi_gpu_utilization_pct)
```

```yaml
# prometheus.yml - drop high-cardinality series at scrape
scrape_configs:
  - job_name: 'dcgm-exporter'
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: 'dcgmi_power_usage'
        action: drop
```

### Cardinality Debugging Commands

```bash
# Find highest-cardinality metrics
curl -s 'http://localhost:9090/api/v1/label/__name__/values' | jq '.data[]' | \
  while read metric; do
    count=$(curl -s "http://localhost:9090/api/v1/series?match[]=$metric" | jq '.data | length')
    echo "$count $metric"
  done | sort -rn | head -20

# Quick cardinality check per label
count by (label_name) (metric_name)
```

### When to Break the Rule

High cardinality is fine for **logs** and **traces** (they're indexed differently). The problem is only with **metrics**. Use exemplars to link metrics → traces instead of putting trace IDs in metric labels.

## ⚡ Micro-action (5 min)

1. Open Prometheus/Thanos UI → Status → TSDB Status → check "Head Cardinality"
2. Run: `topk(10, count by (__name__) ({cluster="your-cluster"}))`
3. For any metric over 10k series, check if labels like `id`, `uuid`, or `name` are the culprit
4. Add a `metric_relabel_config` to drop or hash those labels at scrape time
