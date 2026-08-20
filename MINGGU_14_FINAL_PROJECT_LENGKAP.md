# MINGGU 14: PROJECT PRESENTATION & INTEGRATION
## WORKSHOP ADMIN JARINGAN - PENS TI 2026

### TUJUAN
Minggu terakhir mengintegrasikan seluruh praktikum menggunakan baseline lab sederhana:
```
RB1100 gateway
   ↓
srv1 (.10) Infrastructure / Control Plane
   ↕
srv2 (.11) Workload / Data Plane
   ↕
Laptop mahasiswa
```

### PRE-PRESENTATION CHECKLIST
```bash
# srv1
systemctl status bind9 nginx prometheus
sudo wg show
sudo nft list ruleset
sudo systemctl status suricata

# Kubernetes
sudo kubectl get nodes
sudo kubectl get pods -A

# DNS
 dig @192.168.1XX.10 kelompokXX.lab

# Monitoring
curl http://192.168.1XX.10:9090/-/ready
```

### DEMO SCRIPT (15 MENIT)
1. **Topologi dan IP plan** — 2 menit
2. **Core services** — 3 menit
   - DNS query
   - DHCP lease
   - NFS/SMB client-server
3. **Web + monitoring** — 3 menit
   - Nginx load balancing ke `srv2:8081` dan `srv2:8082`
   - Prometheus/Grafana untuk srv1 dan srv2
4. **Containers + automation** — 3 menit
   - Docker Compose
   - K3s 2-node
   - Ansible inventory 2 host
5. **Security** — 2 menit
   - WireGuard
   - nftables
   - Suricata alert
6. **Q&A** — 2 menit

### PROJECT AKHIR REQUIREMENTS

**1. Network Design**
- VLAN/subnet kelompok terdokumentasi.
- RB1100 sebagai gateway/NAT.
- `srv1` dan `srv2` memiliki fungsi jelas.
- Overlay WireGuard `10.10.0.0/24` terdokumentasi.
- Pod/Service CIDR K3s terdokumentasi.

**2. Core Services**
- DNS forward/reverse zone.
- DHCP pool/reservation.
- Chrony server/client.
- Nginx reverse proxy.
- Dua backend logis pada srv2.
- NFS/SMB server-client.

**3. Monitoring**
- Prometheus memonitor srv1 dan srv2.
- Grafana minimal empat panel.
- Ada satu failure/recovery scenario.

**4. Container & Kubernetes**
- Docker Compose minimal tiga service/container untuk demonstrasi aplikasi.
- K3s 2-node (`srv1` server, `srv2` agent).
- Deployment, Service, NetworkPolicy, dan Ingress.

**5. Automation**
- Inventory Ansible berisi srv1 dan srv2.
- Minimal satu playbook multi-host.
- Ada bukti idempotency.

**6. Security**
- nftables baseline policy drop pada srv1.
- WireGuard tunnel srv1-srv2; laptop peer opsional/bonus.
- Suricata menghasilkan alert dari trafik uji yang diotorisasi.
- Log security dianalisis.

### DOKUMENTASI
Struktur repository yang disarankan:
```
project-kelompokXX/
├── README.md
├── docs/
│   ├── network-design.pdf
│   └── presentation.pptx
├── ansible/
│   ├── inventory.ini
│   └── playbooks/
├── configs/
│   ├── bind9/
│   ├── nginx/
│   ├── wireguard/
│   └── nftables.conf
├── docker/
│   └── docker-compose.yml
├── kubernetes/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── networkpolicy.yaml
├── monitoring/
│   ├── prometheus.yml
│   └── grafana/
└── scripts/
    ├── backup.sh
    └── health-check.sh
```

**Penting:** private key, password, token K3s, dan credential tidak boleh dimasukkan ke Git repository.

### MINIMUM ACCEPTANCE CHECKLIST
- [ ] Topologi 2 VM konsisten dengan dokumentasi
- [ ] Routing dan Internet access bekerja
- [ ] DNS + DHCP + NTP bekerja
- [ ] Web reverse proxy + 2 backend logis bekerja
- [ ] NFS/SMB bekerja antara srv1 dan srv2
- [ ] Prometheus/Grafana memonitor 2 node
- [ ] Docker Compose berjalan
- [ ] K3s 2-node berjalan
- [ ] Ansible multi-host idempotent
- [ ] WireGuard tunnel aktif
- [ ] nftables policy diterapkan
- [ ] Suricata alert dibuktikan
- [ ] Dokumentasi dan Git repository rapi

### RUBRIK PENILAIAN PROJECT (100 poin)

| Komponen | Poin | Kriteria |
|---|---:|---|
| Network Design | 15 | Topologi sederhana, IP scheme konsisten, routing jelas |
| Core Services | 20 | DNS/DHCP/NTP/Web/File functional |
| Monitoring | 15 | 2 node monitored, dashboard + failure test |
| Automation | 15 | Multi-host playbook dan idempotency |
| Containerization | 10 | Docker Compose + K3s 2-node |
| Security | 10 | Firewall, VPN, IDS |
| Documentation | 10 | Reproducible, credential-safe, repo rapi |
| Presentation | 5 | Demo jelas dan tepat waktu |
| **TOTAL** | **100** | |

### REFLEKSI
1. Bagian mana yang paling sulit diintegrasikan antara srv1 dan srv2?
2. Apa keuntungan dan keterbatasan topologi dua node untuk belajar?
3. Komponen mana yang perlu dipisahkan jika desain ini dibawa ke production?
4. Bagaimana automation dan observability mengurangi biaya operasional jaringan?

**Output:** Project akhir terintegrasi yang lebih ringan, reproducible, dan fokus pada kompetensi administrasi jaringan daripada jumlah mesin.
