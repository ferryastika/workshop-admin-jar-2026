
---

## 📊 Network Topology

# TOPOLOGI JARINGAN LAB - DIAGRAM MERMAID
## WORKSHOP ADMINISTRASI JARINGAN PENS 2026

## 1. TOPOLOGI INFRASTRUKTUR LENGKAP LAB

```mermaid
graph TB
    subgraph Internet["🌐 INTERNET"]
        WAN[Public Internet<br/>ISP Gateway]
    end

    subgraph Backbone["🏢 BACKBONE NETWORK<br/>10.252.108.0/24"]
        GW[🔴 Gateway Router<br/>10.252.108.1<br/>NAT + Firewall]
        CORE_SW[⚡ Core Switch<br/>Mikrotik 24-port<br/>10.252.108.2<br/>VLAN 101-110]

        subgraph Management["Management Zone"]
            PROX[🖥️ Proxmox VE<br/>10.252.108.10<br/>Xeon 40C / 64GB RAM<br/>Hypervisor]
            MON[📊 Monitoring Server<br/>10.252.108.100<br/>Prometheus + Grafana]
            DNS_MAIN[🌐 DNS Lab Main<br/>10.252.108.53<br/>lab.pens.ac.id]
            NTP_MAIN[⏰ NTP Stratum 1<br/>10.252.108.123]
        end
    end

    WAN -->|NAT Translation| GW
    GW <-->|Trunk VLAN All| CORE_SW
    CORE_SW --> PROX
    CORE_SW --> MON
    CORE_SW --> DNS_MAIN
    CORE_SW --> NTP_MAIN

    subgraph K01["👥 KELOMPOK 01<br/>VLAN 101 - 192.168.101.0/24"]
        RB01[🔶 Mikrotik RB1100<br/>WAN: 10.252.108.11<br/>LAN: 192.168.101.1/24<br/>Gateway + DHCP Server]

        subgraph VMs01["Virtual Machines (Proxmox)"]
            VM01_1[💻 k01-srv1<br/>192.168.101.10<br/>Ubuntu 24.04 LTS<br/>4C/8GB - Primary Services]
            VM01_2[💻 k01-srv2<br/>192.168.101.11<br/>Ubuntu 24.04 LTS<br/>2C/4GB - Backend App]
            VM01_3[💻 k01-srv3<br/>192.168.101.12<br/>Rocky Linux 9<br/>2C/4GB - Database]
        end

        LAPTOP01[💼 Laptop Mahasiswa<br/>192.168.101.100-150<br/>DHCP Dynamic]
    end

    subgraph K02["👥 KELOMPOK 02<br/>VLAN 102 - 192.168.102.0/24"]
        RB02[🔶 Mikrotik RB1100<br/>WAN: 10.252.108.12<br/>LAN: 192.168.102.1/24]

        VM02_1[💻 k02-srv1<br/>192.168.102.10]
        VM02_2[💻 k02-srv2<br/>192.168.102.11]
        VM02_3[💻 k02-srv3<br/>192.168.102.12]
        LAPTOP02[💼 Laptop k02<br/>DHCP]
    end

    subgraph K03["👥 KELOMPOK 03-09<br/>VLAN 103-109"]
        K03_09[🔶 RB1100 x7<br/>Similar topology<br/>192.168.103-109.0/24]
    end

    subgraph K10["👥 KELOMPOK 10<br/>VLAN 110 - 192.168.110.0/24"]
        RB10[🔶 Mikrotik RB1100<br/>WAN: 10.252.108.20<br/>LAN: 192.168.110.1/24]

        VM10_1[💻 k10-srv1<br/>192.168.110.10]
        VM10_2[💻 k10-srv2<br/>192.168.110.11]
        VM10_3[💻 k10-srv3<br/>192.168.110.12]
        LAPTOP10[💼 Laptop k10<br/>DHCP]
    end

    CORE_SW -.VLAN 101.-> RB01
    CORE_SW -.VLAN 102.-> RB02
    CORE_SW -.VLAN 103-109.-> K03_09
    CORE_SW -.VLAN 110.-> RB10

    PROX ==>|VM Hosting| VM01_1
    PROX ==>|VM Hosting| VM01_2
    PROX ==>|VM Hosting| VM01_3
    PROX ==>|VM Hosting| VM02_1
    PROX ==>|VM Hosting| VM10_1

    RB01 --> VM01_1
    RB01 --> VM01_2
    RB01 --> VM01_3
    RB01 --> LAPTOP01

    RB02 --> VM02_1
    RB02 --> VM02_2
    RB02 --> VM02_3
    RB02 --> LAPTOP02

    RB10 --> VM10_1
    RB10 --> VM10_2
    RB10 --> VM10_3
    RB10 --> LAPTOP10

    style WAN fill:#e74c3c,color:#fff
    style GW fill:#c0392b,color:#fff
    style CORE_SW fill:#16a085,color:#fff
    style PROX fill:#2980b9,color:#fff
    style MON fill:#27ae60,color:#fff
    style DNS_MAIN fill:#8e44ad,color:#fff
    style NTP_MAIN fill:#d35400,color:#fff
    style RB01 fill:#f39c12,color:#000
    style RB02 fill:#f39c12,color:#000
    style RB10 fill:#f39c12,color:#000
    style VM01_1 fill:#3498db,color:#fff
    style VM01_2 fill:#3498db,color:#fff
    style VM01_3 fill:#e67e22,color:#fff
```

