
# MINGGU 13: NETWORK SECURITY & ZERO TRUST
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam)
**Zero Trust Architecture:**
```
Traditional: Trust internal network
Zero Trust: "Never trust, always verify"

Principles:
├── Verify explicitly (MFA, context)
├── Least privilege access
├── Assume breach (micro-segmentation)
└── Continuous monitoring
```

**Security Layers:**
```
Layer 7: Application (WAF)
Layer 4: Transport (Firewall rules)
Layer 3: Network (Segmentation)
Layer 2: Data link (802.1X)
```

**Tools:**
- **nftables**: Modern Linux firewall (replaces iptables)
- **WireGuard**: Secure VPN (already from Minggu 12)
- **Suricata**: Network IDS/IPS

### PERTANYAAN TEORI
1. Zero Trust vs perimeter security model?
2. nftables vs iptables syntax/performance?
3. IDS vs IPS mode deployment?
4. WireGuard vs OpenVPN security audit?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
DMZ Zone (srv2) ← nftables → Internal (srv3)
        ↓ WireGuard VPN          ↓ Suricata IDS
    Laptop Remote           srv1 (Monitor)
```

**Hosts:**
```
Firewall/IDS: kXX-srv1 192.168.1XX.10
DMZ Web: kXX-srv2 192.168.1XX.11
Internal DB: kXX-srv3 192.168.1XX.12
```

**Aplikasi:**
```
nftables, wireguard, suricata, fail2ban
```

### LANGKAH PRAKTIKUM (2 jam)

**1. nftables Firewall Baseline (25 menit)**
```bash
# srv1: Install nftables
sudo apt install nftables
sudo systemctl enable nftables
sudo systemctl start nftables

# Backup existing rules
sudo nft list ruleset > /tmp/nft-backup.conf
```
```bash
sudo nano /etc/nftables.conf
```
```
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;

        # Allow loopback
        iif lo accept

        # Allow established/related
        ct state established,related accept

        # Allow SSH (rate limit)
        tcp dport 22 ct state new limit rate 3/minute accept

        # Allow WireGuard
        udp dport 51820 accept

        # Allow Prometheus/Grafana
        tcp dport { 9090, 3000, 9100 } accept

        # Allow DNS
        tcp dport 53 accept
        udp dport 53 accept

        # Drop invalid
        ct state invalid drop

        # Log dropped
        limit rate 5/minute log prefix "nftables-drop: "
    }

    chain forward {
        type filter hook forward priority 0; policy drop;

        # Allow WireGuard tunnel
        iifname wg0 accept
        oifname wg0 accept

        # Allow established
        ct state established,related accept
    }

    chain output {
        type filter hook output priority 0; policy accept;
    }
}

# NAT for WireGuard
table ip nat {
    chain postrouting {
        type nat hook postrouting priority 100; policy accept;
        oifname enp1s0 masquerade
    }
}
```
```bash
sudo nft -f /etc/nftables.conf
sudo nft list ruleset
```

**2. Advanced Security Rules (20 menit)**
```bash
# Geo-blocking simulation (block range)
sudo nft add rule inet filter input ip saddr 192.0.2.0/24 drop

# Port scan detection
sudo nft add rule inet filter input tcp flags syn / syn,ack ct state new \
    limit rate over 10/second drop

# SYN flood protection
sudo nft add rule inet filter input tcp flags syn tcp dport 80 \
    limit rate 25/second accept
```

**3. WireGuard VPN Access (sudah dari Minggu 12 - 15 menit)**
```bash
# Verify WireGuard running
sudo wg show

# Add laptop as remote peer
LAPTOP_PUBKEY="<laptop_wireguard_public_key>"
sudo wg set wg0 peer $LAPTOP_PUBKEY allowed-ips 10.10.0.4/32

# Test remote access
ping 10.10.0.1  # From laptop
ssh adminXX@10.10.0.1  # Secure SSH via VPN
```

**4. Suricata IDS Installation (25 menit)**
```bash
# srv1
sudo apt install suricata suricata-update
sudo suricata-update
sudo systemctl enable suricata
```
```bash
sudo nano /etc/suricata/suricata.yaml
```
```yaml
# Edit interface
af-packet:
  - interface: enp1s0
    threads: 2
    cluster-id: 99

# HOME_NET
vars:
  address-groups:
    HOME_NET: "[192.168.1XX.0/24]"
    EXTERNAL_NET: "!$HOME_NET"
```
```bash
sudo systemctl start suricata
sudo systemctl status suricata
```

**5. Custom Suricata Rules (20 menit)**
```bash
sudo nano /etc/suricata/rules/local.rules
```
```
# Alert ICMP flood
alert icmp any any -> $HOME_NET any (msg:"ICMP Flood Detected"; \
    threshold: type both, track by_src, count 10, seconds 5; sid:1000001;)

# Alert SSH brute force
alert tcp any any -> $HOME_NET 22 (msg:"SSH Brute Force Attempt"; \
    flags:S; threshold: type both, track by_src, count 5, seconds 60; sid:1000002;)

# Alert SQL injection attempt
alert http any any -> $HOME_NET any (msg:"Possible SQL Injection"; \
    content:"UNION SELECT"; nocase; sid:1000003;)

# Alert DNS tunneling
alert dns any any -> any any (msg:"DNS Query Excessive Length"; \
    dsize:>512; sid:1000004;)
```
```bash
sudo systemctl restart suricata
sudo tail -f /var/log/suricata/fast.log
```

**6. Testing & Attack Simulation (20 menit)**
```bash
# Test 1: Port scan detection (dari srv2)
nmap -sS 192.168.1XX.10  # Should trigger alert

# Test 2: ICMP flood
ping -f 192.168.1XX.10  # Fast ping

# Test 3: SSH brute force simulation
for i in {1..10}; do ssh fake@192.168.1XX.10; done

# Check logs
sudo tail -20 /var/log/suricata/fast.log
sudo journalctl -u nftables | grep drop
```

**7. Fail2ban Integration (15 menit)**
```bash
sudo apt install fail2ban
sudo nano /etc/fail2ban/jail.local
```
```
[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
```
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo fail2ban-client status sshd
```

### UJI KONFIGURASI
```
**nftables:**
sudo nft list ruleset | grep policy
chain input { ... policy drop; }

**Suricata:**
sudo suricatasc -c "dump-counters" | grep alerts
alerts: 15

**WireGuard VPN:**
sudo wg show | grep handshake
latest handshake: 5 seconds ago
```

**Screenshot Wajib (14 gambar):**
1. `/etc/nftables.conf` full rules
2. `nft list ruleset`
3. WireGuard peer laptop
4. `wg show` handshake
5. Suricata.yaml config
6. local.rules custom
7. `systemctl status suricata`
8. fast.log alerts
9. nmap scan detection
10. SSH brute force alert
11. fail2ban status
12. `fail2ban-client status sshd`
13. nftables drop log
14. Grafana security dashboard (optional)

### PERTANYAAN SEKITAR PRAKTIKUM
1. Policy drop vs accept default tradeoff?
2. Suricata IDS vs IPS mode deployment?
3. Rate limiting vs connection tracking?
4. Zero Trust micro-segmentation implementation?

### CHECKLIST TUGAS MINGGU 13
- [ ] nftables baseline policy drop
- [ ] WireGuard remote access laptop
- [ ] Suricata IDS + custom rules
- [ ] Attack simulation detected
- [ ] fail2ban SSH protection
- [ ] 14 screenshot lengkap

**Waktu Total:** 2 jam
**Output:** Production-grade network security stack dengan Zero Trust principles
