
# MINGGU 5: WEB SERVICES & REVERSE PROXY
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam)
**Web Server Comparison:**
| Server | Pros | Cons | Best Use |
|--------|------|------|----------|
| Apache | Modules, .htaccess | Memory heavy | Legacy sites |
| Nginx | Async, low memory | Learning curve | Proxy, high traffic |
| Caddy | Auto HTTPS | Newer | Modern apps |

**Reverse Proxy Concepts:**
```
Client → Nginx (443) → Backend1:8080
                    ↘ Backend2:8080 (Load Balance)
```

**TLS/SSL Modern:**
- Let's Encrypt ACME v2
- HTTP/2 multiplexing
- HSTS preload
- OCSP Stapling

### PERTANYAAN TEORI
1. Nginx event-driven vs Apache process model?
2. Apa fungsi `proxy_pass` vs `proxy_cache`?
3. Kenapa HTTP/2 butuh ALPN?
4. Bedakan TLS termination vs TLS passthrough?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
Laptop/Browser → Nginx Proxy:443/80 (srv1)
                    ↓ upstream
Backend srv2:8080 + srv3:8080 (Node.js apps)
                    ↓ DNS BIND9
                srv1:53 (Minggu 3)
```

**Hosts:**
```
Reverse Proxy: kXX-srv1 192.168.1XX.10
Backend 1: kXX-srv2 192.168.1XX.11:8080
Backend 2: kXX-srv3 192.168.1XX.12:8080
```

**Aplikasi:**
```
nginx, nodejs (simple HTTP server), certbot
```

### LANGKAH PRAKTIKUM (2 jam)

**1. Backend Servers Setup (20 menit)**
```bash
# srv2 & srv3: Simple Node.js server
sudo apt install -y nodejs npm
cat > app.js << 'EOF'
const http = require('http');
const server = http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end(`Hello from Backend ${process.env.HOSTNAME}`);
});
server.listen(8080);
EOF
HOSTNAME=$(hostname) node app.js &
```

**2. Nginx Installation (5 menit)**
```bash
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
curl localhost
```

**3. Basic Virtual Host (15 menit)**
```bash
sudo nano /etc/nginx/sites-available/kelompokXX
```
```
server {
    listen 80;
    server_name kelompokXX.lab;

    location / {
        root /var/www/html;
        index index.html index.htm;
    }
}
```
```bash
sudo ln -s /etc/nginx/sites-available/kelompokXX /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

**4. Reverse Proxy Configuration (25 menit)**
```bash
sudo nano /etc/nginx/sites-available/proxy
```
```
upstream backend_pool {
    least_conn;
    server 192.168.1XX.11:8080;
    server 192.168.1XX.12:8080;
}

server {
    listen 80;
    server_name proxy.kelompokXX.lab;

    location / {
        proxy_pass http://backend_pool;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    }
}
```
```bash
sudo ln -s /etc/nginx/sites-available/proxy /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

**5. SSL/TLS Let's Encrypt (25 menit)**
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d kelompokXX.lab -d proxy.kelompokXX.lab
# Email: [email lab]
# Agree TOS ✓
```

**6. Load Balancing Test (15 menit)**
```bash
# Test round-robin
for i in {1..10}; do curl -H "Host: proxy.kelompokXX.lab" proxy.kelompokXX.lab; done

# HTTPS test
curl -k https://kelompokXX.lab
curl -k -H "Host: proxy.kelompokXX.lab" https://kelompokXX.lab
```

### UJI KONFIGURASI
```
**curl proxy.kelompokXX.lab expected:**
Hello from Backend k02-srv1
Hello from Backend k02-srv2
(alternating load balance)

**curl -k https://kelompokXX.lab:**
Welcome to Nginx HTTPS!
```

**Screenshot Wajib (10 gambar):**
1. Backend app.js srv2
2. nginx.conf virtual host
3. upstream pool config
4. certbot success
5. `nginx -t` OK
6. Load balance curl 10x
7. HTTPS curl output
8. `ss -tulpn | nginx`
9. Browser localhost
10. Browser HTTPS proxy

### PERTANYAAN SEKITAR PRAKTIKUM
1. Jika load balancing stuck 1 backend, cek `least_conn`?
2. Kenapa certbot butuh domain resolvable?
3. Apa fungsi `proxy_set_header X-Real-IP`?
4. Jika HTTPS 502, kemungkinan TLS termination?

### CHECKLIST TUGAS MINGGU 5
- [ ] 2 backend Node.js running
- [ ] Nginx virtual host + proxy
- [ ] Load balancing working
- [ ] Let's Encrypt SSL certificate
- [ ] HTTPS proxy functional
- [ ] 10 screenshot lengkap

**Waktu Total:** 2 jam
**Output:** Production-ready web infrastructure dengan SSL
