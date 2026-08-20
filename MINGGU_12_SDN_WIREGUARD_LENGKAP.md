# MINGGU 12: SOFTWARE-DEFINED NETWORKING (SDN) DENGAN WIREGUARD
## WORKSHOP ADMIN JARINGAN - PENS TI 2026

### DASAR TEORI (1 jam)
WireGuard digunakan sebagai contoh software-defined overlay. Baseline lab dibuat minimal:
```
srv1 (.10 / wg0 10.10.0.1) ← encrypted tunnel → srv2 (.11 / wg0 10.10.0.2)
```
Laptop dapat ditambahkan sebagai peer ketiga pada Minggu 13.

### PERTANYAAN TEORI
1. Apa beda underlay dan overlay network?
2. Apa fungsi `AllowedIPs` pada WireGuard?
3. Apa fungsi `PersistentKeepalive`?
4. Apa tradeoff hub-spoke, point-to-point, dan full mesh?

### KEBUTUHAN PRAKTIKUM
**Hosts:**
```
Gateway/Hub: kXX-srv1 192.168.1XX.10, wg0 10.10.0.1/24
Peer:        kXX-srv2 192.168.1XX.11, wg0 10.10.0.2/24
```

### LANGKAH PRAKTIKUM (2 jam)

**1. Install WireGuard pada kedua node (10 menit)**
```bash
sudo apt update
sudo apt install -y wireguard wireguard-tools
```

**2. Generate Key Pair pada srv1 dan srv2 (15 menit)**
```bash
umask 077
wg genkey | tee private.key | wg pubkey > public.key
cat public.key
```
Catat public key masing-masing host. Jangan memasukkan private key ke repository Git.

**3. Konfigurasi srv1 (20 menit)**
`/etc/wireguard/wg0.conf`:
```ini
[Interface]
Address = 10.10.0.1/24
ListenPort = 51820
PrivateKey = <SRV1_PRIVATE_KEY>

[Peer]
PublicKey = <SRV2_PUBLIC_KEY>
AllowedIPs = 10.10.0.2/32
```

**4. Konfigurasi srv2 (20 menit)**
```ini
[Interface]
Address = 10.10.0.2/24
PrivateKey = <SRV2_PRIVATE_KEY>

[Peer]
PublicKey = <SRV1_PUBLIC_KEY>
Endpoint = 192.168.1XX.10:51820
AllowedIPs = 10.10.0.1/32
PersistentKeepalive = 25
```

**5. Bring Up dan Verify (20 menit)**
Pada kedua node:
```bash
sudo systemctl enable --now wg-quick@wg0
sudo wg show
ip addr show wg0
```
Dari srv2:
```bash
ping -c4 10.10.0.1
```
Dari srv1:
```bash
ping -c4 10.10.0.2
```

**6. Routing Exercise (25 menit)**
Aktifkan IP forwarding pada srv1:
```bash
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-lab-forward.conf
sudo sysctl --system
```
Tambahkan route/AllowedIPs secara bertahap untuk mendemonstrasikan bagaimana policy overlay mempengaruhi trafik. Jangan langsung mengalihkan `0.0.0.0/0`; mahasiswa terlebih dahulu memverifikasi route spesifik agar tidak kehilangan akses SSH.

**7. Throughput & Failure Test (20 menit)**
```bash
# srv1
iperf3 -s

# srv2
iperf3 -c 10.10.0.1
```
Kemudian hentikan tunnel, amati routing, dan hidupkan kembali:
```bash
sudo systemctl stop wg-quick@wg0
ip route
sudo systemctl start wg-quick@wg0
```

### UJI KONFIGURASI
- Handshake aktif pada kedua node.
- `10.10.0.1` dan `10.10.0.2` saling reachable.
- Mahasiswa dapat menjelaskan route yang dibuat dari `AllowedIPs`.
- Failure/recovery tunnel dapat diamati.

### CHECKLIST TUGAS MINGGU 12
- [ ] WireGuard installed pada srv1 dan srv2
- [ ] Key pair dibuat dengan permission aman
- [ ] Tunnel 2-node aktif
- [ ] Routing exercise selesai
- [ ] Throughput dan failure test selesai

**Output:** Overlay WireGuard minimal yang cukup untuk memahami tunnel, routing policy, dan encrypted transport.
