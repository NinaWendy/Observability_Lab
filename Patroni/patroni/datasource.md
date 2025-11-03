# 📘 Patroni Prometheus Datasource Setup

## Objective

To collect Patroni cluster metrics (health, roles, replication, and failovers) using **Prometheus**, and configure **alerting rules** to monitor cluster stability and performance.


## ⚙️ Step 1 — Enable Patroni Metrics Endpoint

Patroni exposes Prometheus-compatible metrics on port `8008` by default.
To confirm, check your Patroni configuration file (e.g., `/etc/patroni/patroni.yml`):

```yaml
restapi:
  listen: 0.0.0.0:8008
  connect_address: patroni-node1:8008
```

Then verify it’s accessible:

```bash
curl http://patroni-node1:8008/metrics
```

✅ You should see metrics like:

```
# HELP patroni_cluster_unhealthy
# HELP patroni_timeline_id
# HELP patroni_postgres_replication_lag_bytes
```

If you get a response, the metrics endpoint is active.

---

## 🧭 Step 2 — Configure Prometheus to Scrape Patroni Metrics

Edit your Prometheus configuration (commonly `/etc/prometheus/prometheus.yml`)

Save and reload Prometheus:

```bash
sudo systemctl restart prometheus
```

Check the targets via Prometheus web UI →
`http://<prometheus-server>:9090/targets`

✅ All Patroni nodes should show **UP**.

---

## 🚨 Step 3 — Add Alerting Rules for Patroni Cluster

Create a rule file `/etc/prometheus/rules/patroni-alerts.yml`:

Then include this file in `prometheus.yml`:

```yaml
rule_files:
  - "/etc/prometheus/rules/patroni-alerts.yml"
```

Reload Prometheus again:

```bash
sudo systemctl reload prometheus
```

You can verify the alert rules from:
`http://<prometheus-server>:9090/rules`

---

## 🧩 Step 4 — Validate

1. Temporarily stop one Patroni node:

   ```bash
   sudo systemctl stop patroni
   ```

   Within ~1 minute, you should see **PatroniProcessDown** alert firing.

2. Simulate lag or failover to trigger other alerts.

---

## ✅ Step 5 — Connect to Grafana

Once metrics and alerts are live:

1. Go to Grafana → **Connections → Data Sources → Add Prometheus**
2. Enter the URL:

   ```
   http://<prometheus-server>:9090
   ```
3. Click **Save & Test** — Grafana will confirm connection.
4. Import your `dashboard.json` to visualize the data.

---

## 📦 Outcome

After completing these steps, you’ll have:

* Prometheus continuously scraping Patroni cluster metrics
* Alerting rules detecting failures, unhealthy states, or high lag
* Grafana ready to visualize all cluster health data