---

## 2. TOPOLOGI DETAIL KELOMPOK 01 (Representatif Semua Kelompok)

```mermaid
graph TB
    subgraph External["🌐 EXTERNAL ACCESS"]
        INET[Internet<br/>via 10.252.108.1]
        BACKBONE[Backbone Switch<br/>10.252.108.0/24]
    end

    subgraph EdgeRouter["🔶 EDGE ROUTER - Mikrotik RB1100"]
        WAN_IF[ether1 WAN<br/>10.252.108.11/24<br/>VLAN 101]
        LAN_IF[ether2 LAN<br/>192.168.101.1/24<br/>Gateway]
        NAT_RULE[NAT Masquerade<br/>src: 192.168.101.0/24]
        FW_FILTER[Firewall Filter<br/>Input: Drop<br/>Forward: Accept]
        DHCP_SRV[DHCP Server<br/>Pool: .100-.200<br/>Lease: 1 hour]
    end

    INET --> BACKBONE
    BACKBONE -.VLAN 101 Tagged.-> WAN_IF
    WAN_IF --> NAT_RULE
    NAT_RULE --> LAN_IF
    LAN_IF --> FW_FILTER
    LAN_IF --> DHCP_SRV

    subgraph InternalNet["🏠 INTERNAL NETWORK 192.168.101.0/24"]

        subgraph SRV1["💻 k01-srv1 - 192.168.101.10<br/>Ubuntu 24.04 (4C/8GB)"]
            direction TB
            DNS_SRV[🌐 BIND9 DNS<br/>Port 53<br/>Zone: kelompok01.lab]
            WEB_SRV[🌍 Nginx Web<br/>Port 80/443<br/>Reverse Proxy + SSL]
            PROM[📊 Prometheus<br/>Port 9090<br/>Scrape Interval: 15s]
            GRAF[📈 Grafana<br/>Port 3000<br/>Dashboard]
            NODE_EXP1[📡 node_exporter<br/>Port 9100]
            DOCKER1[🐳 Docker Engine<br/>20+ containers]
            K8S_MASTER[☸️ K3s Master<br/>Control Plane]
            WG_HUB[🔐 WireGuard Hub<br/>wg0: 10.10.0.1<br/>Port 51820]
            NFT1[🛡️ nftables<br/>Policy: Drop]
        end

        subgraph SRV2["💻 k01-srv2 - 192.168.101.11<br/>Ubuntu 24.04 (2C/4GB)"]
            direction TB
            APP_BE1[⚙️ Node.js App<br/>Port 8080<br/>Backend API]
            NODE_EXP2[📡 node_exporter<br/>Port 9100]
            DOCKER2[🐳 Docker Engine]
            K8S_WORK1[☸️ K3s Worker 1<br/>Kubelet]
            NFS_CLIENT[📁 NFS Client<br/>Mount: /mnt/nfs_share]
            WG_SPOKE1[🔐 WireGuard Spoke<br/>wg0: 10.10.0.2]
        end

        subgraph SRV3["💻 k01-srv3 - 192.168.101.12<br/>Rocky Linux 9 (2C/4GB)"]
            direction TB
            DB_MYSQL[🗄️ MySQL 8.0<br/>Port 3306<br/>Database]
            NODE_EXP3[📡 node_exporter<br/>Port 9100]
            K8S_WORK2[☸️ K3s Worker 2<br/>Kubelet]
            SMB_CLIENT[📁 Samba Client<br/>Mount: /mnt/smb_secure]
            SURI[🚨 Suricata IDS<br/>Monitor enp1s0]
        end

        LAPTOP_DEV[💼 Laptop Mahasiswa<br/>192.168.101.100-150<br/>DHCP Client<br/>SSH + Browser Access]
    end

    LAN_IF --> SRV1
    LAN_IF --> SRV2
    LAN_IF --> SRV3
    LAN_IF --> LAPTOP_DEV

    LAPTOP_DEV -->|SSH :22| SRV1
    LAPTOP_DEV -->|HTTP :3000| GRAF
    LAPTOP_DEV -->|HTTPS :443| WEB_SRV
    LAPTOP_DEV -->|DNS Query :53| DNS_SRV

    WEB_SRV -->|Proxy Pass| APP_BE1
    PROM -->|Scrape| NODE_EXP1
    PROM -->|Scrape| NODE_EXP2
    PROM -->|Scrape| NODE_EXP3
    GRAF -->|PromQL| PROM

    K8S_MASTER -->|kubectl| K8S_WORK1
    K8S_MASTER -->|kubectl| K8S_WORK2

    WG_HUB <-->|Encrypted Tunnel| WG_SPOKE1

    APP_BE1 -->|SQL Query| DB_MYSQL

    NODE_EXP2 -->|NFS Mount| DNS_SRV
    NODE_EXP3 -->|SMB Mount| DNS_SRV

    style INET fill:#e74c3c,color:#fff
    style WAN_IF fill:#e67e22,color:#fff
    style LAN_IF fill:#f39c12,color:#000
    style DNS_SRV fill:#9b59b6,color:#fff
    style WEB_SRV fill:#3498db,color:#fff
    style PROM fill:#e74c3c,color:#fff
    style GRAF fill:#f39c12,color:#000
    style APP_BE1 fill:#1abc9c,color:#fff
    style DB_MYSQL fill:#34495e,color:#fff
    style K8S_MASTER fill:#2c3e50,color:#fff
    style WG_HUB fill:#c0392b,color:#fff
```

