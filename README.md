# 📚 Buku Petunjuk Praktikum - Workshop Administrasi Jaringan

**Politeknik Elektronika Negeri Surabaya (PENS)**  
**Program Studi Teknik Informatika - Tahun Akademik 2026**

[![Lab Infrastructure](https://img.shields.io/badge/Lab-10%20Kelompok-blue)](./TOPOLOGI_MERMAID_LENGKAP.md)
[![Semester Duration](https://img.shields.io/badge/Duration-14%20Minggu-green)]()
[![Tech Stack](https://img.shields.io/badge/Stack-Linux%20%7C%20Docker%20%7C%20K8s-orange)]()

---

## 🎯 Deskripsi

Workshop Administrasi Jaringan adalah mata kuliah praktikum **3 SKS** (1 teori + 2 praktik) yang mengajarkan administrasi jaringan modern dari fundamental hingga advanced topics meliputi:

- 🖧 Linux networking (routing, bridging, VLAN)
- 🌐 Core services (DNS, DHCP, Web, File)
- 📊 Monitoring & observability (Prometheus, Grafana)
- 🐳 Containerization (Docker, Kubernetes)
- ⚙️ Network automation (Ansible, IaC)
- 🔐 Security & Zero Trust (nftables, VPN, IDS)
- 🌀 SDN/SD-WAN (WireGuard overlay)

---

## 🏗️ Infrastruktur Lab

### Hardware
- 1x Mikrotik Manageable Switch 24-port
- 10x Mikrotik RB1100 (Router per kelompok)
- 10x Mikrotik RB3011 (Router backup)
- 1x Server MSI Xeon 40-core, 64GB RAM (Proxmox)

### Network Design
- **Backbone:** `10.252.108.0/24` (Internet NAT access)
- **Kelompok 01-10:** `192.168.101-110.0/24` (VLAN isolated)
- **3 VM per kelompok:** Ubuntu Server 24.04 + Rocky Linux 9

📐 **[Lihat Topologi Lengkap](./TOPOLOGI_MERMAID_LENGKAP.md)** (13 diagram Mermaid)

---

## 📖 Modul Praktikum Per Minggu

| Minggu | Topik | File Modul | Highlight |
|--------|-------|------------|-----------|
| **1** | Setup Environment Lab | [MINGGU_1_PRAKTIKUM_LENGKAP.md](./MINGGU_1_EVOLUSI_NETWORK_LENGKAP.md) | Proxmox, Ubuntu install, SSH, networking |
| **2** | Linux Network Admin | [MINGGU_2_PRAKTIKUM_LENGKAP.md](./MINGGU_2_LINUX_STACK_LENGKAP.md) | Multi-IP, routing, bonding, bridge, VLAN |
| **3** | DNS Services | [MINGGU_3_DNS_LENGKAP.md](./MINGGU_3_DNS_LENGKAP.md) | BIND9 forward/reverse zones |
| **4** | DHCP & NTP | [MINGGU_4_DHCP_NTP_LENGKAP.md](./MINGGU_4_DHCP_NTP_LENGKAP.md) | ISC DHCP pools, Chrony time sync |
| **5** | Web Services & Proxy | [MINGGU_5_WEB_LENGKAP.md](./MINGGU_5_WEB_LENGKAP.md) | Nginx reverse proxy, SSL, load balancing |
| **6** | File Services & Storage | [MINGGU_6_FILE_SERVICES_LENGKAP.md](./MINGGU_6_FILE_SERVICES_LENGKAP.md) | NFS, Samba, rsync backup automation |
| **7** | Monitoring & Observability | [MINGGU_7_MONITORING_LENGKAP.md](./MINGGU_7_MONITORING_LENGKAP.md) | Prometheus, node_exporter, Grafana |
| **8** | **UTS (Ujian Tengah Semester)** | [MINGGU_8_UTS_LENGKAP.md](./MINGGU_8_UTS_LENGKAP.md) | 3 scenario praktik + troubleshooting |
| **9** | Container Networking | [MINGGU_9_DOCKER_LENGKAP.md](./MINGGU_9_DOCKER_LENGKAP.md) | Docker bridge/host/macvlan, Compose |
| **10** | Kubernetes Networking | [MINGGU_10_K8S_LENGKAP.md](./MINGGU_10_K8S_LENGKAP.md) | K3s cluster, Calico CNI, NetworkPolicy |
| **11** | Network Automation | [MINGGU_11_ANSIBLE_LENGKAP.md](./MINGGU_11_ANSIBLE_LENGKAP.md) | Ansible playbooks, roles, inventory |
| **12** | SDN/SD-WAN | [MINGGU_12_SDN_WIREGUARD_LENGKAP.md](./MINGGU_12_SDN_WIREGUARD_LENGKAP.md) | WireGuard overlay, policy routing |
| **13** | Network Security | [MINGGU_13_SECURITY_LENGKAP.md](./MINGGU_13_SECURITY_LENGKAP.md) | nftables, Suricata IDS, fail2ban |
| **14** | **Project Presentation** | [MINGGU_14_FINAL_PROJECT_LENGKAP.md](./MINGGU_14_FINAL_PROJECT_LENGKAP.md) | Demo infrastruktur lengkap + AIOps |

---

## 📋 Format Modul (Standar Politeknik)

Setiap file modul berisi:
1. ✅ **Dasar Teori** (1 jam kuliah)
2. ✅ **Pertanyaan Teori** (4 soal diskusi)
3. ✅ **Kebutuhan Praktikum** (topologi + tools + hosts)
4. ✅ **Langkah Praktikum** (6-8 step hands-on 2 jam)
5. ✅ **Uji Konfigurasi** (expected output + screenshot wajib)
6. ✅ **Pertanyaan Post-Praktikum** (4 refleksi)
7. ✅ **Checklist Tugas** (deliverables)

---

## 🎓 Project Akhir (Minggu 14)

### Requirements
- ✅ Network design 3+ subnets
- ✅ Core services (DNS, DHCP, Web, File)
- ✅ Prometheus + Grafana monitoring (4+ panels)
- ✅ Ansible automation deployment
- ✅ Docker Compose (3+ containers)
- ✅ Kubernetes deployment + service
- ✅ Security (nftables, VPN, IDS)

### Deliverables
1. **PDF Documentation** (15 halaman)
2. **Git Repository** (configs + playbooks + docker)
3. **Grafana Dashboards** (screenshots)
4. **Video Demo** (5-10 menit)
5. **Technical Presentation** (15 menit)

**Rubrik Penilaian:** 100 poin (detail di [Minggu 14](./MINGGU_14_FINAL_PROJECT_LENGKAP.md))

---

## 🛠️ Tech Stack

