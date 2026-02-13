
# MINGGU 10: KUBERNETES NETWORKING
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam)
**Kubernetes Networking Model:**
```
Pod-to-Pod: Same node (veth) / Cross node (Overlay)
Pod-to-Service: kube-proxy iptables/ipvs
External-to-Service: NodePort/LoadBalancer/Ingress
```

**CNI Plugins:**
| Plugin | Overlay | NetworkPolicy | Performance |
|--------|---------|---------------|-------------|
| Flannel | VXLAN | Basic | Good |
| Calico | BGP/IPIP | Advanced | Excellent |
| Weave | VXLAN | Good | Excellent |

### PERTANYAAN TEORI
1. Pod IP ephemeral vs Service IP stable kenapa?
2. kube-proxy modes: iptables vs ipvs?
3. NetworkPolicy default deny bagaimana?
4. Ingress vs LoadBalancer perbedaan?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
K3s Master (srv1) ← CNI → Worker Nodes (srv2,srv3)
                     ↓ Services/Ingress
                 Laptop kubectl/curl
```

**Hosts:**
```
K3s Master: kXX-srv1 192.168.1XX.10
K3s Worker1: kXX-srv2 192.168.1XX.11
K3s Worker2: kXX-srv3 192.168.1XX.12
```

**Aplikasi:**
```
k3s, kubectl, calico/flannel CNI
```

### LANGKAH PRAKTIKUM (2 jam)

**1. K3s Cluster Installation (25 menit)**
```bash
# srv1: Master
curl -sfL https://get.k3s.io | sh -
sudo kubectl get nodes

# srv2,srv3: Workers
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1XX.10:6443 \
  K3S_TOKEN=$(sudo cat /var/lib/rancher/k3s/server/node-token) sh -
```

**2. CNI Plugin Calico (15 menit)**
```bash
# Master: Install Calico
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml

# Verify
kubectl get pods -n calico-system
kubectl get daemonsets calico-node
```

**3. Basic Deployment & Service (20 menit)**
```bash
# Nginx deployment
kubectl create deployment nginx --image=nginx --replicas=3
kubectl expose deployment nginx --port=80 --type=ClusterIP

# Verify
kubectl get deployments,pods,svc
kubectl get endpoints nginx
```

**4. Service Types Testing (20 menit)**
```bash
# NodePort
kubectl expose deployment nginx --name nginx-nodeport --type=NodePort --port=80
kubectl get svc nginx-nodeport
NODE_PORT=$(kubectl get svc nginx-nodeport -o jsonpath='{.spec.ports[0].nodePort}')
curl 192.168.1XX.11:$NODE_PORT

# LoadBalancer simulation
kubectl expose deployment nginx --name nginx-lb --type=LoadBalancer
```

**5. Network Policies (25 menit)**
```bash
# Allow nginx → mariadb only
nano policy.yaml
```
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: nginx-egress
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: mariadb
```
```bash
kubectl apply -f policy.yaml
kubectl describe networkpolicy nginx-egress
```

**6. Ingress Controller (15 menit)**
```bash
# Ingress nginx
nano ingress.yaml
```
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
kubectl apply -f ingress.yaml
kubectl get ingress
```

### UJI KONFIGURASI
```
kubectl get all
NAME                          READY   STATUS
pod/nginx-xxx                 1/1     Running
service/nginx                 ClusterIP
service/nginx-nodeport        NodePort   80:3XXXX/TCP

kubectl exec nginx-pod -- curl nginx-service:80  # Pod-to-Service OK
```

**Screenshot Wajib (12 gambar):**
1. `kubectl get nodes`
2. Calico pods running
3. Deployment + service list
4. NodePort curl output
5. NetworkPolicy describe
6. Ingress resource
7. `kubectl top nodes`
8. `kubectl top pods`
9. Calico daemonset
10. Endpoints nginx
11. Exec pod-to-service
12. Custom dashboard Grafana (opsional)

### PERTANYAAN SEKITAR PRAKTIKUM
1. Jika pod cross-node tidak connect, cek CNI?
2. NetworkPolicy label selector salah efeknya?
3. LoadBalancer type di single node cluster?
4. kubectl port-forward vs NodePort?

### CHECKLIST TUGAS MINGGU 10
- [ ] K3s 3-node cluster
- [ ] Calico CNI operational
- [ ] Deployment + 3 service types
- [ ] NetworkPolicy applied
- [ ] Ingress controller
- [ ] 12 screenshot lengkap

**Waktu Total:** 2 jam
**Output:** Kubernetes cluster dengan networking lengkap
