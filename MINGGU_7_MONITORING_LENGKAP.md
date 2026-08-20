# MINGGU 7: NETWORK MONITORING & OBSERVABILITY
## WORKSHOP ADMIN JARINGAN - PENS TI 2026

### DASAR TEORI (1 jam)
**Monitoring vs Observability:**
```
Monitoring: "Is it up?"
Observability: "Why is it down?"
```

**Topologi monitoring sederhana:**
```
node_exporter srv1:9100 ─┐
                         ├→ Prometheus srv1:9090 → Grafana srv1:3000
node_exporter srv2:9100 ─┘
```

### PERTANYAAN TEORI
1. Apa beda SNMP polling vs Prometheus scraping?
2. Kenapa Prometheus menggunakan pull model?
3. Apa fungsi label `instance` pada time series?
4. Mengapa target lokal dan remote tetap berguna untuk troubleshooting?

### KEBUTUHAN PRAKTIKUM
**Hosts:**
```
Monitoring Server: kXX-srv1 192.168.1XX.10
Remote Target:     kXX-srv2 192.168.1XX.11
```

### LANGKAH PRAKTIKUM (2 jam)

**1. Install node_exporter pada srv1 dan srv2 (20 menit)**
Gunakan package distribusi bila tersedia:
```bash
sudo apt update
sudo apt install -y prometheus-node-exporter
sudo systemctl enable --now prometheus-node-exporter
curl http://localhost:9100/metrics | head
```

**2. Install Prometheus pada srv1 (20 menit)**
```bash
sudo apt install -y prometheus
sudo systemctl enable --now prometheus
```

Edit `/etc/prometheus/prometheus.yml`:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'nodes'
    static_configs:
      - targets:
          - '192.168.1XX.10:9100'
          - '192.168.1XX.11:9100'
```

```bash
sudo promtool check config /etc/prometheus/prometheus.yml
sudo systemctl restart prometheus
```

**3. Verify Targets (15 menit)**
```bash
curl http://192.168.1XX.10:9090/-/ready
curl http://192.168.1XX.11:9100/metrics | head
```
Buka `http://192.168.1XX.10:9090/targets` dan pastikan kedua target `UP`.

**4. Install Grafana pada srv1 (20 menit)**
Gunakan package/repository Grafana yang disediakan instruktur atau image Docker bila repo lokal tidak menyediakan versi yang dibutuhkan. Setelah service aktif, akses:
```
http://192.168.1XX.10:3000
```
Tambahkan Prometheus datasource:
```
http://localhost:9090
```

**5. Dashboard Dasar (25 menit)**
Buat minimal empat panel:
1. CPU usage per instance
2. Memory utilization per instance
3. Filesystem available
4. Network receive/transmit rate

Contoh query memory:
```promql
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
```

Contoh network receive:
```promql
rate(node_network_receive_bytes_total[5m])
```

**6. Failure Simulation (20 menit)**
Pada srv2:
```bash
sudo systemctl stop prometheus-node-exporter
```
Amati target menjadi `DOWN`, identifikasi penyebab, kemudian pulihkan:
```bash
sudo systemctl start prometheus-node-exporter
```

### UJI KONFIGURASI
- Prometheus: 2/2 node targets `UP`.
- Grafana menampilkan data srv1 dan srv2.
- Mahasiswa dapat menjelaskan perubahan target saat exporter srv2 dihentikan.

### CHECKLIST TUGAS MINGGU 7
- [ ] node_exporter berjalan di srv1 dan srv2
- [ ] Prometheus scrape 2 node targets
- [ ] Grafana datasource OK
- [ ] Dashboard minimal 4 panel
- [ ] Failure simulation dan recovery terdokumentasi

**Output:** Monitoring dua node yang cukup untuk mempraktikkan observability dan troubleshooting target remote.
