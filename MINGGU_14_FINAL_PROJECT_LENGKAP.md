
# MINGGU 14: ADVANCED TOPICS & PROJECT PRESENTATION
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam - Advanced Topics)
**AI-Driven Network Operations (AIOps):**
```
Traditional NOC:
- Manual monitoring
- Reactive troubleshooting
- Alert fatigue

AIOps 2026:
- Predictive analytics (ML)
- Anomaly detection
- Self-healing networks
- Root cause analysis automation
```

**Emerging Technologies:**
1. **Edge Computing + 5G:** Network slicing, MEC
2. **Intent-Based Networking:** Policy abstraction
3. **Network Digital Twin:** Virtual simulation
4. **eBPF:** Programmable kernel networking

### PERTANYAAN TEORI
1. AIOps vs traditional monitoring alert reduction?
2. Network digital twin use case?
3. eBPF vs kernel module networking?
4. Career path: Network Engineer → DevOps/SRE?

### KEBUTUHAN PRAKTIKUM
**Topologi:** Semua infrastruktur Minggu 1-13 aktif
**Focus:** Project presentation + peer review

### LANGKAH PRAKTIKUM (2 jam)

**1. Pre-Presentation Checklist (15 menit)**
```bash
# Verify all services running
systemctl status bind9 nginx prometheus grafana suricata

# Check monitoring dashboard
curl http://192.168.1XX.10:3000/api/health

# Verify WireGuard VPN
sudo wg show | grep handshake

# Test DNS resolution
dig @192.168.1XX.10 kelompokXX.lab

# Kubernetes cluster health
kubectl get nodes
kubectl get pods --all-namespaces
```

**2. Project Demo Preparation (20 menit)**
```
Demo Script (15 menit):
├── 1. Network topology overview (2 min)
│   └── Show Grafana dashboard all nodes
├── 2. Core services (3 min)
│   ├── DNS query demo
│   ├── DHCP lease show
│   └── Nginx load balancing curl
├── 3. Containerization (3 min)
│   ├── Docker compose up
│   └── Kubernetes service expose
├── 4. Automation (2 min)
│   └── Ansible playbook execution
├── 5. Security (3 min)
│   ├── nftables rules show
│   ├── Suricata alert demo
│   └── WireGuard remote access
└── 6. Q&A (2 min)
```

**3. Presentation Format (60 menit - semua kelompok)**
```
Setiap kelompok: 15 menit
├── Slides: 5 menit (network design + architecture)
├── Live demo: 8 menit (6 komponen dari prep)
└── Q&A: 2 menit (dosen + peer review)

Peer Review Form:
├── Network design clarity (1-5)
├── Service integration (1-5)
├── Security implementation (1-5)
├── Automation level (1-5)
└── Presentation quality (1-5)
```

**4. Advanced Topic Discussion (15 menit)**
```
Panel Discussion Topics:
1. AIOps adoption di Indonesia
2. Cloud vs On-Premise network management
3. Zero Trust implementation challenge
4. Kubernetes networking production tips
5. Career path: Network Admin → SRE/DevOps
```

**5. Lab Cleanup & Documentation (10 menit)**
```bash
# Backup final configs
mkdir ~/project_final
sudo cp -r /etc/bind/zones ~/project_final/
sudo cp /etc/nginx/sites-available/* ~/project_final/
sudo cp /etc/wireguard/wg0.conf ~/project_final/
kubectl get all -A -o yaml > ~/project_final/k8s-all.yaml

# Export monitoring dashboards
curl http://192.168.1XX.10:3000/api/dashboards/uid/kelompokXX   -o ~/project_final/grafana-dashboard.json

# Git repository
cd ~/project_final
git init
git add .
git commit -m "Final project Kelompok XX"
```

### PROJECT AKHIR REQUIREMENTS (Review Minggu 1-13)

**1. Network Design Document (15 halaman PDF)**
```
Sections:
├── Executive Summary (1 hal)
├── Network Topology Diagram (1 hal)
├── IP Addressing Table (1 hal)
├── Services Architecture (3 hal)
│   ├── DNS/DHCP
│   ├── Web/File Services
│   └── Monitoring Stack
├── Security Implementation (3 hal)
│   ├── Firewall rules
│   ├── VPN access
│   └── IDS/IPS
├── Automation Scripts (2 hal)
├── Troubleshooting Guide (2 hal)
├── Testing Results (1 hal)
└── Conclusion (1 hal)
```

