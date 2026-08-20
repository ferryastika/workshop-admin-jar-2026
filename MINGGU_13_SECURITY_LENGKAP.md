# MINGGU 13: NETWORK SECURITY & ZERO TRUST
## WORKSHOP ADMIN JARINGAN - PENS TI 2026

### DASAR TEORI (1 jam)
**Security baseline workshop:**
```
Laptop / test traffic
        ↓
   srv1 (.10)
 nftables + Suricata + WireGuard
        ↓
   srv2 (.11)
 backend / protected workload
```

Desain ini tidak mencoba meniru DMZ production secara penuh. Tujuan praktikum adalah memahami policy enforcement, VPN access, IDS, logging, dan response menggunakan dua host yang sudah tersedia.

### PERTANYAAN TEORI
1. Zero Trust vs perimeter security model?
2. nftables vs iptables?
3. IDS vs IPS?
4. Apa keterbatasan menjalankan firewall dan IDS pada host yang sama dalam teaching lab?

### KEBUTUHAN PRAKTIKUM
**Hosts:**
```
Security Gateway/Monitor: kXX-srv1 192.168.1XX.10
Protected Workload:        kXX-srv2 192.168.1XX.11
Optional VPN peer:         Laptop mahasiswa
```

### LANGKAH PRAKTIKUM (2 jam)

**1. nftables Baseline pada srv1 (25 menit)**
```bash
sudo apt update
sudo apt install -y nftables
sudo systemctl enable --now nftables
sudo nft list ruleset > /tmp/nft-before.conf
```

Gunakan baseline `/etc/nftables.conf`:
```nft
#!/usr/sbin/nft -f
flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        iif lo accept
        ct state established,related accept
        ip protocol icmp accept
        tcp dport 22 accept
        tcp dport { 53, 80, 443, 3000, 9090, 9100 } accept
        udp dport { 53, 51820 } accept
        ct state invalid drop
        limit rate 5/minute log prefix "nft-drop: "
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
        ct state established,related accept
        iifname "wg0" accept
        oifname "wg0" accept
    }

    chain output {
        type filter hook output priority 0; policy accept;
    }
}
```

```bash
sudo nft -c -f /etc/nftables.conf
sudo nft -f /etc/nftables.conf
sudo nft list ruleset
```

**2. Protected Workload pada srv2 (15 menit)**
Gunakan backend dari Minggu 5 atau jalankan service sederhana:
```bash
python3 -m http.server 8081 --bind 0.0.0.0
```
Uji akses langsung dan diskusikan aturan mana yang seharusnya diterapkan pada srv1 bila trafik dirutekan melalui gateway/VPN.

**3. WireGuard Remote Access (20 menit)**
Gunakan tunnel Minggu 12. Tambahkan laptop sebagai peer opsional pada srv1:
```ini
[Peer]
PublicKey = <LAPTOP_PUBLIC_KEY>
AllowedIPs = 10.10.0.10/32
```
Verifikasi handshake dan SSH melalui alamat WireGuard.

**4. Install Suricata pada srv1 (25 menit)**
```bash
sudo apt install -y suricata suricata-update
sudo suricata-update
sudo systemctl enable suricata
```
Set `HOME_NET` ke subnet kelompok dan interface ke interface lab yang benar, lalu:
```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
sudo systemctl restart suricata
```

**5. Custom Detection Rule (20 menit)**
Tambahkan rule sederhana yang aman untuk lab, misalnya mendeteksi ICMP atau HTTP test pattern. Contoh:
```text
alert icmp any any -> $HOME_NET any (msg:"LAB ICMP detected"; sid:1000001; rev:1;)
```
Reload Suricata dan periksa log.

**6. Detection Test dari srv2/laptop (20 menit)**
```bash
ping -c 5 192.168.1XX.10
nmap -sT -p 22,53,80,443 192.168.1XX.10
```
Gunakan hanya host lab milik kelompok sendiri. Periksa:
```bash
sudo tail -50 /var/log/suricata/fast.log
sudo journalctl -k | grep nft-drop
```

**7. Fail2ban untuk SSH (15 menit)**
```bash
sudo apt install -y fail2ban
sudo systemctl enable --now fail2ban
sudo fail2ban-client status
```
Konfigurasikan jail SSH sederhana dan uji dengan cara yang terkendali pada akun lab.

### UJI KONFIGURASI
- nftables dapat divalidasi dan policy input adalah drop.
- WireGuard peer dapat digunakan untuk akses terautentikasi.
- Suricata menghasilkan alert dari trafik uji.
- Mahasiswa dapat membedakan blocking oleh firewall dan detection oleh IDS.

### CHECKLIST TUGAS MINGGU 13
- [ ] nftables baseline aktif
- [ ] Protected workload srv2 tersedia
- [ ] WireGuard remote peer diuji
- [ ] Suricata aktif dan rule test menghasilkan alert
- [ ] Fail2ban aktif
- [ ] Log firewall dan IDS dianalisis

**Output:** Security enforcement dan monitoring dipraktikkan dengan topologi dua node yang konsisten.