---

## 3. TOPOLOGI SERVICES ARCHITECTURE (Layer View)

```mermaid
graph LR
    subgraph Layer7["📱 LAYER 7 - APPLICATION"]
        HTTPS[HTTPS/SSL<br/>Port 443]
        DNS[DNS<br/>Port 53]
        HTTP[HTTP<br/>Port 80]
        SSH[SSH<br/>Port 22]
    end

    subgraph Layer4["🚦 LAYER 4 - TRANSPORT"]
        TCP[TCP<br/>Connection-oriented]
        UDP[UDP<br/>Connectionless]
    end

    subgraph Layer3["🌐 LAYER 3 - NETWORK"]
        ROUTING[IP Routing<br/>192.168.101.0/24<br/>Default: .1]
        NAT_L3[NAT Translation<br/>Private ↔ Public]
        IPSEC[IPsec Tunnel<br/>VPN Encryption]
    end

    subgraph Layer2["🔗 LAYER 2 - DATA LINK"]
        VLAN[VLAN Tagging<br/>802.1Q<br/>VLAN 101-110]
        SWITCH[Ethernet Switching<br/>MAC Learning]
        BRIDGE[Linux Bridge<br/>br0]
    end

    subgraph Layer1["⚡ LAYER 1 - PHYSICAL"]
        FIBER[Fiber Optic<br/>Backbone 1Gbps]
        COPPER[Cat6 UTP<br/>1Gbps]
    end

    HTTPS --> TCP
    DNS --> UDP
    HTTP --> TCP
    SSH --> TCP

    TCP --> ROUTING
    UDP --> ROUTING

    ROUTING --> NAT_L3
    NAT_L3 --> VLAN
    IPSEC --> ROUTING

    VLAN --> SWITCH
    SWITCH --> BRIDGE
    BRIDGE --> FIBER
    BRIDGE --> COPPER

    style Layer7 fill:#3498db,color:#fff
    style Layer4 fill:#2ecc71,color:#fff
    style Layer3 fill:#f39c12,color:#000
    style Layer2 fill:#9b59b6,color:#fff
    style Layer1 fill:#e74c3c,color:#fff
```

---

## 4. TOPOLOGI WIREGUARD VPN OVERLAY (SDN)

