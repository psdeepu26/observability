# SLOs & Error Budgets: The Contract Between Reliability and Innovation

**Date:** June 24, 2026
**Topic:** Observability
**Category:** slo-error-budgets

## ELI5

Imagine your parents say you can eat **9 cookies per week**. That's your "budget." If you eat only 7 this week, you have 2 left over. But if you eat all 9 by Wednesday, no more cookies until next week!

An **error budget** works the same way for your services. You promise your users "99.9% uptime" — that means you're *allowed* to be down for about 43 minutes per month. Every minute of downtime "spends" your budget. Once it's gone, the team must focus on reliability before shipping new features.

## For the Engineer

### The Core Math

An SLO (Service Level Objective) defines the target reliability. The error budget is the *inverse* — the allowable failure.

| SLO | Monthly Error Budget |
|-----|---------------------|
| 99.9% | ~43.8 min |
| 99.95% | ~21.9 min |
| 99.99% | ~4.38 min |

### Burn Rate & Alerting

Raw error budget consumption is too coarse. **Burn rate alerts** detect when you're on track to exhaust the budget too quickly.

```yaml
# Multi-window, multi-burn-rate alert (Prometheus-style)
- alert: ErrorBudgetBurningFast
  expr: |
    (
      1 - (
        sum(rate(http_requests_total{status!~"5.."}[1h]))
        /
        sum(rate(http_requests_total[1h]))
      )
    ) > (1 - 0.999) * 14.4
    AND
    (
      1 - (
        sum(rate(http_requests_total{status!~"5.."}[5m]))
        /
        sum(rate(http_requests_total[5m]))
      )
    ) > (1 - 0.999) * 14.4
  for: 2m
  labels:
    severity: critical
```

A burn rate of **14.4x** means you'd exhaust a 30-day error budget in ~2 days. The dual-window (5m + 1h) prevents flapping.

### SLI vs SLO vs SLA

- **SLI (Indicator):** The measurement — `good_events / total_events`
- **SLO (Objective):** The target — `SLI >= 99.9%`
- **SLA (Agreement):** The contract — `Credits if SLO missed`
- **Error Budget:** The allowance — `1 - SLO = 0.1%`

### Policy When Budget Is Exhausted

1. Freeze non-critical deploys
2. Error budget policy document (not tribal knowledge)
3. SLO status dashboard visible to leadership
4. Prorate budgets for new services ramping up

## Micro-Action

**Do this today (5 minutes):** Pick one service you own or depend on. Check: does it have a documented SLO? If not, draft one — even a rough "99.9% of requests succeed with p99 <500ms" is a starting point. Write it in a doc and share it with your team. The act of defining the number forces a conversation about what "good" means.
