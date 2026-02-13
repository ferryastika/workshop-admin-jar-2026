
# MINGGU 6: FILE SERVICES & NETWORK STORAGE
## WORKSHOP ADMIN JARINGAN - PENS TI 2026 [file:1]

### DASAR TEORI (1 jam)
**File Sharing Protocols:**
| Protocol | OS Native | Security | Performance |
|----------|-----------|----------|-------------|
| NFSv4 | Linux/Unix | Kerberos | Excellent |
| SMB3/CIFS | Windows/Linux | NTLM/Kerberos | Good |
| iSCSI | Block storage | CHAP | Highest |

**NFS vs SMB:**
```
NFS: User/group ID based (Unix)
SMB: User authentication (AD integration)
```

**Storage Types:**
- NAS: File level (NFS/SMB)
- SAN: Block level (iSCSI)

### PERTANYAAN TEORI
1. NFS permission 755 vs SMB NTFS ACL?
2. Apa risiko NFS tanpa Kerberos?
3. Bedakan NAS vs SAN performance?
4. Kapan pakai rsync vs NFS untuk backup?

### KEBUTUHAN PRAKTIKUM
**Topologi:**
```
Client srv2, srv3 ← NFS/SMB → srv1 (File Server)
                            ↓ Backup
                        rsync cron → /backup
```

**Hosts:**
```
File Server: kXX-srv1 192.168.1XX.10
NFS/SMB Client: kXX-srv2 192.168.1XX.11
Backup Target: kXX-srv1:/backup
```

**Aplikasi:**
```
nfs-kernel-server, samba, rsync, cron
```

### LANGKAH PRAKTIKUM (2 jam)

**1. NFS Server (25 menit)**
```bash
sudo apt install nfs-kernel-server
sudo mkdir -p /srv/nfs/share /srv/nfs/secure
sudo chown nobody:nogroup /srv/nfs/share
sudo chmod 777 /srv/nfs/share
sudo chmod 755 /srv/nfs/secure
```
```bash
sudo nano /etc/exports
```
```
/srv/nfs/share 192.168.1XX.0/24(rw,sync,no_subtree_check)
/srv/nfs/secure 192.168.1XX.11(rw,sync,no_subtree_check)
```
```bash
sudo exportfs -ra
sudo exportfs -v
sudo systemctl restart nfs-kernel-server
showmount -e 192.168.1XX.10
```

**2. NFS Client srv2 (15 menit)**
```bash
sudo apt install nfs-common
showmount -e 192.168.1XX.10
sudo mkdir /mnt/nfs_share /mnt/nfs_secure
sudo mount 192.168.1XX.10:/srv/nfs/share /mnt/nfs_share
sudo mount 192.168.1XX.10:/srv/nfs/secure /mnt/nfs_secure
df -h | grep nfs
echo "NFS Test" > /mnt/nfs_share/test.txt
cat /mnt/nfs_share/test.txt  # srv1
```

**3. Samba SMB Server (25 menit)**
```bash
sudo apt install samba
sudo systemctl enable smbd
```
```bash
sudo useradd -M smbuser
sudo smbpasswd -a smbuser
```
```bash
sudo nano /etc/samba/smb.conf
```
```
[global]
   workgroup = PENS
   server string = Kelompok XX File Server

[public]
   path = /srv/smb/public
   browseable = yes
   writable = yes
   guest ok = yes
   read only = no

[secure]
   path = /srv/smb/secure
   valid users = smbuser
   browseable = yes
   writable = yes
   read only = no
```
```bash
sudo mkdir -p /srv/smb/public /srv/smb/secure
sudo chmod 777 /srv/smb/public
sudo testparm
sudo systemctl restart smbd
```

**4. SMB Client Test (15 menit)**
```bash
# srv2 mount SMB
sudo apt install cifs-utils
sudo mkdir /mnt/smb_public /mnt/smb_secure
sudo mount -t cifs //192.168.1XX.10/public /mnt/smb_public -o guest
sudo mount -t cifs //192.168.1XX.10/secure /mnt/smb_secure -o username=smbuser
echo "SMB Test" > /mnt/smb_public/smb.txt
```

**5. Automated Backup rsync + cron (20 menit)**
```bash
# srv1 backup directory
sudo mkdir /backup
sudo chown root:root /backup
```
```bash
# Backup script
sudo nano /usr/local/bin/backup.sh
```
```bash
#!/bin/bash
rsync -avz --delete /srv/nfs/ /backup/nfs/
rsync -avz --delete /srv/smb/ /backup/smb/
find /backup -mtime +7 -delete
```
```bash
sudo chmod +x /usr/local/bin/backup.sh
sudo crontab -e
```
```
0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

**6. Verification (10 menit)**
```bash
# NFS status
exportfs -v
showmount -e localhost

# SMB status
sudo smbstatus
testparm -s

# Cron backup test
sudo /usr/local/bin/backup.sh
ls -la /backup/
tail /var/log/backup.log
```

### UJI KONFIGURASI
```
**NFS mount expected:**
/mnt/nfs_share on /srv/nfs/share type nfs (rw,...)

**SMB mount expected:**
/mnt/smb_public on //srv1/public type cifs (rw,...)

**Backup log:**
2026/02/13 14:30:01 rsync: /srv/nfs → /backup/nfs OK
```

**Screenshot Wajib (12 gambar):**
1. `/etc/exports` NFS
2. `showmount -e`
3. srv2 `df -h` NFS mounts
4. `smb.conf` sections
5. `smbstatus` connections
6. srv2 SMB mounts
7. `/usr/local/bin/backup.sh`
8. crontab listing
9. `/backup/` contents
10. `/var/log/backup.log`
11. `exportfs -v`
12. Browser file listing

### PERTANYAAN SEKITAR PRAKTIKUM
1. Jika NFS mount permission denied, cek `/etc/exports` options?
2. SMB guest access risiko security apa?
3. Kenapa rsync `--delete` di backup script?
4. Cron job gagal, cek log mana?

### CHECKLIST TUGAS MINGGU 6
- [ ] NFS share public + secure
- [ ] SMB public + authenticated
- [ ] srv2 mount NFS + SMB OK
- [ ] rsync backup script + cron
- [ ] 12 screenshot lengkap

**Waktu Total:** 2 jam
**Output:** Multi-protocol file server + automated backup
