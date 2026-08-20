# MINGGU 8: UJIAN TENGAH SEMESTER (UTS)
## WORKSHOP ADMIN JARINGAN - PENS TI 2026

### DASAR TEORI (1 jam - Review)
**Materi UTS (Minggu 1-7):**
1. Linux networking
2. DNS BIND9
3. DHCP + Chrony
4. Nginx reverse proxy
5. NFS/SMB
6. Prometheus + Grafana

### PERTANYAAN TEORI UTS
1. Jelaskan DHCP DORA dan port yang digunakan.
2. Tulis contoh konfigurasi VLAN subinterface Linux.
3. Apa fungsi SOA record?
4. Jelaskan Nginx upstream dengan dua backend.
5. Perbedaan NFS `rw,sync` dan `ro,async`.
6. Prometheus scrape vs SNMP polling.
7. Fungsi Chrony `allow`.
8. Risiko penggunaan `rsync --delete`.

### KEBUTUHAN UJIAN PRAKTIK
**Environment:** dua VM kelompok (`srv1` dan `srv2`)
**Waktu:** 90 menit
**Tools:** Console Proxmox + SSH

### LANGKAH UJIAN PRAKTIK

**Scenario 1: DNS + DHCP Integration (30 menit)**
```
Target:
- BIND9 pada srv1
- Zone: ujianXX.lab
- DHCP pool: 192.168.1XX.150-180
- srv2 menjadi DHCP client/reservation
- srv2 dapat resolve ujianXX.lab
```

**Scenario 2: Web Proxy Load Balancing (30 menit)**
```
Target:
- Nginx reverse proxy pada srv1
- Backend A srv2:8081
- Backend B srv2:8082
- proxy.ujianXX.lab melakukan load balancing
- TLS internal aktif
```

**Scenario 3: Troubleshooting (30 menit)**
```
Problem 1: srv2 tidak dapat mencapai gateway
Problem 2: Prometheus target srv2 DOWN
Problem 3: NFS mount permission denied
```

### UJI KONFIGURASI UTS
```
Scenario 1:
dig srv2.ujianXX.lab → alamat srv2
dhcpd.leases/reservation → sesuai

Scenario 2:
curl proxy.ujianXX.lab → respons backend-A/backend-B
curl -k https://proxy.ujianXX.lab → HTTPS response

Scenario 3:
3 masalah diidentifikasi dan diperbaiki dengan bukti command/output.
```

### KRITERIA PENILAIAN UTS
- Teori: 10 poin
- Scenario 1: 5 poin
- Scenario 2: 5 poin
- Troubleshooting: 5 poin

### CHECKLIST UTS
- [ ] DNS + DHCP working
- [ ] Reverse proxy + dua backend working
- [ ] TLS lab working
- [ ] 3 troubleshooting selesai
- [ ] Config files dan evidence dikumpulkan

**Output:** Assessment tengah semester pada baseline lab dua node.
