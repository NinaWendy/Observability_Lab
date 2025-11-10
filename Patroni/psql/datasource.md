# Prometheus Datasource Configuration

To add Prometheus as a datasource in Grafana:

1. Go to **Grafana → Connections → Data Sources → Add data source**
2. Choose **Prometheus**
3. Set URL:

   ```
   http://<prometheus-ip>:9090
   ```
   
4. Click **Save & Test**

---

## Prometheus Scrape Configuration

Add to `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'postgresql'
    static_configs:
      - targets: ['192.168.100.145:9187']
```

---

### Verification

In Grafana → **Explore**, run queries like:

```promql
pg_up
pg_stat_activity_count
pg_replication_lag
```

You should see real-time values from your PostgreSQL exporter.

