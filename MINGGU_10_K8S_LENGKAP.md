# MINGGU 10: KUBERNETES NETWORKING
## WORKSHOP ADMIN JARINGAN - PENS TI 2026

### DASAR TEORI (1 jam)
**Kubernetes Networking Model:**
```
Pod-to-Pod: veth + CNI
Pod-to-Service: ClusterIP
External-to-Service: NodePort / Ingress
```

Untuk baseline workshop digunakan **K3s 2-node**:
- `srv1`: K3s server/control-plane
- `srv2`: K3s agent/worker
- CNI: Flannel bawaan K3s

Custom CNI seperti Calico dibahas sebagai materi pengayaan, bukan dependency wajib lab.

### PERTANYAAN TEORI
1. Mengapa Pod IP bersifat ephemeral sedangkan Service IP stabil?
2. Apa beda ClusterIP, NodePort, dan Ingress?
3. Apa fungsi CNI?
4. Apa tradeoff cluster 2-node untuk teaching lab dibanding production cluster?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
Laptop
  ↓ kubectl/curl
srv1 (.10) K3s Server  ←→  srv2 (.11) K3s Agent
        \______ Flannel Pod Network ______/
```

### LANGKAH PRAKTIKUM (2 jam)

**1. Install K3s Server pada srv1 (20 menit)**
```bash
curl -sfL https://get.k3s.io | sh -
sudo kubectl get nodes
sudo cat /var/lib/rancher/k3s/server/node-token
```

**2. Join srv2 sebagai Agent (20 menit)**
Salin token dari srv1, kemudian pada srv2:
```bash
curl -sfL https://get.k3s.io | \
  K3S_URL=https://192.168.1XX.10:6443 \
  K3S_TOKEN='<TOKEN_DARI_SRV1>' sh -
```

Kembali ke srv1:
```bash
sudo kubectl get nodes -o wide
```
Expected: 2 node `Ready`.

**3. Verifikasi Networking Bawaan (15 menit)**
```bash
sudo kubectl get pods -A
sudo kubectl get nodes -o wide
ip link | grep flannel || true
```
Identifikasi Pod CIDR dan Service CIDR yang digunakan cluster.

**4. Deployment + ClusterIP (20 menit)**
```bash
sudo kubectl create deployment nginx --image=nginx --replicas=2
sudo kubectl expose deployment nginx --port=80 --type=ClusterIP
sudo kubectl get pods -o wide
sudo kubectl get svc nginx
```
Pastikan replica tersebar atau identifikasi node tempat pod berjalan.

**5. NodePort (15 menit)**
```bash
sudo kubectl expose deployment nginx \
  --name nginx-nodeport --type=NodePort --port=80
sudo kubectl get svc nginx-nodeport
```
Uji dari laptop ke alamat node dan NodePort yang diberikan.

**6. NetworkPolicy Dasar (20 menit)**
Buat pod client dan policy default-deny ingress untuk pod nginx:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-nginx-ingress
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
    - Ingress
```

```bash
sudo kubectl apply -f policy.yaml
sudo kubectl describe networkpolicy deny-nginx-ingress
```
Amati perubahan akses dan diskusikan bagaimana enforcement policy bekerja pada distribusi K3s yang digunakan.

**7. Ingress (20 menit)**
K3s menyediakan ingress controller bawaan pada instalasi default. Buat resource:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: nginx.kelompokXX.lab
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

```bash
sudo kubectl apply -f ingress.yaml
sudo kubectl get ingress
```
Tambahkan record DNS/hosts yang diperlukan lalu uji dari laptop.

### UJI KONFIGURASI
```bash
sudo kubectl get nodes
sudo kubectl get pods -o wide
sudo kubectl get svc
sudo kubectl get ingress
sudo kubectl get networkpolicy
```

### CHECKLIST TUGAS MINGGU 10
- [ ] K3s 2-node cluster
- [ ] srv1 server + srv2 agent `Ready`
- [ ] Pod-to-Pod/Service connectivity diuji
- [ ] ClusterIP + NodePort diuji
- [ ] NetworkPolicy diterapkan
- [ ] Ingress diuji

**Output:** Kubernetes networking dipraktikkan dengan cluster minimal dua node yang lebih ringan dan mudah direproduksi.
