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
[cloud]
path = /zfs-pool/cloud
valid users = @sambashare
read only = no
browsable = yes
force group = sambashare
create mask = 0770
directory mask = 0770
```

### 5. Set Folder Permissions:
Ensure both the folders have the correct permissions:

```bash
chown -R francis:sambashare /zfs-pool/cloud/francis # Sets francis as the owner and sambashare as the group for francis's directory.
chmod -R 750 /zfs-pool/cloud/francis

chown -R leyna:sambashare /zfs-pool/cloud/leyna # Sets leyna as the owner and sambashare as the group for leyna's directory.
chmod -R 750 /zfs-pool/cloud/leyna

chown -R :sambashare /zfs-pool/cloud/shared
chmod -R 770 /zfs-pool/cloud/shared

# 750 permissions:
  # 7 (Owner): Full permissions (read, write, execute).
  # 5 (Group): Read and execute permissions.
  # 0 (Others): No permissions.
```

### 6. Configure ACLs
Use Access Control Lists (ACLs) to define default permissions for all new files and directories in /zfs-pool/cloud/shared.

```
apt install acl

# Set the setgid (set group ID) bit on the cloud/shared directory.
# This ensures that new files and subdirectories created within cloud/shared inherit the group ownership (sambashare).
chmod g+s /zfs-pool/cloud/shared

# Set default ACLs for the shared directory:
setfacl -R -m d:g:sambashare:rwX /zfs-pool/cloud/shared

# Apply the same permissions to existing files:
setfacl -R -m g:sambashare:rwX /zfs-pool/cloud/shared

# Verify ACLs:
getfacl /zfs-pool/cloud/shared
```

### 7. Restart Samba:
Restart the Samba services to apply the changes:

```bash
systemctl restart smbd # samba daemon
systemctl restart nmbd # NetBIOS name server daemon for Samba
```
