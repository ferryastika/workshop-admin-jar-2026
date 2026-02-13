
# MINGGU 9: CONTAINER NETWORKING FUNDAMENTALS
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam)
**Docker Networking Models:**
| Mode | Namespace | Use Case | Connectivity |
|------|-----------|----------|--------------|
| bridge | Isolated | Default multi-container | Container ↔ Container |
| host | Shared | Single container | Container ↔ Host |
| none | Isolated | Static IP | No network |
| overlay | VXLAN | Swarm multi-host | Cross-host containers |

**CNI Concepts (Kubernetes preview):**
- Container Network Interface standard
- Plugins: bridge, flannel, calico, weave

### PERTANYAAN TEORI
1. Bridge network vs host network security implikasi?
2. Apa fungsi Docker0 interface 172.17.0.1?
3. Overlay network butuh VXLAN kenapa?
4. Port mapping vs service publish?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
Host kXX-srv1:
  bridge_net (custom) ← app1, app2
  default_bridge ← app3
  host_mode ← app4
  macvlan ← app5 (host-like IP)
```

**Host:** kXX-srv1 192.168.1XX.10
**Apps:** nginx, busybox, alpine

### LANGKAH PRAKTIKUM (2 jam)

**1. Docker & Compose Install (10 menit)**
```bash
sudo apt install docker.io docker-compose
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER  # Logout/login
docker run hello-world
```

**2. Default Bridge Exploration (15 menit)**
```bash
docker network ls
docker network inspect bridge

# Run test containers
docker run -d --name web1 nginx
docker run -d --name web2 nginx
docker exec web1 ping web2  # Container-to-container OK
docker exec web1 ping host.docker.internal  # NO
curl localhost  # NO - port not published
```

**3. Custom Bridge Network (20 menit)**
```bash
docker network create --driver bridge my_bridge \
  --subnet=172.20.0.0/16 --gateway=172.20.0.1
docker network inspect my_bridge
```
```bash
# Run with custom network
docker run -d --name app1 --network my_bridge nginx
docker run -d --name app2 --network my_bridge nginx:alpine
docker exec app1 ping app2
docker exec app1 ping 172.20.0.1  # Gateway
```

**4. Port Publishing & Host Mode (20 menit)**
```bash
# Port mapping
docker run -d --name web_public -p 8080:80 nginx
curl localhost:8080

# Host network (no isolation)
docker run -d --network host --name host_nginx nginx
curl localhost:80  # Direct host port
docker network inspect host
```

**5. Macvlan Network (20 menit)**
```bash
docker network create -d macvlan \
  --subnet=192.168.1XX.0/24 --gateway=192.168.1XX.1 \
  -o parent=enp1s0 mac_net
```
```bash
docker run -d --name mac_container --network mac_net \
  --ip 192.168.1XX.50 alpine sleep 3600
ping 192.168.1XX.50  # Dari host seperti device lain
```

**6. Docker Compose Multi-Service (25 menit)**
```bash
mkdir ~/docker_lab && cd ~/docker_lab
nano docker-compose.yml
```
```yaml
version: '3.8'
services:
  web:
    image: nginx
    networks:
      app_net:
        ipv4_address: 172.30.0.10
    ports:
      - "8081:80"

  db:
    image: mariadb
    networks:
      - app_net
    environment:
      MYSQL_ROOT_PASSWORD: example

networks:
  app_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.30.0.0/24
```
```bash
docker-compose up -d
docker-compose ps
docker-compose logs web
curl localhost:8081
docker exec web ping db  # Container discovery
```

**7. Cleanup & Verification (10 menit)**
```bash
docker network ls -q | xargs docker network rm  # Custom networks
docker-compose down
docker system prune -f
docker stats  # Resource usage
```

### UJI KONFIGURASI
```
**Success Criteria:**
- 4 network types tested (bridge, custom, host, macvlan)
- Docker Compose 2 services + custom subnet
- Container-to-container ping OK
- Port publish accessible
- Macvlan IP pingable dari host
```

**Screenshot Wajib (12 gambar):**
1. `docker network ls`
2. Custom bridge inspect
3. Container ping container
4. Port mapping curl
5. Host network localhost
6. Macvlan network + ping
7. docker-compose.yml
8. `docker-compose ps`
9. Compose logs
10. `docker stats`
11. Network cleanup
12. Final `docker network ls`

### PERTANYAAN SEKITAR PRAKTIKUM
1. Macvlan security risk apa dibanding bridge?
2. Docker Compose `depends_on` vs network discovery?
3. Jika container tidak connect custom network, cek subnet conflict?
4. Overlay network kapan digunakan?

### CHECKLIST TUGAS MINGGU 9
- [ ] Docker + Compose installed
- [ ] 4 network modes tested
- [ ] Docker Compose multi-service
- [ ] 12 screenshot lengkap
- [ ] Cleanup OK

**Waktu Total:** 2 jam
**Output:** Container networking mastery untuk Kubernetes Minggu 10
