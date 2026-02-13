
# MINGGU 8: UJIAN TENGAH SEMESTER (UTS)
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam - Review)
**Materi UTS (Minggu 1-7):**
1. Linux networking (multi-IP, routing, bridge, VLAN, bonding)
2. DNS BIND9 (forward/reverse zones)
3. DHCP ISC (pools, reservations)
4. NTP Chrony (server/client)
5. Nginx reverse proxy + SSL
6. NFS/SMB file sharing
7. Prometheus monitoring

### PERTANYAAN TEORI UTS (30 menit tertulis)
**Pilih 5 dari 8 (Bobot 10%):**
1. Jelaskan DHCP DORA process lengkap dengan port numbers (4 poin)
2. Tulis netplan YAML untuk VLAN subinterface enp1s0.100 (3 poin)
3. Apa fungsi SOA record field dan contoh formatnya? (3 poin)
4. Konfigurasi Nginx upstream least_conn 2 backend (4 poin)
5. Perbedaan NFS export `rw,sync` vs `ro,async` (3 poin)
6. Prometheus scrape vs SNMP polling kelebihan/kekurangan (3 poin)
7. Chrony `allow 192.168.1XX.0/24` fungsi apa? (2 poin)
8. rsync `--delete` di backup script bahaya apa? (2 poin)

### KEBUTUHAN UJIAN PRAKTIK
**Environment:** VM kelompokXX bersih (reset Minggu 1)
**Waktu:** 90 menit
**Tools:** Console Proxmox + SSH

### LANGKAH UJIAN PRAKTIK (90 menit)
**Scenario 1: DNS + DHCP Integration (30 menit)**
```
Tugas: Konfigurasi BIND9 + ISC DHCP dengan DDNS update
Target:
- Zone: ujianXX.lab
- DHCP pool: 192.168.1XX.150-180
- srv2 dapat IP .155 via DHCP
- srv2 resolve ujianXX.lab
```

**Scenario 2: Web Proxy Load Balancing (30 menit)**
```
Tugas: Nginx reverse proxy + SSL + 2 backend
Target:
- Backend srv2:8080, srv3:8080 (Node.js)
- proxy.ujianXX.lab → load balance
- HTTPS certificate valid
```

**Scenario 3: Troubleshooting (30 menit)**
```
Problem 1: srv2 tidak bisa ping gateway → diagnosis + fix
Problem 2: Prometheus target DOWN → port/firewall fix
Problem 3: NFS mount permission denied → exportfs fix
```

### UJI KONFIGURASI UTS
```
**Scenario 1 Success:**
dig srv2.ujianXX.lab → 192.168.1XX.155
dhcpd.leases → lease for srv2 MAC

**Scenario 2 Success:**
curl proxy.ujianXX.lab → alternating srv2/srv3
curl -k https://proxy.ujianXX.lab → Backend response

**Scenario 3 Success:**
3 masalah diidentifikasi + command fix
```

**Deliverables UTS:**
1. **Screenshot 9 gambar** (3 per scenario)
2. **Config files** (`named.conf.local`, `dhcpd.conf`, `nginx proxy`)
3. **Troubleshooting log** (3 masalah + solusi)

### KRITERIA PENILAIAN UTS (25%)
```
Teori Tertulis: 10 poin (2 poin/soal)
Praktik Scenario 1: 5 poin
Praktik Scenario 2: 5 poin
Troubleshooting: 5 poin
**Total: 25 poin**
```

### PERTANYAAN REFLEKSI POST-UTS
1. Scenario tersulit mana dan kenapa?
2. Tools diagnosis paling berguna?
3. Konfigurasi mana paling rawan typo?

### CHECKLIST UTS
```
Teori: 5/5 soal dijawab ✓
Scenario 1: DNS+DHCP working ✓
Scenario 2: Proxy+SSL OK ✓
Scenario 3: 3 troubleshooting ✓
Screenshot 9 gambar ✓
Config files lengkap ✓
```

**Waktu Total UTS:** 3 jam (1 review + 2 praktik)
**Output:** Assessment tengah semester + skill consolidation Minggu 1-7
