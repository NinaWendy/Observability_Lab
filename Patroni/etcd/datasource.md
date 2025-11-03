# Prometheus Datasource Configuration

This document explains how to configure **Prometheus** as a data source for collecting etcd metrics and connecting it to **Grafana** for visualization.

---

## 🧩 Step 1: Prometheus Scrape Configuration

In your Prometheus configuration file (`/etc/prometheus/prometheus.yml`), add an **etcd** job under `scrape_configs`:

```yaml
scrape_configs:
  - job_name: 'etcd'
    static_configs:
      - targets:
          - '10.0.0.1:2379'
          - '10.0.0.2:2379'
          - '10.0.0.3:2379'
````

Then verify it’s accessible:

```bash
curl http://etcd-node1:2379/metrics
```

Restart Prometheus:

```bash
sudo systemctl restart prometheus
```

✅ Verify the scrape targets:

```
http://<prometheus-ip>:9090/targets
```

Each etcd instance should appear as **UP**.

---

## ⚠️ Step 2: Add Alerting Rules

Define alert rules in a file such as `etcd_alerts.yml`, and include it in Prometheus config:

```yaml
rule_files:
  - "etcd_alerts.yml"
```

### Example Alert Rules

```yaml
groups:
  - name: etcd_alerts
    rules:
      - alert: EtcdNoLeader
        expr: etcd_server_has_leader == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "No leader in etcd cluster"
          description: "etcd cluster {{ $labels.instance }} has lost its leader."

      - alert: EtcdHighProposalFailures
        expr: rate(etcd_server_proposals_failed_total[5m]) > 0
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High proposal failure rate"
          description: "Raft proposals failing on {{ $labels.instance }}"
```

Reload Prometheus to apply changes:

```bash
curl -X POST http://localhost:9090/-/reload
```

---

## 🌐 Step 3: Configure Grafana Data Source

1. Go to **Grafana → Settings → Data Sources**
2. Click **Add data source**
3. Select **Prometheus**
4. Enter the URL:

   ```
   http://<prometheus-ip>:9090
   ```
5. Click **Save & Test** — should return *Data source is working*

---

## 🔍 Step 4: Validate Metrics Collection

In Grafana’s **Explore** tab, verify Prometheus is receiving etcd metrics.

Try queries like:

```promql
etcd_server_has_leader
etcd_network_peer_round_trip_time_seconds
etcd_mvcc_db_total_size_in_bytes
```

If these return results, the data source is correctly configured.

---

## 🧠 Step 5: Dashboard Integration

When importing your Grafana dashboard (`dashboard.json`), select the configured **Prometheus** data source.
If your dashboard JSON includes a variable such as `${DS_PROMETHEUS}`, Grafana will prompt you to choose the data source.

---

## ✅ Verification Checklist

| Check               | Expected Result                         |
| ------------------- | --------------------------------------- |
| Prometheus target   | UP                                      |
| Grafana data source | Connected                               |
| Dashboard panels    | Displaying etcd metrics                 |
| Alerts              | Trigger on leader loss or high failures |

---

## 📚 References

* [etcd Monitoring Guide](https://etcd.io/docs/v3.5/op-guide/monitoring/)
* [Prometheus Official Docs](https://prometheus.io/docs/)
* [Grafana Documentation](https://grafana.com/docs/)