```mermaid
graph TB
    subgraph Underlay["🌐 UNDERLAY NETWORK (Physical)"]
        BB[Backbone<br/>10.252.108.0/24]
        NET101[Kelompok 01<br/>192.168.101.0/24]
        NET102[Kelompok 02<br/>192.168.102.0/24]
        NET110[Kelompok 10<br/>192.168.110.0/24]
    end

    subgraph Overlay["🔐 OVERLAY NETWORK (WireGuard VPN)<br/>10.10.0.0/24"]
        HUB[🔴 HUB<br/>k01-srv1<br/>Physical: 192.168.101.10<br/>wg0: 10.10.0.1/24<br/>ListenPort: 51820]

        SPOKE1[🔵 SPOKE 1<br/>k01-srv2<br/>Physical: 192.168.101.11<br/>wg0: 10.10.0.2/24]

        SPOKE2[🔵 SPOKE 2<br/>k01-srv3<br/>Physical: 192.168.101.12<br/>wg0: 10.10.0.3/24]

        SPOKE_K02[🔵 SPOKE K02<br/>k02-srv1<br/>Physical: 192.168.102.10<br/>wg0: 10.10.0.10/24]

        SPOKE_K10[🔵 SPOKE K10<br/>k10-srv1<br/>Physical: 192.168.110.10<br/>wg0: 10.10.0.20/24]

        LAPTOP_VPN[💼 Remote Laptop<br/>Public IP: 114.79.x.x<br/>wg0: 10.10.0.100/24]
    end

    BB --> NET101
    BB --> NET102
    BB --> NET110

    NET101 -.Physical Routing.-> HUB
    NET101 -.Physical Routing.-> SPOKE1
    NET101 -.Physical Routing.-> SPOKE2
    NET102 -.Physical Routing.-> SPOKE_K02
    NET110 -.Physical Routing.-> SPOKE_K10

    HUB <===>|Encrypted Tunnel<br/>UDP :51820<br/>ChaCha20-Poly1305| SPOKE1
    HUB <===>|Encrypted Tunnel| SPOKE2
    HUB <===>|Encrypted Tunnel<br/>Cross-VLAN| SPOKE_K02
    HUB <===>|Encrypted Tunnel<br/>Cross-VLAN| SPOKE_K10
    HUB <===>|Internet VPN<br/>Endpoint: 10.252.108.11:51820| LAPTOP_VPN

    SPOKE1 -.Mesh via Hub.-> SPOKE2
    SPOKE1 -.Mesh via Hub.-> SPOKE_K02
    SPOKE_K02 -.Mesh via Hub.-> SPOKE_K10

    INTERNET[🌍 Internet<br/>0.0.0.0/0]
    HUB -->|NAT Policy Routing<br/>AllowedIPs: 0.0.0.0/0| INTERNET

    style HUB fill:#c0392b,color:#fff
    style SPOKE1 fill:#3498db,color:#fff
    style SPOKE2 fill:#3498db,color:#fff
    style SPOKE_K02 fill:#3498db,color:#fff
    style SPOKE_K10 fill:#3498db,color:#fff
    style LAPTOP_VPN fill:#95a5a6,color:#fff
    style INTERNET fill:#e74c3c,color:#fff
```

---

## 5. TOPOLOGI KUBERNETES CLUSTER (K3s)

