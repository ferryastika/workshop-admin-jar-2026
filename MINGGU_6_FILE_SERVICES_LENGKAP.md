# MINGGU 6: FILE SERVICES & NETWORK STORAGE
## WORKSHOP ADMIN JARINGAN - PENS TI 2026

### DASAR TEORI (1 jam)
**File Sharing Protocols:**
| Protocol | Model | Use Case |
|---|---|---|
| NFSv4 | Unix/Linux file sharing | Linux-to-Linux |
| SMB3/CIFS | Authenticated file sharing | Windows/Linux interoperability |
| rsync | File synchronization | Backup/copy |

### PERTANYAAN TEORI
1. Apa beda NFS dan SMB dari sisi authentication dan permission?
2. Apa risiko export NFS terlalu permisif?
3. Kapan rsync lebih tepat daripada mounted network filesystem?
4. Apa fungsi `sync` pada NFS export?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
srv2 (client) ← NFS/SMB → srv1 (file server)
                            ↓
                       local backup /backup
```

**Hosts:**
```
File Server: kXX-srv1 192.168.1XX.10
Client:      kXX-srv2 192.168.1XX.11
```

### LANGKAH PRAKTIKUM (2 jam)

**1. NFS Server pada srv1 (25 menit)**
```bash
sudo apt install -y nfs-kernel-server
sudo mkdir -p /srv/nfs/share /srv/nfs/secure
sudo chown nobody:nogroup /srv/nfs/share
sudo chmod 777 /srv/nfs/share
sudo chmod 755 /srv/nfs/secure
```
`/etc/exports`:
```
/srv/nfs/share 192.168.1XX.0/24(rw,sync,no_subtree_check)
/srv/nfs/secure 192.168.1XX.11(rw,sync,no_subtree_check)
```
```bash
sudo exportfs -ra
sudo exportfs -v
```

**2. NFS Client pada srv2 (20 menit)**
```bash
sudo apt install -y nfs-common
sudo mkdir -p /mnt/nfs_share /mnt/nfs_secure
sudo mount 192.168.1XX.10:/srv/nfs/share /mnt/nfs_share
sudo mount 192.168.1XX.10:/srv/nfs/secure /mnt/nfs_secure
echo 'NFS Test' | sudo tee /mnt/nfs_share/test.txt
```

**3. Samba pada srv1 (25 menit)**
```bash
sudo apt install -y samba
sudo mkdir -p /srv/smb/public /srv/smb/secure
sudo chmod 777 /srv/smb/public
sudo useradd -M smbuser || true
sudo smbpasswd -a smbuser
```
Tambahkan share public dan secure pada `/etc/samba/smb.conf`, kemudian:
```bash
sudo testparm
sudo systemctl restart smbd
```

**4. SMB Client pada srv2 (20 menit)**
```bash
sudo apt install -y cifs-utils
sudo mkdir -p /mnt/smb_public /mnt/smb_secure
sudo mount -t cifs //192.168.1XX.10/public /mnt/smb_public -o guest
sudo mount -t cifs //192.168.1XX.10/secure /mnt/smb_secure -o username=smbuser
```

**5. Backup rsync pada srv1 (20 menit)**
```bash
sudo mkdir -p /backup
sudo nano /usr/local/bin/backup.sh
```
```bash
#!/bin/bash
set -euo pipefail
rsync -a --delete /srv/nfs/ /backup/nfs/
rsync -a --delete /srv/smb/ /backup/smb/
```
```bash
sudo chmod +x /usr/local/bin/backup.sh
sudo /usr/local/bin/backup.sh
```

**6. Verification (10 menit)**
```bash
exportfs -v
sudo smbstatus
df -h | grep -E 'nfs|cifs'
ls -la /backup
```

### CHECKLIST TUGAS MINGGU 6
- [ ] NFS server pada srv1
- [ ] NFS client pada srv2
- [ ] SMB public + authenticated share
- [ ] srv2 mount NFS + SMB
- [ ] rsync backup berhasil

**Output:** Client/server file service dipraktikkan dengan dua VM.
