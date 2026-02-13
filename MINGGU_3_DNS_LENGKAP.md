
# MINGGU 3: DNS SERVICES
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam)
**DNS Architecture:**
```
Client ← UDP/TCP 53 → Recursive Resolver ← Root Servers
                                            ↓
                                        TLD (.com, .id)
                                            ↓
                                        Authoritative NS
```

**BIND9 Components:**
- `named`: DNS daemon
- Zone files: Master data
- named.conf: Configuration

**Record Types:**
| Type | Purpose | Example |
|------|---------|---------|
| A | IPv4 | srv1 192.168.1XX.10 |
| AAAA | IPv6 | srv1 2001:db8::10 |
| CNAME | Alias | www srv2 |
| MX | Mail | mail 10 srv1 |
| PTR | Reverse | 10.1XX.168.192 srv1 |

### PERTANYAAN TEORI
1. Bedakan recursive vs authoritative DNS?
2. Apa fungsi SOA record field 'Serial'?
3. Kenapa DNSSEC penting di 2026?
4. Kapan pakai DNS forwarding vs root hints?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
Laptop ← dig → kXX-srv1:53 (BIND9 Master)
                    ↓ SOA
              Zone kelompokXX.lab
                    ↓
Forward: 10.252.108.53 (Lab DNS)
         8.8.8.8 (Public)
```

**Hosts:**
```
DNS Server: kXX-srv1 192.168.1XX.10
Client: kXX-srv2 192.168.1XX.11
Laptop: SSH + dig/nslookup
```

**Aplikasi:**
```
BIND9, dnsutils, nano/vim
```

### LANGKAH PRAKTIKUM (2 jam)

**1. BIND9 Installation (10 menit)**
```bash
sudo apt update
sudo apt install -y bind9 bind9utils dnsutils
sudo systemctl enable bind9
sudo systemctl start bind9
sudo systemctl status bind9
```

**2. Directory Structure (5 menit)**
```bash
sudo mkdir -p /etc/bind/zones
sudo chown -R bind:bind /etc/bind/zones
sudo chmod 755 /etc/bind/zones
```

**3. Forward Zone Configuration (25 menit)**
```bash
sudo nano /etc/bind/named.conf.local
```
```
zone "kelompokXX.lab" {
    type master;
    file "/etc/bind/zones/db.kelompokXX";
};

zone "1XX.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.1XX";
};
```

**4. Forward Zone File (20 menit)**
```bash
sudo nano /etc/bind/zones/db.kelompokXX
```
```
\$TTL    3600
@       IN      SOA     ns1.kelompokXX.lab. admin.kelompokXX.lab. (
                        2026021301         ; Serial
                                 3600         ; Refresh
                                  1800         ; Retry
                                604800         ; Expire
                                  3600 )       ; Negative Cache TTL
;
@       IN      NS      ns1
ns1     IN      A       192.168.1XX.10
srv1    IN      A       192.168.1XX.10
srv2    IN      A       192.168.1XX.11
www     IN      CNAME   srv2
mail    IN      MX 10   srv1
```

**5. Reverse Zone File (15 menit)**
```bash
sudo nano /etc/bind/zones/db.1XX
```
```
\$TTL    3600
@       IN      SOA     ns1.kelompokXX.lab. admin.kelompokXX.lab. (
                              2026021301 ; Serial
                                     3600 ; Refresh
                                      1800 ; Retry
                                  604800 ; Expire
                                     3600 ); Negative Cache TTL
;
@       IN      NS     ns1.kelompokXX.lab.
10      IN      PTR    srv1.kelompokXX.lab.
11      IN      PTR    srv2.kelompokXX.lab.
```

**6. Syntax Check & Restart (10 menit)**
```bash
sudo named-checkconf
sudo named-checkzone kelompokXX.lab /etc/bind/zones/db.kelompokXX
sudo named-checkzone 1XX.168.192.in-addr.arpa /etc/bind/zones/db.1XX
sudo systemctl restart bind9
sudo systemctl status bind9
```

**7. Client Configuration srv2 (10 menit)**
```bash
# srv2 netplan
sudo nano /etc/netplan/50-cloud-init.yaml
# nameservers: addresses: [192.168.1XX.10]
sudo netplan apply
```

**8. Testing Suite (25 menit)**
```bash
# Basic resolution
dig @192.168.1XX.10 srv1.kelompokXX.lab
dig @192.168.1XX.10 srv2.kelompokXX.lab
dig @192.168.1XX.10 www.kelompokXX.lab

# Reverse lookup
dig @192.168.1XX.10 -x 192.168.1XX.10
dig @192.168.1XX.10 -x 192.168.1XX.11

# Client test (dari srv2)
dig srv1.kelompokXX.lab
nslookup srv2.kelompokXX.lab
host www.kelompokXX.lab 192.168.1XX.10

# Port verification
sudo ss -tulpn | grep :53
sudo netstat -tulpn | grep :53
```

### UJI KONFIGURASI
```
**Expected dig srv1.kelompokXX.lab @192.168.1XX.10:**
;; ANSWER SECTION:
srv1.kelompokXX.lab. 3600 IN A 192.168.1XX.10

**Expected dig -x 192.168.1XX.10 @192.168.1XX.10:**
10.1XX.168.192.in-addr.arpa. 3600 IN PTR srv1.kelompokXX.lab.
```

**Screenshot Wajib (8 gambar):**
1. `named.conf.local`
2. Forward zone file
3. Reverse zone file
4. `named-checkzone` OK
5. `systemctl status bind9`
6. `dig srv1` forward
7. `dig -x` reverse
8. `ss -tulpn :53`

### PERTANYAAN SEKITAR PRAKTIKUM
1. Jika `named-checkzone` error, kemungkinan syntax SOA?
2. Apa akibat jika Serial number tidak naik?
3. Kenapa TTL 3600 bukan 86400 di lab?
4. Jika client tetap pakai 8.8.8.8, solusi?

### CHECKLIST TUGAS MINGGU 3
- [ ] BIND9 installed & running
- [ ] Forward zone OK
- [ ] Reverse zone OK
- [ ] Client srv2 resolve internal
- [ ] 8 screenshot lengkap
- [ ] named-checkzone PASS

**Waktu Total:** 2 jam
**Output:** Authoritative DNS server untuk domain kelompok