```mermaid
graph TB
    subgraph ControlPlane["☸️ CONTROL PLANE - k01-srv1"]
        API_SRV[kube-apiserver<br/>Port 6443]
        ETCD[etcd<br/>Key-Value Store]
        SCHED[kube-scheduler<br/>Pod Placement]
        CTRL_MGR[kube-controller-manager<br/>Reconciliation]
        CLOUD_CTRL[cloud-controller-manager]
    end

    subgraph WorkerNode1["🔧 WORKER NODE 1 - k01-srv2"]
        KUBELET1[kubelet<br/>Pod Lifecycle]
        KPROXY1[kube-proxy<br/>Network Rules]
        RUNTIME1[containerd<br/>Container Runtime]
        CALICO_FELIX1[calico-felix<br/>Network Policy Enforcement]

        subgraph Pods1["Pods"]
            POD1_1[nginx-web<br/>10.42.1.5]
            POD1_2[app-backend<br/>10.42.1.6]
        end
    end

    subgraph WorkerNode2["🔧 WORKER NODE 2 - k01-srv3"]
        KUBELET2[kubelet]
        KPROXY2[kube-proxy]
        RUNTIME2[containerd]
        CALICO_FELIX2[calico-felix]

        subgraph Pods2["Pods"]
            POD2_1[nginx-web<br/>10.42.2.8]
            POD2_2[mysql-db<br/>10.42.2.9]
        end
    end

    subgraph NetworkLayer["🌐 CALICO CNI - Overlay Network"]
        CALICO_CTRL[calico-controller<br/>IPAM + Route Sync]
        BGP_MESH[BGP Full Mesh<br/>Route Advertisement]
        IP_POOL[IP Pool<br/>10.42.0.0/16<br/>Pod CIDR]
    end

    subgraph Services["⚖️ KUBERNETES SERVICES"]
        SVC_NGINX[Service: nginx-svc<br/>ClusterIP: 10.43.0.10<br/>Type: ClusterIP]
        SVC_DB[Service: mysql-svc<br/>ClusterIP: 10.43.1.20<br/>Type: ClusterIP]
        INGRESS[Ingress Controller<br/>nginx.kelompok01.lab<br/>TLS Termination]
    end

    EXTERNAL_CLIENT[💼 External Client<br/>Laptop<br/>192.168.101.100]

    API_SRV --> ETCD
    API_SRV --> SCHED
    API_SRV --> CTRL_MGR

    API_SRV -->|Register| KUBELET1
    API_SRV -->|Register| KUBELET2

    KUBELET1 --> RUNTIME1
    RUNTIME1 --> POD1_1
    RUNTIME1 --> POD1_2

    KUBELET2 --> RUNTIME2
    RUNTIME2 --> POD2_1
    RUNTIME2 --> POD2_2

    CALICO_CTRL --> CALICO_FELIX1
    CALICO_CTRL --> CALICO_FELIX2
    CALICO_FELIX1 --> BGP_MESH
    CALICO_FELIX2 --> BGP_MESH
    BGP_MESH --> IP_POOL

    POD1_1 --> SVC_NGINX
    POD2_1 --> SVC_NGINX
    POD2_2 --> SVC_DB

    SVC_NGINX --> INGRESS

    EXTERNAL_CLIENT -->|HTTP/HTTPS| INGRESS
    INGRESS -->|Load Balance| SVC_NGINX

    POD1_2 -->|SQL Query| SVC_DB

    style API_SRV fill:#2c3e50,color:#fff
    style ETCD fill:#34495e,color:#fff
    style KUBELET1 fill:#3498db,color:#fff
    style KUBELET2 fill:#3498db,color:#fff
    style CALICO_CTRL fill:#16a085,color:#fff
    style SVC_NGINX fill:#27ae60,color:#fff
    style INGRESS fill:#f39c12,color:#000
```

---

## 6. TOPOLOGI MONITORING STACK (Prometheus + Grafana)

