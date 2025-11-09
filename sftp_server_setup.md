# 🛠️ Secure SFTP Server Setup using OpenSSH

This guide provides step-by-step instructions to configure a **secure SFTP-only server** on **CentOS / RHEL / Fedora** using **OpenSSH**.  
It isolates users in their own home directories (`chroot jail`) and disables SSH shell access for added security.

---

## 📋 Prerequisites

- Root or `sudo` privileges
- Internet access to install packages
- Tested on: **CentOS 8 / RHEL 8 / Fedora**

---

## 🚀 1. Install and Start OpenSSH Server

SFTP is built into OpenSSH.  
Install and enable the SSH service:

```bash
sudo dnf install -y openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
systemctl status sshd
```

✅ **Explanation:**
- Installs the OpenSSH server package.
- Enables it to start automatically at boot.
- Starts the SSH daemon immediately.
- Verifies the service status.

---

## 👥 2. Create a Dedicated SFTP Group

We'll isolate all SFTP users under a specific group.

```bash
sudo groupadd sftpusers
```

✅ **Explanation:**  
Creates a system group named `sftpusers` — only users in this group will have SFTP access.

---

## 👤 3. Create an SFTP-Only User

Create a new user restricted from shell access:

```bash
sudo useradd -m -g sftpusers -s /sbin/nologin username
sudo passwd username
```

✅ **Explanation:**  
- `-m`: Creates a home directory (`/home/username`)  
- `-g sftpusers`: Assigns the user to `sftpusers` group  
- `-s /sbin/nologin`: Disables shell access — user can’t SSH into the system  
- `passwd`: Sets the login password for SFTP authentication

---

## 🗂️ 4. Set Up the Directory Structure

SFTP users must be jailed to their own directories, owned by root, with a writable subfolder.

```bash
sudo chown root:root /home/username
sudo mkdir /home/username/uploads
sudo chown username:sftpusers /home/username/uploads
sudo chmod 755 /home/username
sudo chmod 755 /home/username/uploads
```

✅ **Explanation:**  
- The home directory must be owned by `root:root` (chroot security rule).  
- `uploads` folder is writable by the user for file transfers.  
- Permissions restrict access and prevent privilege escalation.

---

## ⚙️ 5. Configure SSHD for SFTP-Only Access

Edit SSH configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Add these lines **at the bottom**:

```bash
Match Group sftpusers
    ChrootDirectory /home/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

Also ensure this line exists (for SFTP subsystem):

```bash
Subsystem sftp internal-sftp
```

✅ **Explanation:**
- `Match Group sftpusers`: Applies these rules only to SFTP users.  
- `ChrootDirectory /home/%u`: Jails user inside `/home/username`.  
- `ForceCommand internal-sftp`: Forces SFTP-only (no SSH shell).  
- `AllowTcpForwarding` and `X11Forwarding`: Disabled for security.

---

## 🔄 6. Restart SSHD and Verify Configuration

```bash
sudo systemctl restart sshd
sudo journalctl -xeu sshd
```

✅ **Explanation:**
- Restarts the SSH service to apply new configuration.
- Checks logs for errors (useful if permissions or ownerships are wrong).

---

## 🧪 7. Test SFTP Access

From a client machine (Windows, Linux, or macOS):

```bash
sftp username@server_ip
```

✅ **Expected Behavior:**
- Prompts for password.
- Logs in successfully.
- User sees only their jailed directory (`/home/username`).
- Can upload/download inside `/uploads`.

---

## 🚫 8. (Optional) Restrict SSH Access to SFTP Users Only

If you want **only** SFTP users to log in:

```bash
AllowGroups sftpusers
```

Add the above line to `/etc/ssh/sshd_config`.

✅ **Caution:**  
If you use other SSH accounts (like `root` or `admin`), you must add their groups too; otherwise, they’ll be blocked.

---

## 🧱 Directory Ownership Summary

| Directory | Owner | Group | Permission | Description |
|------------|--------|--------|-------------|-------------|
| `/home/username` | root | root | 755 | Chroot jail root, not writable |
| `/home/username/uploads` | username | sftpusers | 755 | User’s writable directory |

---

## 🧮 Useful Commands

| Command | Description |
|----------|--------------|
| `sudo systemctl status sshd` | Check SSH service status |
| `sudo systemctl restart sshd` | Restart SSH service |
| `sudo journalctl -xeu sshd` | View SSHD error logs |
| `sftp username@server_ip` | Connect to server via SFTP |
| `sudo tail -f /var/log/secure` | Watch SSH login attempts |

---

## 🛩 Security Best Practices

- Disable password login and use **SSH keys** for trusted users.
- Disable root login (`PermitRootLogin no`).
- Use a **firewall** (`firewalld` or `ufw`) to allow only required ports.
- Monitor `/var/log/secure` for suspicious activity.
- Keep your system and OpenSSH package updated.

---

## 📜 License

This setup guide is open-source and available under the **MIT License**.  
Feel free to use and modify it for your own projects.

---
