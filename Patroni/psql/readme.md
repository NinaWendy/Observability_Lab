# 📘PostgreSQL Observability Dashboard

## Overview

This lab focuses on **monitoring PostgreSQL performance, availability, and health** using Prometheus and Grafana.
It demonstrates how to collect metrics, visualize them in Grafana, and configure alerting for key database health indicators.

---

### 🧩 Architecture

**Prometheus → PostgreSQL Exporter → Grafana**

* **PostgreSQL Exporter** exposes database metrics over `/metrics`.
* **Prometheus** scrapes the metrics and stores them.
* **Grafana** visualizes and alerts based on Prometheus data.

```
+--------------------+
|   PostgreSQL DB    |
| (Primary/Replica)  |
+---------+----------+
          |
          v
+--------------------+
| PostgreSQL Exporter|
|  (port 9187)       |
+---------+----------+
          |
          v
+--------------------+
|    Prometheus      |
| (Scrape + Alerts)  |
+---------+----------+
          |
          v
+--------------------+
|     Grafana        |
| (Visual Dashboards)|
+--------------------+
```

---

### ⚙️ Setup Steps

#### 1. Install and Configure PostgreSQL Exporter

Run the exporter on the same host as PostgreSQL or as a container.

```bash
docker run -d \
  --name=postgres_exporter \
  -p 9187:9187 \
  -e DATA_SOURCE_NAME="postgresql://postgres:password@192.168.100.145:5432/postgres?sslmode=disable" \
  prometheuscommunity/postgres-exporter
```

✅ **Verify exporter**

```bash
curl http://<exporter-ip>:9187/metrics
```

You should see metrics like `pg_up`, `pg_stat_activity_count`, etc.

---

#### 2. Add PostgreSQL Exporter to Prometheus Scrape Config

Update your `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'postgresql'
    static_configs:
      - targets: ['192.168.100.145:9187']
```

Then restart Prometheus:

```bash
systemctl restart prometheus
```

✅ **Verify in Prometheus UI:**
Navigate to `http://<prometheus-ip>:9090/targets` → ensure `postgresql` job is *UP*.

---

#### 3. Import Grafana Dashboard

You can use either:

* **Official dashboard**: [PostgreSQL Exporter Dashboard ID 9628](https://grafana.com/grafana/dashboards/9628)
* Or the provided `dashboard.json` file in this folder.

Steps:

1. Go to **Grafana → Dashboards → Import**
2. Upload the `dashboard.json` file or enter ID `9628`
3. Select your **Prometheus datasource**
4. Click **Import**

---

### 📊 Key Metrics Visualized

| Category        | Example Metric                                                                         | Description                  |
| --------------- | -------------------------------------------------------------------------------------- | ---------------------------- |
| Availability    | `pg_up`                                                                                | Shows if the database is up  |
| Connections     | `pg_stat_activity_count`                                                               | Active database sessions     |
| Query Load      | `pg_stat_database_xact_commit`, `pg_stat_database_xact_rollback`                       | Transaction performance      |
| Buffer Usage    | `pg_stat_bgwriter_buffers_backend_fsync`                                               | Buffer I/O activity          |
| Replication     | `pg_replication_lag`                                                                   | Replica delay behind primary |
| Table Stats     | `pg_stat_user_tables_seq_scan`                                                         | Sequential vs index scans    |
| Cache Hit Ratio | `pg_stat_database_blks_hit / (pg_stat_database_blks_hit + pg_stat_database_blks_read)` | Query performance efficiency |

---

### 🚨 Alerting Rules

Create a file `postgres-alerts.yml` and reference it in your Prometheus config:

Reload Prometheus after adding:

```bash
curl -X POST http://localhost:9090/-/reload
```

---

### 🧠 Insights

This setup helps you:

* Monitor database uptime and replication health
* Identify slow queries or I/O bottlenecks
* Track connection pool usage
* Receive alerts on outages or performance degradation

---

### 📸 Dashboard Preview

*(Add a screenshot of your PostgreSQL Grafana dashboard here)*