```mermaid
graph LR
    subgraph Targets["📡 MONITORING TARGETS"]
        TARGET1[k01-srv1<br/>192.168.101.10:9100<br/>node_exporter<br/>CPU/RAM/Disk/Net]
        TARGET2[k01-srv2<br/>192.168.101.11:9100<br/>node_exporter]
        TARGET3[k01-srv3<br/>192.168.101.12:9100<br/>node_exporter]
        TARGET_NGINX[k01-srv1<br/>192.168.101.10:9113<br/>nginx_exporter<br/>HTTP metrics]
        TARGET_MYSQL[k01-srv3<br/>192.168.101.12:9104<br/>mysqld_exporter<br/>DB queries]
    end

    subgraph PrometheusServer["📊 PROMETHEUS - k01-srv1:9090"]
        SCRAPER[Scraper<br/>Interval: 15s<br/>Timeout: 10s]
        TSDB[Time-Series DB<br/>Retention: 15 days<br/>WAL + Chunks]
        ALERTS[Alertmanager<br/>Port 9093<br/>Email/Slack Alerts]
        PROM_API[HTTP API<br/>/api/v1/query<br/>PromQL Endpoint]
    end

    subgraph Grafana["📈 GRAFANA - k01-srv1:3000"]
        DATASOURCE[Data Source<br/>Prometheus<br/>URL: http://localhost:9090]
        DASHBOARD1[Dashboard: Node Overview<br/>CPU/Memory/Disk Usage<br/>Panels: 12]
        DASHBOARD2[Dashboard: Network Traffic<br/>RX/TX Bytes<br/>Drop/Error Rate]
        DASHBOARD3[Dashboard: Services Health<br/>Nginx/MySQL Uptime<br/>Response Time]
        ALERTS_GRAF[Alert Rules<br/>Threshold: CPU > 80%<br/>Notification: Email]
    end

    CLIENT[💼 Browser<br/>http://192.168.101.10:3000<br/>User: admin]

    TARGET1 -->|HTTP GET /metrics| SCRAPER
    TARGET2 -->|HTTP GET /metrics| SCRAPER
    TARGET3 -->|HTTP GET /metrics| SCRAPER
    TARGET_NGINX -->|HTTP GET /metrics| SCRAPER
    TARGET_MYSQL -->|HTTP GET /metrics| SCRAPER

    SCRAPER --> TSDB
    TSDB --> PROM_API
    TSDB --> ALERTS

    PROM_API -->|PromQL Query<br/>rate --node_cpu-seconds-total 5m| DATASOURCE
    DATASOURCE --> DASHBOARD1
    DATASOURCE --> DASHBOARD2
    DATASOURCE --> DASHBOARD3
    DATASOURCE --> ALERTS_GRAF

    CLIENT -->|Login + View| DASHBOARD1
    CLIENT -->|Real-time Refresh 5s| DASHBOARD2

    ALERTS -->|SMTP| EMAIL[📧 Email<br/>admin@kelompok01.lab]
    ALERTS_GRAF -->|Webhook| SLACK[💬 Slack<br/>#alerts channel]

    style SCRAPER fill:#e74c3c,color:#fff
    style TSDB fill:#c0392b,color:#fff
    style DASHBOARD1 fill:#f39c12,color:#000
    style DASHBOARD2 fill:#f39c12,color:#000
    style DASHBOARD3 fill:#f39c12,color:#000
    style CLIENT fill:#95a5a6,color:#fff
```
---

## 7. TOPOLOGI DOCKER NETWORKING (Multi-Mode)

```mermaid
graph TB
    subgraph DockerHost["🐳 DOCKER HOST - k01-srv1"]
        DOCKER_DAEMON[Docker Daemon<br/>dockerd<br/>API: unix:///var/run/docker.sock]

        subgraph DefaultBridge["🌉 docker0 (Default Bridge)<br/>172.17.0.0/16"]
            C1[web1<br/>nginx:alpine<br/>172.17.0.2<br/>No Port Expose]
            C2[web2<br/>nginx:alpine<br/>172.17.0.3]
        end

        subgraph CustomBridge["🌉 my_bridge (Custom Bridge)<br/>172.20.0.0/16<br/>DNS Resolution Enabled"]
            C3[app1<br/>node:18<br/>172.20.0.2<br/>Name: app1.my_bridge]
            C4[app2<br/>node:18<br/>172.20.0.3<br/>Name: app2.my_bridge]
            C5[redis<br/>redis:7<br/>172.20.0.4]
        end

        subgraph HostNetwork["🖥️ Host Network<br/>Shared with Host"]
            C6[nginx-host<br/>nginx:alpine<br/>Binds: 192.168.101.10:80<br/>No Network Isolation]
        end

        subgraph Macvlan["📡 Macvlan Network<br/>Physical-like Addressing"]
            C7[mac-container<br/>alpine<br/>192.168.101.50/24<br/>Gateway: .1<br/>Direct L2 Access]
        end

        HOST_ETH[enp1s0<br/>192.168.101.10/24<br/>Host Primary Interface]
    end

    EXTERNAL_NET[🌐 External Network<br/>192.168.101.0/24]

    DOCKER_DAEMON --> DefaultBridge
    DOCKER_DAEMON --> CustomBridge
    DOCKER_DAEMON --> HostNetwork
    DOCKER_DAEMON --> Macvlan

    C1 <-->|Container-to-Container<br/>Bridge NAT| C2
    C3 <-->|DNS: ping app2| C4
    C4 <-->|Redis Client| C5

    C6 -.Shares Network Stack.-> HOST_ETH
    C7 -.MAC Address Clone.-> HOST_ETH

    DefaultBridge -->|NAT via iptables| HOST_ETH
    CustomBridge -->|NAT via iptables| HOST_ETH
    HOST_ETH --> EXTERNAL_NET

    C7 -->|Direct L2| EXTERNAL_NET

    CLIENT_EXT[💼 External Client<br/>192.168.101.100]
    CLIENT_EXT -->|HTTP :80| C6
    CLIENT_EXT -->|Ping 192.168.101.50| C7

    style DOCKER_DAEMON fill:#0db7ed,color:#fff
    style DefaultBridge fill:#3498db,color:#fff
    style CustomBridge fill:#16a085,color:#fff
    style HostNetwork fill:#f39c12,color:#000
    style Macvlan fill:#e74c3c,color:#fff
```

