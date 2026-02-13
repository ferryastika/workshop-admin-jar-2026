
# MINGGU 12: SOFTWARE-DEFINED NETWORKING (SDN)
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam)
**SDN Architecture:**
```
Application Layer (Network Orchestration)
        ↓ Northbound API
Control Plane (SDN Controller)
        ↓ Southbound API (OpenFlow)
Data Plane (Virtual Network/Tunnels)
```

**WireGuard sebagai SDN Data Plane:**
- Encrypted tunnels (software-defined overlay)
- Peer-to-peer mesh network
- Dynamic routing antar subnet
- Centralized config management (SDN-like)

**SD-WAN Concepts:**
```
Site A ← WireGuard Tunnel → Controller ← Tunnel → Site B
    (192.168.101.0/24)                    (192.168.102.0/24)
                    ↓ Policy routing
              Internet + MPLS (Path selection)
```

### PERTANYAAN TEORI
1. WireGuard vs VXLAN untuk overlay network?
2. SDN controller centralized vs WireGuard P2P?
3. Policy-based routing di SD-WAN use case?
4. WireGuard handshake vs IPsec overhead?

### KEBUTUHAN PRAKTIKUM
**Topologi SDN WireGuard:**
```
Controller/Hub: srv1 (192.168.1XX.10, wg0: 10.10.0.1/24)
                    ↓ WireGuard tunnels
Spoke 1: srv2 (192.168.1XX.11, wg0: 10.10.0.2/24)
Spoke 2: srv3 (192.168.1XX.12, wg0: 10.10.0.3/24)
                    ↓ Policy routing
            Internet traffic via Hub (NAT)
```

**Hosts:**
```
Hub (Controller): kXX-srv1 192.168.1XX.10
Spoke 1: kXX-srv2 192.168.1XX.11
Spoke 2: kXX-srv3 192.168.1XX.12
```

**Aplikasi:**
```
wireguard, wireguard-tools, qrencode, python3
```

### LANGKAH PRAKTIKUM (2 jam)

**1. WireGuard Installation All Nodes (10 menit)**
```bash
# srv1, srv2, srv3
sudo apt update
sudo apt install wireguard wireguard-tools qrencode
sudo modprobe wireguard
lsmod | grep wireguard
```

**2. Key Generation Hub & Spokes (15 menit)**
```bash
# srv1 Hub
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod 600 /etc/wireguard/private.key
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
HUB_PUBKEY=$(sudo cat /etc/wireguard/public.key)

# srv2 Spoke1 (ulangi untuk srv3)
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod 600 /etc/wireguard/private.key
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
SPOKE1_PUBKEY=$(sudo cat /etc/wireguard/public.key)
```

**3. Hub Configuration srv1 (20 menit)**
```bash
sudo nano /etc/wireguard/wg0.conf
```
```
[Interface]
Address = 10.10.0.1/24
ListenPort = 51820
PrivateKey = <HUB_PRIVATE_KEY>
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o enp1s0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o enp1s0 -j MASQUERADE

# Spoke 1 (srv2)
[Peer]
PublicKey = <SPOKE1_PUBLIC_KEY>
AllowedIPs = 10.10.0.2/32, 192.168.1XX.11/32

# Spoke 2 (srv3)
[Peer]
PublicKey = <SPOKE2_PUBLIC_KEY>
AllowedIPs = 10.10.0.3/32, 192.168.1XX.12/32
```
```bash
# Enable IP forwarding
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Start WireGuard
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0
sudo wg show
```

**4. Spoke Configuration srv2 & srv3 (20 menit)**
```bash
# srv2
sudo nano /etc/wireguard/wg0.conf
```
```
[Interface]
Address = 10.10.0.2/24
PrivateKey = <SPOKE1_PRIVATE_KEY>

[Peer]
PublicKey = <HUB_PUBLIC_KEY>
Endpoint = 192.168.1XX.10:51820
AllowedIPs = 10.10.0.0/24, 192.168.1XX.0/24
PersistentKeepalive = 25
```
```bash
# srv3
sudo nano /etc/wireguard/wg0.conf
```
```
[Interface]
Address = 10.10.0.3/24
PrivateKey = <SPOKE2_PRIVATE_KEY>

[Peer]
PublicKey = <HUB_PUBLIC_KEY>
Endpoint = 192.168.1XX.10:51820
AllowedIPs = 10.10.0.0/24, 192.168.1XX.0/24
PersistentKeepalive = 25
```
```bash
# Start spokes
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0
```

