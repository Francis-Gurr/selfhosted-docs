# Samba

Location: `fileserver` LXC in proxmox (CT100).

## Setup

### 1. Install Samba:
```bash
apt update
apt install samba
```

### 2. Create samba users:
Create users with `useradd`.
`-M`: Ensures that no home directory is created for the user. This is useful when creating system users who do not need a home directory for interactive login.
`-s /sbin/nologin`: Specifies the shell for the user. `/sbin/nologin` is a special shell that prevents the user from logging into the system interactively. This is commonly used for service accounts or accounts that only need access for specific purposes, such as running services or accessing Samba shares.
```bash
useradd -M -s /sbin/nologin francis
useradd -M -s /sbin/nologin leyna
```

Set samba password for each user using `smbpasswd -a`
```bash
smbpasswd -a francis
smbpasswd -a leyna
```

### 3. Add users to a group:
Create a single user group using `groupadd` for managing permissions for both shares.
```bash
groupadd samba-share
```

Add the users to the user group with `usermod`.
```bash
usermod -aG sambashare francis # -aG: append to supplementary group
usermod -aG sambashare leyna
```

### 4. Configure samba:
Edit the Samba configuration file to define both shares. Open /etc/samba/smb.conf in your text editor:
```bash
vim /etc/samba/smb.conf
```

Add the following sections at the end of the file:
```ini
[zfs-pool]
path = /zfs-pool
valid users = @sambashare
read only = yes
browsable = yes

[francis]
path = /zfs-pool/cloud/francis
valid users = francis
read only = no
browsable = yes

[leyna]
path = /zfs-pool/cloud/leyna
valid users = leyna
read only = no
browsable = yes

[shared]
path = /zfs-pool/cloud/shared
valid users = @sambashare
read only = no
browsable = yes
```

### 5. Set Folder Permissions:
Ensure both the folders have the correct permissions:

```bash
chown -R :sambashare /zfs-pool/cloud # change owner to sambashare
chmod -R 770 /zfs-pool/cloud # change mode, set permissions owner (5 - rx), group (5 - rx), others (0 - none).

chown -R francis /zfs-pool/cloud/francis # change owner to francis
chmod -R 750 /zfs-pool/cloud/francis # change mode, set permissions owner (7 - rwx), group (5 - rx), others (0 - none).

chown -R francis /zfs-pool/cloud/leyna # change owner to leyna
chmod -R 750 /zfs-pool/cloud/leyna # change mode, set permissions owner (7 - rwx), group (5 - rx), others (0 - none).
```

### 6. Restart Samba:
Restart the Samba services to apply the changes:

```bash
systemctl restart smbd # samba daemon
systemctl restart nmbd # NetBIOS name server daemon for Samba
```
