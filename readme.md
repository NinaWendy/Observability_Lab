# 🧠 Observability Lab

An end-to-end observability showcase covering databases, load balancers, Kubernetes, and security tools with dashboards, metrics setup, and configuration for each.

Each folder (e.g., Patroni, HAProxy, K8s, Wazuh, UptimeKuma) is **independent**, containing:
- Step-by-step setup guide
- Metrics datasource setup
- Grafana dashboard JSON
- Final dashboard screenshots

---

## 🧩 Components Covered

| Component | Submodules | Metrics Tool | Visualization |
|------------|-------------|---------------|----------------|
| Patroni | psql, etcd, patroni | Prometheus exporters | Grafana |
| HAProxy | etcd, haproxy | Prometheus exporter | Grafana |
| K8s | cluster | kube-state-metrics, node-exporter | Grafana |
| Wazuh | agent | Wazuh API | Grafana |
| UptimeKuma | services | Uptime API | Grafana |

---

## 📊 Example Flow (for each component)

1. **Metrics Setup:** Configure exporter → Confirm /metrics endpoint.
2. **Datasource:** Connect exporter endpoint to Prometheus or direct Grafana datasource.
3. **Visualization:** Import dashboard JSON and tune metrics.
4. **Validation:** Check alerting rules, screenshot final dashboard.

---

## 🖼️ Architecture Overview
![architecture](docs/architecture.png)

---

## 📚 Docs
- [Setup Guide](docs/setup-guide.md)
- [Alerting Rules](docs/alerts.md)
- [What I Learned](docs/overview.md)

---

## 💡 Example Dashboard Screenshots
(See `/docs/screenshots` for visuals)

---

## 🧑‍💻 Author
**Honeycomb**  
🔗 [LinkedIn](#) | [Twitter](#) | [Portfolio](#)
