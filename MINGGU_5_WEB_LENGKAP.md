# MINGGU 5: WEB SERVICES & REVERSE PROXY
## WORKSHOP ADMIN JARINGAN - PENS TI 2026

### DASAR TEORI (1 jam)
**Reverse Proxy Concepts:**
```
Client → Nginx srv1:80/443 → Backend A srv2:8081
                            → Backend B srv2:8082
```

Dua backend dijalankan sebagai proses/container terpisah pada `srv2`. Tujuannya tetap sama: memahami upstream pool, health, proxying, dan load balancing tanpa membutuhkan VM ketiga.

### PERTANYAAN TEORI
1. Nginx event-driven vs Apache process model?
2. Apa fungsi `proxy_pass` vs `proxy_cache`?
3. Apa beda TLS termination dan TLS passthrough?
4. Dalam production, kapan dua backend sebaiknya ditempatkan pada host berbeda?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
Laptop/Browser → Nginx Proxy srv1:80/443
                    ↓ upstream
          srv2:8081 + srv2:8082
                    ↓
              DNS srv1:53
```

**Hosts:**
```
Reverse Proxy: kXX-srv1 192.168.1XX.10
Backend Host:  kXX-srv2 192.168.1XX.11
Backend A:     192.168.1XX.11:8081
Backend B:     192.168.1XX.11:8082
```

### LANGKAH PRAKTIKUM (2 jam)

**1. Backend Servers Setup pada srv2 (20 menit)**
```bash
sudo apt update
sudo apt install -y nodejs npm
mkdir -p ~/web-lab && cd ~/web-lab
cat > app.js << 'EOF'
const http = require('http');
const port = process.env.PORT || 8081;
const name = process.env.BACKEND || 'backend';
http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end(`Hello from ${name} on ${require('os').hostname()}:${port}\n`);
}).listen(port);
EOF

PORT=8081 BACKEND=backend-A nohup node app.js > backend-A.log 2>&1 &
PORT=8082 BACKEND=backend-B nohup node app.js > backend-B.log 2>&1 &
ss -lntp | grep -E ':8081|:8082'
```

**2. Nginx Installation pada srv1 (10 menit)**
```bash
sudo apt install -y nginx
sudo systemctl enable --now nginx
```

**3. Reverse Proxy Configuration (25 menit)**
```bash
sudo nano /etc/nginx/sites-available/proxy
```
```nginx
upstream backend_pool {
    least_conn;
    server 192.168.1XX.11:8081;
    server 192.168.1XX.11:8082;
}

server {
    listen 80;
    server_name proxy.kelompokXX.lab;

    location / {
        proxy_pass http://backend_pool;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
```bash
sudo ln -sf /etc/nginx/sites-available/proxy /etc/nginx/sites-enabled/proxy
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

**4. DNS Record (10 menit)**
Tambahkan record pada zone Minggu 3:
```
proxy   IN A 192.168.1XX.10
```
Lalu reload BIND9 dan verifikasi dengan `dig`.

**5. Load Balancing Test (20 menit)**
```bash
for i in {1..10}; do
  curl -H 'Host: proxy.kelompokXX.lab' http://192.168.1XX.10/
done
```
Expected output bergantian antara backend-A dan backend-B.

**6. TLS untuk Lab (20 menit)**
Untuk domain internal `.lab`, gunakan sertifikat self-signed agar praktikum tidak bergantung pada DNS publik/ACME.
```bash
sudo mkdir -p /etc/nginx/tls
sudo openssl req -x509 -nodes -newkey rsa:2048 -days 30 \
  -keyout /etc/nginx/tls/kelompokXX.key \
  -out /etc/nginx/tls/kelompokXX.crt \
  -subj '/CN=proxy.kelompokXX.lab'
```
Tambahkan server block HTTPS pada Nginx dan uji:
```bash
curl -k https://proxy.kelompokXX.lab
```

### UJI KONFIGURASI
- `nginx -t` sukses.
- Port 8081 dan 8082 aktif pada srv2.
- Request melalui srv1 mencapai kedua backend.
- HTTPS internal dapat diakses dengan `curl -k`.

### CHECKLIST TUGAS MINGGU 5
- [ ] Backend A dan B berjalan pada srv2
- [ ] Nginx reverse proxy pada srv1
- [ ] Load balancing working
- [ ] DNS proxy record working
- [ ] TLS lab functional

**Output:** Reverse proxy dan load balancing dipahami tanpa menambah VM ketiga.