---

## 8. TOPOLOGI SECURITY LAYERS (Defense in Depth)

```mermaid
---
config:
  layout: elk
---
flowchart TB
 subgraph Internet["🌍 INTERNET - Untrusted"]
        ATTACKER["🔴 Attacker<br>Scanning/Brute Force<br>DDoS Attempts"]
        LEGIT_USER["✅ Legitimate User<br>VPN Client"]
  end
 subgraph Perimeter["🛡️ PERIMETER SECURITY"]
        GW_FW["Gateway Firewall<br>10.252.108.1<br>Stateful Inspection<br>Drop Invalid Packets"]
        RATE_LIMIT["Rate Limiting<br>Max 100 conn/s per IP"]
  end
 subgraph DMZ["🔶 DMZ ZONE"]
        WG_GATEWAY["WireGuard VPN<br>k01-srv1:51820<br>Public Key Auth<br>Only Authenticated Access"]
  end
 subgraph HostFirewall["🛡️ HOST-BASED FIREWALL (nftables)"]
        NFT_INPUT["Input Chain<br>Policy: DROP<br>Allow: SSH(22), HTTP(80), HTTPS(443)"]
        NFT_FORWARD["Forward Chain<br>Policy: ACCEPT<br>+ State Tracking"]
        NFT_OUTPUT["Output Chain<br>Policy: ACCEPT"]
  end
 subgraph IDS["🚨 INTRUSION DETECTION"]
        SURICATA["Suricata IDS<br>k01-srv3:enp1s0<br>Rules: ET Open<br>Alert to /var/log/suricata/"]
        F2B["fail2ban<br>Monitor: /var/log/auth.log<br>Ban After: 5 attempts<br>Ban Time: 1 hour"]
  end
 subgraph AppSecurity["🔐 APPLICATION SECURITY"]
        SSL_TERM["SSL/TLS Termination<br>Nginx<br>TLSv1.3 only<br>Strong Ciphers"]
        APP_FW["ModSecurity WAF<br>OWASP Core Rules<br>SQL Injection Protection"]
  end
 subgraph DataSecurity["💾 DATA SECURITY"]
        ENCRYPT_DISK["LUKS Encryption<br>/dev/vda2<br>AES-256"]
        ENCRYPT_TRANSIT["Data in Transit<br>TLS 1.3<br>Perfect Forward Secrecy"]
        BACKUP_ENC["Encrypted Backups<br>GPG Encrypted rsync<br>Offsite Storage"]
  end
 subgraph InternalZone["🏠 INTERNAL ZONE - 192.168.101.0/24"]
        HostFirewall
        IDS
        AppSecurity
        DataSecurity
        SERVER["🖥️ k01-srv1<br>Ubuntu 24.04<br>Hardened Kernel<br>AppArmor Enforcing"]
  end
    ATTACKER -- Port Scan --> GW_FW
    GW_FW -- Block Invalid --> ATTACKER
    LEGIT_USER -- WireGuard Handshake --> WG_GATEWAY
    WG_GATEWAY -- Encrypted Tunnel --> NFT_INPUT
    GW_FW --> RATE_LIMIT
    RATE_LIMIT -- Passed --> WG_GATEWAY
    NFT_INPUT -- Allowed Traffic --> SERVER
    SERVER -- All Traffic --> SURICATA
    SURICATA -- Alert on Threat --> F2B
    F2B -- Add IP Ban --> NFT_INPUT
    SERVER -- HTTPS Request --> SSL_TERM
    SSL_TERM -- Decrypt + Inspect --> APP_FW
    APP_FW -- Clean Request --> SERVER
    SERVER -- Write Data --> ENCRYPT_DISK
    SERVER -- Send Data --> ENCRYPT_TRANSIT
    SERVER -- Cron Backup --> BACKUP_ENC

    style ATTACKER fill:#e74c3c,color:#fff
    style GW_FW fill:#c0392b,color:#fff
    style WG_GATEWAY fill:#16a085,color:#fff
    style NFT_INPUT fill:#e67e22,color:#fff
    style SURICATA fill:#9b59b6,color:#fff
    style F2B fill:#34495e,color:#fff
    style SSL_TERM fill:#27ae60,color:#fff
    style ENCRYPT_DISK fill:#2c3e50,color:#fff
```

