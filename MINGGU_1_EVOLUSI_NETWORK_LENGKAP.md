
# MINGGU 1: PENGENALAN ADMINISTRASI JARINGAN MODERN
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam)
**Evolusi Administrasi Jaringan:**
- Traditional: CLI manual, reactive troubleshooting
- Modern 2026: IaC, SDN/NFV, AIOps, Zero Trust

**Lab Infrastructure:**
- Backbone: 10.252.108.0/24 (Internet NAT)
- Kelompok XX: 192.168.1XX.0/24 (VLAN 1XX)
- Mikrotik RB1100: Gateway 192.168.1XX.1
- Proxmox Server: 10.252.108.10

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
              ↓
VM1: 192.168.1XX.10 ← Laptop (SSH)
```

**Hosts:**
- Hostname: kXX-srv1
- IP: 192.168.1XX.10/24
- GW: 192.168.1XX.1
- DNS: 10.252.108.53

### LANGKAH PRAKTIKUM (2 jam)

**1. Akses Proxmox (10 menit)**
```
https://10.252.108.10:8006
User: kelompokXX@pve
```

**2. Install Ubuntu 24.04 (30 menit)**
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
      gateway4: 192.168.1XX.1
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
  traceroute mtr dnsutils nmap tcpdump
```

**5. Test Konektivitas (20 menit)**
```bash
ping -c4 192.168.1XX.1     # Gateway
ping -c4 10.252.108.1      # Backbone
ping -c4 8.8.8.8           # Internet
ping -c4 google.com        # DNS
ssh adminXX@192.168.1XX.10 # Laptop test
```

**6. VM2 & VM3 (15 menit)**
```
kXX-srv2: 192.168.1XX.11 (Ubuntu)
kXX-srv3: 192.168.1XX.12 (Rocky Linux 9)
```

### UJI KONFIGURASI
```
Expected Output:
$ ip addr show enp1s0
... inet 192.168.1XX.10/24

$ ip route
default via 192.168.1XX.1

$ ping google.com
PING google.com (142.250.190.78) 56 bytes
```

**Screenshot Wajib:**
1. Proxmox login + VM list
2. `ip addr` + `ip route`
3. Ping tests (4 screenshot)
4. SSH session dari laptop

### PERTANYAAN SEKITAR PRAKTIKUM
1. Apa yang terjadi jika gateway salah di netplan?
2. Kenapa DNS lab (10.252.108.53) lebih prioritas dari 8.8.8.8?
3. Jika ping backbone gagal, kemungkinan penyebab VLAN?
4. Bedakan `netplan generate` vs `netplan apply`?

### CHECKLIST TUGAS
- [ ] VM1 Ubuntu installed
- [ ] Static IP working
- [ ] Internet access OK
- [ ] SSH dari laptop OK
- [ ] VM2 & VM3 ready
- [ ] 5 screenshot lab report

**Waktu Total:** 2 jam 30 menit
**Output:** Environment siap untuk 13 minggu praktikum selanjutnya
