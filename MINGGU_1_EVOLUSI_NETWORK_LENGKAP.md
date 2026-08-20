# MINGGU 1: PENGENALAN ADMINISTRASI JARINGAN MODERN
## WORKSHOP ADMIN JARINGAN - PENS TI 2026

### DASAR TEORI (1 jam)
**Evolusi Administrasi Jaringan:**
- Traditional: CLI manual, reactive troubleshooting
- Modern 2026: IaC, SDN/NFV, AIOps, Zero Trust

**Lab Infrastructure:**
- Backbone: 10.252.108.0/24 (Internet NAT)
- Kelompok XX: 192.168.1XX.0/24 (VLAN 1XX)
- Mikrotik RB1100: Gateway 192.168.1XX.1
- Proxmox Server: 10.252.108.10
- 2 VM per kelompok:
  - kXX-srv1: infrastructure/control-plane
  - kXX-srv2: workload/data-plane

### PERTANYAAN TEORI
1. Apa beda CLI manual vs Ansible?
2. Kenapa butuh VLAN isolasi?
3. Fungsi NAT di 10.252.108.1?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
Internet ← NAT ← [10.252.108.1]
              ↓
[Mikrotik Switch VLAN 1XX]
              ↓
[RB1100] 192.168.1XX.1
        ├── srv1: 192.168.1XX.10
        ├── srv2: 192.168.1XX.11
        └── Laptop mahasiswa
```

**Hosts:**
- kXX-srv1: 192.168.1XX.10/24
- kXX-srv2: 192.168.1XX.11/24
- GW: 192.168.1XX.1
- DNS awal: 10.252.108.53

### LANGKAH PRAKTIKUM (2 jam)

**1. Akses Proxmox (10 menit)**
```
https://10.252.108.10:8006
User: kelompokXX@pve
```

**2. Install Ubuntu 24.04 pada srv1 (30 menit)**
- Console VM → Install Ubuntu Server
- Network: 192.168.1XX.10/24, GW 192.168.1XX.1
- Username: adminXX
- OpenSSH ✓

**3. Netplan Static IP (15 menit)**
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```
```yaml
network:
  version: 2
  ethernets:
    enp1s0:
      dhcp4: no
      addresses: [192.168.1XX.10/24]
      routes:
        - to: default
          via: 192.168.1XX.1
      nameservers:
        addresses: [10.252.108.53, 8.8.8.8]
```
```bash
sudo netplan apply
```

**4. Install Tools (10 menit)**
```bash
sudo apt update
sudo apt install -y vim htop git curl wget net-tools \
  traceroute mtr dnsutils nmap tcpdump iperf3
```

**5. Test Konektivitas (20 menit)**
```bash
ping -c4 192.168.1XX.1     # Gateway
ping -c4 10.252.108.1      # Backbone
ping -c4 8.8.8.8           # Internet
ping -c4 google.com        # DNS
ssh adminXX@192.168.1XX.10 # Laptop test
```

**6. Siapkan srv2 (20 menit)**
```
kXX-srv2: 192.168.1XX.11/24
Gateway: 192.168.1XX.1
OS: Ubuntu Server 24.04
```

Ulangi konfigurasi dasar srv1 pada srv2, lalu verifikasi konektivitas dua arah:
```bash
# dari srv1
ping -c4 192.168.1XX.11

# dari srv2
ping -c4 192.168.1XX.10
```

### UJI KONFIGURASI
```
Expected Output:
$ ip addr show enp1s0
... inet 192.168.1XX.10/24

$ ip route
default via 192.168.1XX.1

$ ping google.com
PING google.com (...) 56 bytes
```

**Screenshot Wajib:**
1. Proxmox login + 2 VM kelompok
2. `ip addr` + `ip route` srv1
3. Ping gateway/backbone/internet
4. SSH session dari laptop
5. Ping srv1 ↔ srv2

### PERTANYAAN SEKITAR PRAKTIKUM
1. Apa yang terjadi jika gateway salah di netplan?
2. Kenapa DNS lab (10.252.108.53) lebih prioritas dari 8.8.8.8?
3. Jika ping backbone gagal, kemungkinan penyebab VLAN?
4. Bedakan `netplan generate` vs `netplan apply`?

### CHECKLIST TUGAS
- [ ] srv1 Ubuntu installed
- [ ] srv2 Ubuntu installed
- [ ] Static IP kedua VM working
- [ ] Internet access OK
- [ ] SSH dari laptop OK
- [ ] srv1 dan srv2 saling terhubung
- [ ] 5 screenshot lab report

**Waktu Total:** 2 jam 30 menit
**Output:** Environment 2-node siap untuk 13 minggu praktikum selanjutnya