---

## 9. TOPOLOGI DATA FLOW (End-to-End Request)

```mermaid
sequenceDiagram
    participant User as 💼 Laptop Client<br/>192.168.101.100
    participant DNS as 🌐 BIND9 DNS<br/>k01-srv1:53
    participant Nginx as 🌍 Nginx Proxy<br/>k01-srv1:443
    participant Backend as ⚙️ Node.js API<br/>k01-srv2:8080
    participant DB as 🗄️ MySQL DB<br/>k01-srv3:3306
    participant Prom as 📊 Prometheus<br/>k01-srv1:9090

    Note over User,DB: 1. DNS Resolution Phase
    User->>DNS: DNS Query: api.kelompok01.lab
    DNS-->>User: A Record: 192.168.101.10

    Note over User,Nginx: 2. TLS Handshake
    User->>Nginx: ClientHello (TLS 1.3)
    Nginx-->>User: ServerHello + Certificate
    User->>Nginx: Finished (Encrypted)

    Note over User,Backend: 3. HTTP Request Flow
    User->>Nginx: GET /api/users HTTP/1.1<br/>Host: api.kelompok01.lab
    Nginx->>Nginx: Check nftables rules (ACCEPT)
    Nginx->>Backend: Proxy Pass to 192.168.101.11:8080

    Note over Backend,DB: 4. Database Query
    Backend->>DB: SELECT * FROM users LIMIT 10
    DB-->>Backend: Resultset (10 rows)

    Note over Backend,User: 5. Response Chain
    Backend-->>Nginx: JSON Response (200 OK)
    Nginx-->>User: HTTPS Response (Encrypted)

    Note over Prom: 6. Metrics Collection (Background)
    Prom->>Nginx: GET /metrics (nginx_exporter)
    Nginx-->>Prom: http_requests_total{code="200"} 1245
    Prom->>Backend: GET /metrics (node_exporter)
    Backend-->>Prom: node_cpu_seconds_total 8932.45
    Prom->>DB: GET /metrics (mysqld_exporter)
    DB-->>Prom: mysql_queries_total 45678

    Note over User: 7. Total Latency
    Note over User: DNS: 5ms | TLS: 15ms | App: 25ms<br/>Total: 45ms
```

---


## LEGENDA WARNA & ICON

| Warna Hex | Komponen | Emoji |
|-----------|----------|-------|
| `#e74c3c` (Red) | Internet, Critical, Firewall | 🔴 🌍 |
| `#c0392b` (Dark Red) | Gateway, Security Hub | 🛡️ |
| `#f39c12` (Orange) | Infrastructure, Router, Warning | 🔶 ⚡ |
| `#3498db` (Blue) | Servers, Workers, Pods | 💻 🔵 |
| `#2c3e50` (Dark Blue) | Control Plane, Master | ☸️ |
| `#16a085` (Teal) | Network Services, Switches | 🌐 |
| `#27ae60` (Green) | Monitoring, Success | 📊 ✅ |
| `#9b59b6` (Purple) | DNS, Database | 🌐 🗄️ |
| `#95a5a6` (Gray) | Client Devices, Laptop | 💼 |
| `#34495e` (Charcoal) | Security Tools | 🚨 |


---

## 📝 Penilaian (Total 100%)

| Komponen | Bobot | Keterangan |
|----------|-------|------------|
| Kehadiran & Partisipasi | 10% | Min 75% hadir |
| Tugas Praktikum | 30% | Lab report mingguan |
| Quiz & UTS | 25% | Teori + praktik |
| Project Akhir | 25% | Infrastruktur lengkap |
| UAS Praktik | 10% | Troubleshooting |

---

## 🚀 Quick Start

```bash
# Clone repository
git clone https://github.com/[username]/praktikum-admin-jaringan-2026.git
cd praktikum-admin-jaringan-2026

# Baca modul Minggu 1
cat MINGGU_1_PRAKTIKUM_LENGKAP.md

# Akses Proxmox (ganti XX dengan nomor kelompok)
https://10.252.108.10:8006
User: kelompokXX@pve

# SSH ke VM
ssh adminXX@192.168.1XX.10