**5. Mesh Connectivity Test (15 menit)**
```bash
# srv2 → Hub
ping -c 4 10.10.0.1

# srv2 → srv3 (via Hub routing)
ping -c 4 10.10.0.3

# Trace route
traceroute 10.10.0.3

# Bandwidth test encrypted tunnel
iperf3 -s  # srv3
iperf3 -c 10.10.0.3  # srv2
```

**6. Policy-Based Routing (SD-WAN simulation - 20 menit)**
```bash
# srv2: Route internet via Hub (0.0.0.0/0)
sudo wg-quick down wg0
sudo nano /etc/wireguard/wg0.conf
```
```
[Peer]
AllowedIPs = 0.0.0.0/0  # All traffic via Hub
```
```bash
sudo wg-quick up wg0

# Test internet via Hub
curl ifconfig.me  # Should show Hub IP (NAT)
traceroute 8.8.8.8  # Path via 10.10.0.1
```

**7. SDN Controller Simulation (Python - 20 menit)**
```bash
# srv1: Simple WireGuard orchestrator
nano wg_controller.py
```
```python
import subprocess
import json

class WireGuardSDN:
    def add_peer(self, public_key, allowed_ips):
        cmd = f"wg set wg0 peer {public_key} allowed-ips {allowed_ips}"
        subprocess.run(cmd.split())
        print(f"Peer added: {allowed_ips}")

    def remove_peer(self, public_key):
        cmd = f"wg set wg0 peer {public_key} remove"
        subprocess.run(cmd.split())
        print(f"Peer removed: {public_key}")

    def show_status(self):
        result = subprocess.run(['wg', 'show'], capture_output=True, text=True)
        print(result.stdout)

# Usage
controller = WireGuardSDN()
controller.show_status()
```
```bash
sudo python3 wg_controller.py
```

### UJI KONFIGURASI
```
**srv1 wg show:**
interface: wg0
  public key: <HUB_KEY>
  listening port: 51820
peer: <SPOKE1_KEY>
  endpoint: 192.168.1XX.11:random
  allowed ips: 10.10.0.2/32
  latest handshake: 10 seconds ago
  transfer: 5 MiB received, 3 MiB sent

**srv2 ping 10.10.0.3:** 64 bytes from 10.10.0.3: icmp_seq=1 ttl=64 time=2.1 ms
**iperf3:** 850 Mbits/sec (encrypted tunnel overhead ~15%)
```

**Screenshot Wajib (12 gambar):**
1. All nodes `wg show`
2. Hub wg0.conf
3. Spoke1 wg0.conf
4. Handshake success
5. Ping srv2 → srv3
6. Traceroute via Hub
7. iperf3 encrypted tunnel
8. Policy routing `AllowedIPs 0.0.0.0/0`
9. Internet via Hub (curl ifconfig.me)
10. wg_controller.py output
11. `ip addr show wg0`
12. `iptables -t nat -L`

### PERTANYAAN SEKITAR PRAKTIKUM
1. WireGuard handshake gagal, cek firewall port 51820 UDP?
2. AllowedIPs 0.0.0.0/0 vs 10.10.0.0/24 routing difference?
3. PersistentKeepalive 25 fungsi NAT traversal?
4. SD-WAN traffic steering policy implementation?

### CHECKLIST TUGAS MINGGU 12
- [ ] WireGuard Hub-Spoke topology
- [ ] 3-node mesh connectivity
- [ ] Policy-based routing (0.0.0.0/0)
- [ ] SDN controller simulation (Python)
- [ ] 12 screenshot lengkap

**Waktu Total:** 2 jam
**Output:** SD-WAN environment dengan WireGuard overlay network
