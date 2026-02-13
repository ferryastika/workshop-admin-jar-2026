
# MINGGU 7: NETWORK MONITORING & OBSERVABILITY
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam)
**Monitoring vs Observability:**
```
Monitoring: "Is it up?"
Observability: "Why is it down?" (Metrics + Logs + Traces)

3 Pillars Observability:
├── Metrics (Prometheus time series)
├── Logs (Loki/ELK structured)
└── Traces (Jaeger distributed)
```

**Prometheus Architecture:**
```
Node Exporter ← Metrics → Prometheus ← Alertmanager
                              ↓ Query
                           Grafana Dashboards
```

### PERTANYAAN TEORI
1. Apa beda SNMP polling vs Prometheus scraping?
2. Kenapa Prometheus pakai pull bukan push model?
3. Fungsi retention policy 15 hari?
4. Grafana vs Kibana use case?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
node_exporter (srv2,srv3) ← HTTP 9100 → Prometheus srv1:9090
                                              ↓ HTTP 3000
                                          Grafana srv1:3000
```

**Hosts:**
```
Prometheus+Grafana: kXX-srv1 192.168.1XX.10
Node Exporter 1: kXX-srv2 192.168.1XX.11:9100
Node Exporter 2: kXX-srv3 192.168.1XX.12:9100
```

**Aplikasi:**
```
prometheus, node_exporter, grafana, nano
```

### LANGKAH PRAKTIKUM (2 jam)

**1. Node Exporter Installation (15 menit)**
```bash
# srv2 & srv3
sudo useradd --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar xvfz node_exporter-*.tar.gz
sudo mv node_exporter-*/node_exporter /usr/local/bin/
sudo nano /etc/systemd/system/node_exporter.service
```
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter \
  --collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc|run) \$
ExecReload=/bin/kill -HUP \$MAINPID

[Install]
WantedBy=default.target
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
curl 192.168.1XX.11:9100/metrics | head -20
```

**2. Prometheus Installation srv1 (20 menit)**
```bash
sudo useradd --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
tar xvfz prometheus-*.tar.gz
sudo mv prometheus-*/{prometheus,promtool} /usr/local/bin/
sudo mkdir /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

**3. Prometheus Configuration (20 menit)**
```bash
sudo nano /etc/prometheus/prometheus.yml
```
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node'
    static_configs:
      - targets: ['192.168.1XX.10:9100']
  - job_name: 'node_srv2'
    static_configs:
      - targets: ['192.168.1XX.11:9100']
  - job_name: 'node_srv3'
    static_configs:
      - targets: ['192.168.1XX.12:9100']
```
```bash
sudo nano /lib/systemd/system/prometheus.service
```
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries
Restart=always

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
curl http://192.168.1XX.10:9090/targets
```

**4. Grafana Installation (20 menit)**
```bash
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_11.1.0_amd64.deb
sudo dpkg -i grafana_*.deb
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

**5. Grafana Configuration (15 menit)**
```
Browser: http://192.168.1XX.10:3000
Login: admin/admin
```
```
Configuration → Data Sources → Add Prometheus
URL: http://localhost:9090
Save & Test ✓

Dashboards → Import → 1860 (Node Exporter Full)
```

**6. Custom Dashboard (15 menit)**
```
New Dashboard → Add Panel:
1. CPU Usage: node_cpu_seconds_total{instance=~".*"}
2. Memory: (1 - node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes)*100
3. Disk: node_filesystem_avail_bytes
4. Network: rate(node_network_receive_bytes_total[5m])
Save as "Kelompok XX Monitoring"
```

### UJI KONFIGURASI
```
Prometheus targets: 4/4 UP ✓
Grafana Dashboard: CPU <80%, RAM <70%, Disk >20%
```

**Screenshot Wajib (10 gambar):**
1. node_exporter service status
2. Prometheus targets page
3. Grafana Prometheus datasource
4. Node Exporter Full dashboard
5. Custom 4-panel dashboard
6. CPU graph
7. Memory graph
8. Network traffic
9. `curl 192.168.1XX.11:9100/metrics`
10. Grafana user settings

### PERTANYAAN SEKITAR PRAKTIKUM
1. Jika target DOWN, cek firewall port 9100?
2. Kenapa scrape_interval 15s bukan 1 menit?
3. Fungsi `--collector.filesystem.ignored-mount-points`?
4. Grafana alert rule simple seperti apa?

### CHECKLIST TUGAS MINGGU 7
- [ ] node_exporter 3 server running
- [ ] Prometheus scrape 4 targets
- [ ] Grafana datasource OK
- [ ] Custom dashboard 4 panels
- [ ] 10 screenshot lengkap

**Waktu Total:** 2 jam
**Output:** Monitoring stack Prometheus+Grafana operational
