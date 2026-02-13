
# MINGGU 4: DHCP & NETWORK TIME SERVICES
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam)
**DHCP Protocol Flow:**
```
DISCOVER (broadcast) → OFFER → REQUEST → ACK
Client gets: IP, subnet, gateway, DNS, lease time
```

**DHCP Message Types:**
| Packet | Source | Dest | Purpose |
|--------|--------|------|---------|
| DISCOVER | 0.0.0.0:68 | 255.255.255.255:67 | IP request |
| OFFER | Server IP:67 | Client MAC:68 | IP proposal |
| REQUEST | 0.0.0.0:68 | Broadcast | Accept offer |
| ACK | Server IP:67 | Client:68 | Lease granted |

**NTP/Chrony:**
- NTP: Network Time Protocol (RFC 5905)
- Chrony: Modern NTP client/server (faster convergence)

### PERTANYAAN TEORI
1. Apa fungsi DHCP relay agent?
2. Kenapa lease time 1 jam vs 24 jam di lab?
3. Bedakan NTP stratum 1 vs stratum 3?
4. Risiko time drift >5 detik?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
DHCP Client (srv2) ← UDP 68 → srv1:67 (DHCP Server)
NTP Client (srv2) ← UDP 123 → srv1:123 (Chrony Master)
                     ↓ DNS integration
                 srv1:53 (BIND9 Minggu 3)
```

**Hosts:**
```
DHCP/NTP Server: kXX-srv1 192.168.1XX.10
DHCP Client: kXX-srv2 192.168.1XX.11
```

**Aplikasi:**
```
isc-dhcp-server / dnsmasq, chrony, nano
```

### LANGKAH PRAKTIKUM (2 jam)

**1. DHCP Server Installation (10 menit)**
```bash
sudo apt install -y isc-dhcp-server
sudo systemctl enable isc-dhcp-server
```

**2. DHCP Configuration (25 menit)**
```bash
sudo nano /etc/default/isc-dhcp-server
```
```
INTERFACESv4="enp1s0"
```

```bash
sudo nano /etc/dhcp/dhcpd.conf
```
```
# Global options
default-lease-time 3600;
max-lease-time 7200;
authoritative;

# Subnet declaration
subnet 192.168.1XX.0 netmask 255.255.255.0 {
  range 192.168.1XX.100 192.168.1XX.199;
  option routers 192.168.1XX.1;
  option domain-name-servers 192.168.1XX.10, 10.252.108.53;
  option domain-name "kelompokXX.lab";
  option subnet-mask 255.255.255.0;

  # Reservation contoh
  host srv2 {
    hardware ethernet 52:54:00:XX:XX:XX;  # MAC srv2
    fixed-address 192.168.1XX.11;
  }
}
```

**3. Restart & Check Config (10 menit)**
```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf
cat /var/lib/dhcp/dhcpd.leases
```

**4. Client DHCP Test srv2 (15 menit)**
```bash
# srv2: switch to DHCP
sudo nano /etc/netplan/50-cloud-init.yaml
```
```yaml
network:
  ethernets:
    enp1s0:
      dhcp4: yes
```
```bash
sudo netplan apply
ip addr show enp1s0
```

**5. Chrony NTP Server (20 menit)**
```bash
sudo apt install chrony
sudo nano /etc/chrony/chrony.conf
```
```
# Server config
server  pool.ntp.org iburst
allow 192.168.1XX.0/24
keyfile /etc/chrony.keys
driftfile /var/lib/chrony/chrony.drift
rtcsync
logdir /var/log/chrony
```

**6. Chrony Client srv2 (15 menit)**
```bash
# srv2 chrony.conf
sudo nano /etc/chrony/chrony.conf
```
```
server 192.168.1XX.10 iburst
```

**7. NTP Verification (15 menit)**
```bash
# Server status
sudo chronyc tracking
sudo chronyc sources
sudo chronyc sourcestats

# Client sync
sudo systemctl restart chrony
sudo chronyc tracking  # srv2
timedatectl status
```

**8. DDNS Integration (Opsional - bonus 10 menit)**
```bash
# dhcpd.conf tambah:
ddns-update-style interim;
dhcpd-updates /var/lib/dhcp/dhcpd-updates;
```

### UJI KONFIGURASI
```
**DHCP srv2 expected:**
ip addr show: inet 192.168.1XX.1YY/24 (.100-.199)
cat /var/lib/dhcp/dhcpd.leases: lease for srv2 MAC

**Chrony expected:**
chronyc tracking:
Reference ID: POOL.NTP.ORG
Stratum: 3
Root delay: 0.023s
```

**Screenshot Wajib (9 gambar):**
1. dhcpd.conf subnet
2. dhcpd.leases file
3. srv2 `ip addr` DHCP
4. chrony.conf server
5. `chronyc sources` server
6. srv2 `chronyc tracking`
7. `timedatectl status`
8. `systemctl status isc-dhcp-server`
9. `systemctl status chrony`

### PERTANYAAN SEKITAR PRAKTIKUM
1. Jika srv2 tidak dapat IP DHCP, cek interface dhcpd.conf?
2. Apa beda `allow 192.168.1XX.0/24` vs `allow all` di chrony?
3. Jika time drift 10 menit, impact ke Kerberos/SSL?
4. Solusi jika DHCP pool habis?

### CHECKLIST TUGAS MINGGU 4
- [ ] DHCP server + pool dynamic
- [ ] Static reservation srv2
- [ ] Chrony server + client sync
- [ ] srv2 dapat IP via DHCP
- [ ] 9 screenshot lengkap

**Waktu Total:** 2 jam
**Output:** DHCP+NTP infrastructure untuk subnet kelompok
