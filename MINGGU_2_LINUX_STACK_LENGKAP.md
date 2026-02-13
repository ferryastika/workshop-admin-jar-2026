
# MINGGU 2: FUNDAMENTAL LINUX NETWORK ADMINISTRATION
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam)
**Linux Networking Stack:**
```
User Space ← netlink ← Kernel Space
                    ↓
TCP/IP Stack ← Routing Table ← FIB
                    ↓
Network Devices (enp1s0, lo)
```

**Configuration Tools:**
- **Netplan** (Ubuntu/Server): YAML declarative
- **NetworkManager** (Desktop): GUI/CLI
- **systemd-networkd** (RHEL): INI format

**Key Concepts:**
- Interface states (UP/DOWN)
- ARP resolution
- Routing metrics
- Network namespaces

### PERTANYAAN TEORI
1. Apa fungsi FIB vs Routing Table?
2. Kapan pakai bonding vs bridging?
3. Bedakan `ip route` vs `ip neigh`?
4. Apa manfaat network namespace?

### KEBUTUHAN PRAKTIKUM
**Topologi (sama Minggu 1):**
```
Laptop ← SSH → kXX-srv1 (192.168.1XX.10)
             ↓ enp1s0
        [Mikrotik RB1100] 192.168.1XX.1
             ↓ VLAN 1XX
        [Backbone 10.252.108.0/24]
```

**Hosts:**
```
Target: kXX-srv1 (192.168.1XX.10)
Helper: kXX-srv2 (192.168.1XX.11) - ping target
```

**Aplikasi:**
```
Ubuntu 24.04 (sudah dari Minggu 1)
Tools: iproute2, bridge-utils, vlan, nmap, iperf3
```

### LANGKAH PRAKTIKUM (2 jam)

**1. Multiple IP Configuration (20 menit)**
```bash
# Alias IP di interface sama
sudo ip addr add 192.168.1XX.20/24 dev enp1s0 label enp1s0:1
sudo ip addr add 192.168.1XX.30/24 dev enp1s0 label enp1s0:2

# Verifikasi
ip addr show enp1s0
ping -I 192.168.1XX.20 192.168.1XX.11  # Test dari alias IP
```

**2. Static Routing (15 menit)**
```bash
# Route khusus ke subnet simulasi
sudo ip route add 172.16.1.0/24 via 192.168.1XX.11 dev enp1s0

# Policy routing table baru
sudo ip rule add from 192.168.1XX.20 table 100
sudo ip route add default via 192.168.1XX.11 table 100

ip rule show
ip route show table 100
```

**3. Network Bridging (25 menit)**
```bash
# Install bridge-utils
sudo apt install bridge-utils

# Buat bridge br0
sudo ip link add name br0 type bridge
sudo ip link set enp1s0 master br0
sudo ip addr add 192.168.1XX.10/24 dev br0
sudo ip link set br0 up
sudo ip link set enp1s0 up

# Test bridge
bridge link show
```

**4. VLAN Subinterface (20 menit)**
```bash
# Install vlan
sudo apt install vlan

# VLAN 100 di enp1s0
sudo modprobe 8021q
sudo vconfig add enp1s0 100
sudo ip addr add 192.168.100.10/24 dev enp1s0.100
sudo ip link set enp1s0.100 up

ip link show | grep vlan
```

**5. Bonding Interface (20 menit)**
```bash
# Simulasi bonding (balance-rr)
sudo ip link add bond0 type bond mode balance-rr miimon 100
sudo ip link set enp1s0 master bond0
sudo ip addr add 192.168.1XX.40/24 dev bond0
sudo ip link set bond0 up

cat /proc/net/bonding/bond0
```

**6. Diagnostic Tools Suite (20 menit)**
```bash
# Network discovery
nmap -sn 192.168.1XX.0/24

# Bandwidth test
iperf3 -s &                 # srv1 server
iperf3 -c 192.168.1XX.10    # srv2 client

# Path analysis
mtr 8.8.8.8

# Packet capture
sudo tcpdump -i enp1s0 -c 10 host 192.168.1XX.11 -w capture.pcap
tcpdump -r capture.pcap
```

### UJI KONFIGURASI
```
**Expected Results:**
1. `ip addr`: 4 IP berbeda di enp1s0 + br0 + bond0 + vlan
2. `ip route`: 3 routing table (main + table 100)
3. `bridge link`: br0 dengan member enp1s0
4. `nmap`: detect srv2 (.11)
5. `iperf3`: >900Mbps throughput
6. `mtr`: <50ms ke google.com
```

**Screenshot Wajib (6 gambar):**
1. `ip addr show` (multiple IP)
2. `ip route show table all`
3. `bridge link show`
4. `ip link` (VLAN + bonding)
5. `iperf3` throughput graph
6. `nmap` scan results

### PERTANYAAN SEKITAR PRAKTIKUM
1. Jika `ping -I 192.168.1XX.20` gagal, kemungkinan ARP issue?
2. Apa risiko bonding mode `balance-rr` di production?
3. Kenapa VLAN subinterface butuh `8021q` module?
4. Interpretasikan `mtr` output - dimana bottleneck?

### CHECKLIST TUGAS MINGGU 2
- [ ] Multiple IP (3 IP/interface)
- [ ] Static route + policy routing
- [ ] Bridge br0 functional
- [ ] VLAN 100 working
- [ ] Bonding bond0 UP
- [ ] 6 diagnostic tools tested
- [ ] 6 screenshot lab report

**Waktu Total:** 2 jam
**Output:** Linux server multi-interface ready untuk layanan network
