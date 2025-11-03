# 📊 etcd Monitoring with Prometheus and Grafana

This guide demonstrates how to set up **end-to-end observability** for an **etcd cluster**, using **Prometheus** for metrics collection and **Grafana** for visualization.

---

## 🧭 Overview

etcd exposes detailed Prometheus-style metrics that provide insights into:

| Metric Category | Description |
|------------------|-------------|
| 🟢 **Health & Leadership** | Tracks leader availability and leadership transitions |
| ⚙️ **Performance Metrics** | Monitors Raft proposal rates, client requests, and latency |
| 💾 **Storage Metrics** | Observes WAL and database size, fsync durations |
| 📈 **Network Metrics** | Measures peer round-trip latency and RPC performance |

This setup ensures visibility into the health and performance of your etcd cluster.

---

## ⚙️ Step 1: Verify Metrics Endpoint

etcd exposes metrics at port **2379** by default.

Run the following command to verify:
```bash
curl http://localhost:2379/metrics | head
````

Expected output:

```
# HELP etcd_server_has_leader Whether a leader exists
# TYPE etcd_server_has_leader gauge
etcd_server_has_leader 1
```

If you see metric entries like above, the endpoint is active.

---

## ⚙️ Step 2: Configure Prometheus to Scrape etcd Metrics

Update your Prometheus configuration file (`/etc/prometheus/prometheus.yml`) to include the etcd scrape job:

```yaml
scrape_configs:
  - job_name: 'etcd'
    static_configs:
      - targets:
          - '10.0.0.1:2379'
          - '10.0.0.2:2379'
          - '10.0.0.3:2379'
```

Restart Prometheus:

```bash
sudo systemctl restart prometheus
```

Verify the etcd job is active:

```
http://<prometheus-ip>:9090/targets
```

If configured correctly, all etcd targets should appear as **UP**.

---

## 📈 Step 3: Add Prometheus as a Grafana Data Source

1. In Grafana, go to **Settings → Data Sources**
2. Click **Add data source**
3. Select **Prometheus**
4. Enter the URL:

   ```
   http://<prometheus-ip>:9090
   ```
5. Click **Save & Test** — you should see “Data source is working”.

---

## 📊 Step 4: Import the etcd Dashboard

Grafana provides an official etcd dashboard:

| Property         | Value                     |
| ---------------- | ------------------------- |
| **Dashboard ID** | 3070                      |
| **Name**         | Etcd Metrics (Prometheus) |
| **Source**       | Grafana.com               |

### To import:

1. Navigate to **Dashboards → Import**
2. Enter the dashboard ID `3070`
3. Click **Load**
4. Choose your Prometheus data source
5. Click **Import**

You should now see panels showing:

* Leader changes and Raft proposal metrics
* Client request traffic
* Database size and disk latency
* Peer communication and replication delay

---

## ⚡ Step 5: Add Alerting Rules (Optional)

To detect cluster instability or performance degradation, create alert rules in Prometheus.

**Example `etcd_alerts.yml`:**

```yaml
groups:
  - name: etcd_alerts
    rules:
      - alert: EtcdNoLeader
        expr: etcd_server_has_leader == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: "No leader in etcd cluster"
          description: "etcd instance {{ $labels.instance }} has lost its leader."

      - alert: EtcdHighProposalFailures
        expr: rate(etcd_server_proposals_failed_total[5m]) > 0
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High proposal failure rate detected"
          description: "etcd instance {{ $labels.instance }} is experiencing Raft proposal failures."
```

Reload Prometheus:

```bash
curl -X POST http://localhost:9090/-/reload
```

---

## ✅ Step 7: Final Dashboard View

Once configured, you’ll have an observability dashboard showing:

| Metric                                      | Description                       |
| ------------------------------------------- | --------------------------------- |
| `etcd_server_has_leader`                    | Whether a leader currently exists |
| `etcd_server_leader_changes_seen_total`     | Number of leadership transitions  |
| `etcd_server_proposals_failed_total`        | Failed Raft proposals             |
| `etcd_network_peer_round_trip_time_seconds` | Latency between etcd peers        |
| `etcd_disk_wal_fsync_duration_seconds`      | Disk sync times for WAL           |
| `etcd_mvcc_db_total_size_in_bytes`          | Current database size             |

---

## 📚 References

* [etcd Metrics Documentation](https://etcd.io/docs/v3.5/op-guide/monitoring/)
* [Grafana Dashboard #3070](https://grafana.com/grafana/dashboards/3070-etcd/)
* [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)