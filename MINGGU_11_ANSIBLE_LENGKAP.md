
# MINGGU 11: NETWORK AUTOMATION ANSIBLE
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam)
**IaC Principles:**
```
Declarative: "State yang diinginkan"
Idempotent: Run berkali-kali = hasil sama
Versioned: Git untuk config changes
Auditable: Log semua perubahan
```

**Ansible Architecture:**
```
Playbook → Inventory → Modules → Target Hosts
                    ↓ SSH
                (Agentless!)
```

### PERTANYAAN TEORI
1. Ansible agentless vs Puppet agent-based?
2. Inventory dynamic vs static?
3. Playbook vs Role structure?
4. Idempotency di network config?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
Control Node (laptop) ← SSH → srv1 (.10), srv2 (.11), srv3 (.12)
                                    ↓ Automation
                               BIND9, Nginx, Monitoring
```

**Inventory:** srv1-3 kelompokXX
**Target:** Deploy DNS + Web dari Minggu sebelumnya

### LANGKAH PRAKTIKUM (2 jam)

**1. Ansible Installation Laptop (10 menit)**
```bash
sudo apt update
sudo apt install ansible sshpass python3-paramiko
ansible --version
```

**2. Inventory & SSH Setup (15 menit)**
```bash
mkdir ~/ansible_lab && cd ~/ansible_lab
nano inventory.ini
```
```
[kelompokXX]
srv1 ansible_host=192.168.1XX.10 ansible_user=adminXX
srv2 ansible_host=192.168.1XX.11 ansible_user=adminXX
srv3 ansible_host=192.168.1XX.12 ansible_user=adminXX

[kelompokXX:vars]
ansible_ssh_pass=PASSWORD_XX
ansible_become=yes
ansible_become_pass=PASSWORD_XX
```
```bash
# Test connectivity
ansible kelompokXX -i inventory.ini -m ping
ansible kelompokXX -i inventory.ini -m setup | head -20
```

**3. Ad-hoc Commands (10 menit)**
```bash
# Gather facts
ansible kelompokXX -i inventory.ini -m setup -a 'filter=ansible_*eth*' | grep 1s0

# Package install
ansible kelompokXX -i inventory.ini -m apt -a 'name=vim state=present'

# Service status
ansible kelompokXX -i inventory.ini -m service -a 'name=ssh state=started'
```

**4. Playbook DNS Deployment (25 menit)**
```bash
nano dns-playbook.yml
```
```yaml
---
- name: Deploy BIND9 DNS Server
  hosts: srv1
  become: yes
  tasks:
    - name: Install BIND9
      apt:
        name: ['bind9', 'bind9utils', 'dnsutils']
        state: present
        update_cache: yes

    - name: Create zones directory
      file:
        path: /etc/bind/zones
        state: directory
        owner: bind
        group: bind
        mode: '0755'

    - name: Deploy forward zone
      template:
        src: db.kelompokXX.j2
        dest: /etc/bind/zones/db.kelompokXX
        owner: bind
        group: bind
        mode: '0644'
      notify: restart bind9

    - name: Start BIND9
      service:
        name: bind9
        state: started
        enabled: yes
  handlers:
    - name: restart bind9
      service:
        name: bind9
        state: restarted
```
```bash
mkdir templates
nano templates/db.kelompokXX.j2  # Isi sama Minggu 3
ansible-playbook -i inventory.ini dns-playbook.yml
```

**5. Multi-Host Playbook (25 menit)**
```bash
nano full-stack.yml
```
```yaml
---
- name: Deploy Full Stack Infrastructure
  hosts: kelompokXX
  become: yes
  vars:
    domain: kelompokXX.lab
  tasks:
    - name: Update all
      apt: update_cache=yes

    - name: Install monitoring stack
      apt:
        name: 
          - prometheus
          - grafana
          - node-exporter
        state: present

    - name: Start services
      service:
        name: "{{ item }}"
        state: started
        enabled: yes
      loop:
        - prometheus
        - grafana-server
        - node_exporter

    - name: Open firewall
      ufw:
        rule: allow
        port: "{{ item }}"
      loop:
        - 9090
        - 3000
        - 9100
```
```bash
ansible-playbook -i inventory.ini full-stack.yml
```

**6. Roles Structure (20 menit)**
```bash
mkdir -p roles/bind9/{tasks,templates,handlers,defaults}
nano roles/bind9/tasks/main.yml
```
```yaml
---
- name: Install BIND9
  apt: name=bind9 state=present

- name: Deploy zone files
  template: src=db.j2 dest=/etc/bind/zones/db.{{ domain }}
```
```bash
ansible-galaxy init roles/bind9  # Template structure
ansible-playbook -i inventory.ini dns-playbook.yml --extra-vars "role_path=roles/bind9"
```

### UJI KONFIGURASI
```
ansible-playbook --check → No changes = idempotent ✓
ansible srv1 -m service_facts | grep bind9 → enabled ✓
```

**Screenshot Wajib (12 gambar):**
1. `ansible kelompokXX -m ping`
2. Inventory.ini
3. dns-playbook.yml
4. Playbook execution OK
5. srv1 `systemctl status bind9`
6. full-stack.yml
7. Multi-host execution
8. Roles structure
9. `--check` idempotent
10. `ansible-inventory --list`
11. Ad-hoc package install
12. Service facts output

### PERTANYAAN SEKITAR PRAKTIKUM
1. Ansible vault untuk password production?
2. Dynamic inventory dari DHCP leases?
3. Role vs playbook kapan?
4. `--check --diff` output interpretasi?

### CHECKLIST TUGAS MINGGU 11
- [ ] Ansible + inventory 3 hosts
- [ ] Ad-hoc commands working
- [ ] DNS playbook deployment
- [ ] Multi-service playbook
- [ ] Roles structure created
- [ ] 12 screenshot lengkap

**Waktu Total:** 2 jam
**Output:** Automated infrastructure deployment capability
