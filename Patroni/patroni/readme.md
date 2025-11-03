
# Patroni – Full Cluster Observability

This module demonstrates end-to-end **monitoring for a Patroni PostgreSQL cluster** using the built-in `/metrics` endpoint and REST API.

We’ll collect, visualize, and alert on **leadership, liveliness, cluster health, replication lag, history, and configuration changes** using **Prometheus** and **Grafana**.

---

## 📖 Overview

**Patroni** provides high availability and auto-failover for PostgreSQL clusters.  
Each node exposes Prometheus metrics via the **REST API** on port **8008**.

We’ll integrate this with:

- **Prometheus** — to scrape and store metrics
- **Grafana** — to visualize the cluster’s health, state, and history
- (Optional) **Alertmanager** — to send alerts for failovers, replication lag, and unhealthy nodes

---

## ⚙️ Step 1: Enable Patroni Metrics Endpoint

Ensure your Patroni configuration exposes the REST API:

```yaml
restapi:
  listen: 0.0.0.0:8008
  connect_address: <node-ip>:8008
````

Restart Patroni and verify:

```bash
curl http://<patroni-node-ip>:8008/metrics
```

✅ You should see Prometheus-formatted metrics like:

```
# HELP patroni_primary 1 if this node is the leader, 0 otherwise.
# TYPE patroni_primary gauge
patroni_primary{scope="demo",name="patroni1"} 1

# HELP patroni_postgres_up 1 if PostgreSQL is running, 0 otherwise.
# TYPE patroni_postgres_up gauge
patroni_postgres_up{scope="demo",name="patroni1"} 1
```

---

## ⚙️ Step 2: Add Patroni to Prometheus

Edit your Prometheus config (`prometheus.yml`):

```yaml
scrape_configs:
  - job_name: 'patroni'
    metrics_path: '/metrics'
    static_configs:
      - targets:
          - 'patroni1.ip:8008'
          - 'patroni2.ip:8008'
          - 'patroni3.ip:8008'
```

Restart Prometheus and confirm targets are `UP` under
➡️ **Prometheus → Status → Targets**

---

## ⚙️ Step 3: (Optional) Collect Extended Cluster Info

The `/metrics` endpoint exposes real-time metrics.
For deeper insights (history, configuration, timeline), you can query `/patroni`:

```bash
curl http://<patroni-node>:8008/patroni
```

This returns JSON data such as:

```json
{
  "state": "running",
  "role": "leader",
  "timeline": 6,
  "xlog": {
    "received_location": 45687328,
    "replayed_location": 45687000
  },
  "tags": {
    "nosync": false
  }
}
```

You can use this for:

* Cluster history (timeline changes = failovers)
* Node state (leader, replica, initializing)
* Sync/async replication roles
* Failover detection

---

## 📊 Step 4: Key Metrics Breakdown

| Category               | Metric                           | Description                                           |
| ---------------------- | -------------------------------- | ----------------------------------------------------- |
| **Cluster Leadership** | `patroni_primary`                | 1 if node is the current leader, else 0               |
|                        | `patroni_cluster_unlocked`       | 1 if cluster has no active leader                     |
| **PostgreSQL Health**  | `patroni_postgres_up`            | 1 if PostgreSQL is running                            |
|                        | `patroni_postgres_state`         | Process state (e.g., 5 = running)                     |
| **Replication**        | `patroni_xlog_location`          | WAL position of primary                               |
|                        | `patroni_xlog_received_location` | WAL received by replicas                              |
|                        | `patroni_xlog_replayed_location` | WAL replayed on replicas                              |
|                        | `patroni_replication_lag_bytes`  | (Derived) WAL difference between primary and replicas |
| **Cluster State**      | `patroni_is_paused`              | 1 if cluster is in maintenance (paused) mode          |
|                        | `patroni_pending_restart`        | 1 if node requires restart                            |
|                        | `patroni_postgres_version`       | PostgreSQL version on node                            |
| **Liveliness**         | `patroni_postgres_up`            | Verifies DB liveliness                                |
| **History & Timeline** | `patroni_timeline_id`            | Current cluster timeline                              |
|                        | `patroni_timeline_change_count`  | Increments on failover (detects history)              |
| **Configuration**      | `patroni_scope`                  | Cluster name/scope                                    |
|                        | `patroni_name`                   | Node name                                             |
| **Health Summary**     | Derived using PromQL             | See below                                             |

---

## 📈 Step 5: Example PromQL Queries

### Leader node

```promql
patroni_primary == 1
```

### Replica count

```promql
count(patroni_primary == 0)
```

### Replication lag (bytes)

```promql
(patroni_xlog_location{role="leader"} - patroni_xlog_replayed_location{role="replica"})
```

### Cluster without leader

```promql
sum(patroni_primary) == 0
```

### PostgreSQL node down

```promql
patroni_postgres_up == 0
```

### Node requiring restart

```promql
patroni_pending_restart == 1
```

---

## 🛎️ Step 6: Alerts (Prometheus Rule Example)

```yaml
groups:
  - name: patroni-alerts
    rules:
      - alert: PatroniNoLeader
        expr: sum(patroni_primary) == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "No active Patroni leader"
          description: "All nodes report follower state for more than 2 minutes."

      - alert: PatroniReplicationLag
        expr: (patroni_xlog_location - patroni_xlog_replayed_location) > 52428800
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "High replication lag"
          description: "Replication lag exceeds 50MB."

      - alert: PatroniDBDown
        expr: patroni_postgres_up == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: "Patroni PostgreSQL instance down"
```

---

## 📊 Step 7: Grafana Dashboard

📁 **Dashboard File:** [`dashboard.json`](dashboard.json)

Panels include:

* Cluster overview (leader, replicas)
* PostgreSQL health summary
* Replication lag visualization
* Failover timeline
* Pending restarts
* Maintenance mode indicator
* Current timeline vs history

![Patroni Dashboard](../../docs/screenshots/patroni-full-dashboard.png)

---

**References:**

* [Patroni REST API – Monitoring Endpoint](https://patroni.readthedocs.io/en/latest/rest_api.html#monitoring-endpoint)
* [Prometheus Official Documentation](https://prometheus.io/docs/introduction/overview/)
* [Grafana Docs](https://grafana.com/docs/)
* [Postgres Exporter](https://github.com/prometheus-community/postgres_exporter)