**2. Git Repository Structure**
```
project-kelompokXX/
├── README.md
├── docs/
│   ├── network-design.pdf
│   └── presentation.pptx
├── ansible/
│   ├── inventory.ini
│   ├── dns-playbook.yml
│   └── roles/
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
│   └── networkpolicy.yaml
├── monitoring/
│   ├── prometheus.yml
│   └── grafana-dashboards/
└── scripts/
    ├── backup.sh
    └── health-check.sh
```

**3. Video Demo (5-10 menit)**
```
Recording dengan narasi:
├── 0:00-1:00 → Introduction + topology
├── 1:00-3:00 → Services demo (DNS, Web, File)
├── 3:00-5:00 → Monitoring dashboard walkthrough
├── 5:00-7:00 → Security demo (firewall, IDS, VPN)
├── 7:00-9:00 → Automation execution (Ansible)
└── 9:00-10:00 → Conclusion + lessons learned
```

**4. Technical Presentation (15 menit)**
```
Slide Structure (12 slides):
1. Title + Team
2. Problem Statement
3. Solution Architecture
4. Network Topology
5. Core Services (DNS/DHCP/Web)
6. Monitoring & Observability
7. Containerization (Docker/K8s)
8. Automation (Ansible)
9. Security Implementation
10. Testing Results
11. Challenges & Solutions
12. Q&A
```

### UJI KONFIGURASI PROJECT AKHIR
```
Minimum Requirements Checklist:
☑ 3+ subnets dengan routing OK
☑ DNS forward + reverse zones
☑ DHCP pool + reservations
☑ Web server + reverse proxy + SSL
☑ File sharing NFS/SMB
☑ Prometheus + Grafana (4+ panels)
☑ Docker Compose (3+ containers)
☑ Kubernetes deployment + service
☑ Ansible playbook (3+ tasks)
☑ nftables firewall policy drop
☑ WireGuard VPN tunnel
☑ Suricata IDS dengan custom rules
☑ Dokumentasi lengkap
☑ Video demo 5-10 menit
☑ Git repository organized
```

### RUBRIK PENILAIAN PROJECT (100 poin)

| Komponen | Poin | Kriteria |
|----------|------|----------|
| **Network Design** | 15 | Topology jelas, IP scheme konsisten, routing OK |
| **Core Services** | 20 | DNS/DHCP/Web/File functional + integrasi |
| **Monitoring** | 15 | Prometheus scrape OK, Grafana 4+ panels custom |
| **Automation** | 15 | Ansible playbook idempotent, roles structure |
| **Containerization** | 10 | Docker Compose + K8s deployment working |
| **Security** | 10 | Firewall baseline, VPN access, IDS alerts |
| **Documentation** | 15 | PDF lengkap, Git repo organized, README clear |
| **Presentation** | 10 | Slides professional, demo smooth, Q&A solid |
| **TOTAL** | **100** | |

### PERTANYAAN REFLEKSI MINGGU 14
1. Minggu mana paling challenging dan kenapa?
2. Tools baru apa yang akan dipakai di project pribadi?
3. Skill gap mana yang perlu diperdalam?
4. Bagaimana praktikum ini relate ke career path?

### CHECKLIST TUGAS MINGGU 14
- [ ] Pre-presentation checklist OK
- [ ] Project deliverables lengkap (PDF, Git, Video, Slides)
- [ ] Presentation 15 menit practiced
- [ ] Peer review 3 kelompok lain
- [ ] Final backup configs
- [ ] Lab cleanup

**Waktu Total:** 3 jam (1 teori + 2 presentation)
**Output:** Project akhir production-ready + skill mastery Minggu 1-13

---

## SELAMAT! WORKSHOP ADMINISTRASI JARINGAN SELESAI! 🎉

**Skills Acquired (14 Minggu):**
✅ Linux networking mastery
✅ Service deployment (DNS/DHCP/Web/File)
✅ Monitoring & observability (Prometheus/Grafana)
✅ Containerization (Docker/Kubernetes)
✅ Network automation (Ansible/IaC)
✅ SDN/SD-WAN (WireGuard overlay)
✅ Security hardening (nftables/IDS/Zero Trust)
✅ Troubleshooting systematic approach

**Next Steps:**
- CKA/CKAD Certification (Kubernetes)
- RHCSA/RHCE (Red Hat Automation)
- AWS/Azure/GCP Networking Specialty
- Contribute to open-source network tools
- Build personal homelab for continuous learning

**Good luck di UAS dan karir networking! 🚀**
