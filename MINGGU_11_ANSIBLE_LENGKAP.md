# MINGGU 11: NETWORK AUTOMATION ANSIBLE
## WORKSHOP ADMIN JARINGAN - PENS TI 2026

### DASAR TEORI (1 jam)
**IaC Principles:** declarative, idempotent, versioned, auditable.

**Topologi:**
```
Laptop / Control Node
        ↓ SSH
  srv1 (.10)   srv2 (.11)
```

Dua managed host sudah cukup untuk membuktikan inventory, multi-host execution, variable, role, idempotency, dan error handling.

### PERTANYAAN TEORI
1. Ansible agentless vs agent-based configuration management?
2. Inventory static vs dynamic?
3. Playbook vs Role?
4. Mengapa idempotency penting untuk network/system administration?

### KEBUTUHAN PRAKTIKUM
**Inventory:** `srv1` dan `srv2` kelompok XX.

### LANGKAH PRAKTIKUM (2 jam)

**1. Install Ansible pada laptop (10 menit)**
```bash
sudo apt update
sudo apt install -y ansible sshpass
ansible --version
```

**2. Inventory (15 menit)**
```ini
[kelompokXX]
srv1 ansible_host=192.168.1XX.10 ansible_user=adminXX
srv2 ansible_host=192.168.1XX.11 ansible_user=adminXX
```
Gunakan SSH key bila memungkinkan. Password plaintext hanya untuk skenario lab sementara.

```bash
ansible kelompokXX -i inventory.ini -m ping
```

**3. Ad-hoc Commands (15 menit)**
```bash
ansible kelompokXX -i inventory.ini -m setup -a 'filter=ansible_default_ipv4'
ansible kelompokXX -i inventory.ini -b -m apt -a 'name=vim state=present update_cache=yes'
ansible kelompokXX -i inventory.ini -b -m service -a 'name=ssh state=started'
```

**4. Playbook DNS untuk srv1 (25 menit)**
```yaml
---
- name: Configure DNS server
  hosts: srv1
  become: true
  tasks:
    - name: Install BIND9
      apt:
        name:
          - bind9
          - bind9utils
        state: present
        update_cache: true

    - name: Ensure BIND9 running
      service:
        name: bind9
        state: started
        enabled: true
```

**5. Multi-Host Baseline (25 menit)**
```yaml
---
- name: Common baseline
  hosts: kelompokXX
  become: true
  tasks:
    - name: Install common tools
      apt:
        name:
          - curl
          - vim
          - prometheus-node-exporter
        state: present
        update_cache: true

    - name: Enable exporter
      service:
        name: prometheus-node-exporter
        state: started
        enabled: true
```

**6. Host-specific Tasks (20 menit)**
Gunakan group/host variable atau conditional:
```yaml
- name: Install nginx only on srv1
  apt:
    name: nginx
    state: present
  when: inventory_hostname == 'srv1'
```
Diskusikan mengapa role-based structure lebih baik daripada conditional yang terlalu banyak.

**7. Idempotency Test (15 menit)**
```bash
ansible-playbook -i inventory.ini baseline.yml
ansible-playbook -i inventory.ini baseline.yml
ansible-playbook -i inventory.ini baseline.yml --check --diff
```
Run kedua seharusnya menghasilkan perubahan minimal/tidak ada bila playbook idempotent.

### UJI KONFIGURASI
- `ansible kelompokXX -m ping`: 2/2 success.
- Playbook dapat membedakan task srv1 dan srv2.
- Run kedua idempotent.

### CHECKLIST TUGAS MINGGU 11
- [ ] Inventory 2 hosts
- [ ] SSH connectivity OK
- [ ] Ad-hoc command berhasil
- [ ] Playbook single-host dan multi-host
- [ ] Idempotency test
- [ ] Struktur role/vars dipahami

**Output:** Automation multi-host tanpa membutuhkan VM ketiga.
