# Ubuntu Server Setup Notes
This guide provides step-by-step instructions for setting up and securing an `Ubuntu` server. It covers basic OS updates, SSH hardening, and security configurations such as automatic updates, firewall, and Fail2Ban setup.

### 1. Connect via SSH
To start, connect to your server using SSH as the root user:
```sh
ssh root@<ip-address>
```

### 2. Update the Operating System
Ensure your server is up-to-date by running the following commands to update and upgrade all installed packages:
```sh
sudo apt update && sudo apt upgrade -y
```
After the update, reboot the server to apply any pending changes:
```sh
sudo reboot
```

### 3. Setup Automatic Updates
To keep your system secure and up-to-date automatically, install the `unattended-upgrades`:
```sh
sudo apt install unattended-upgrades
```
Next, configure the automatic upgrade options by editing the configuration file:
```sh
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```
In the `Allowed-Origins` section, make sure the following lines are included for automatic security updates:
```
Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}";
        "${distro_id}:${distro_codename}-security";
        "${distro_id}ESMApps:${distro_codename}-apps-security";
        "${distro_id}ESM:${distro_codename}-infra-security";
//      "${distro_id}:${distro_codename}-updates";
//      "${distro_id}:${distro_codename}-proposed";
//      "${distro_id}:${distro_codename}-backports";
};
```
To set the frequency of automatic updates, edit another configuration file:
```sh
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```
Add or verify the following lines to enable daily package list updates, download, and install:
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

### 4. Harden SSH Access
To see which ports are currently being used by services on your server, use the following command:
```sh
ss -tulpn | grep LISTEN
```
Changing the default SSH port improves security by reducing the risk of automated attacks targeting port 22. To change the SSH port to `1429`, edit the SSH configuration file:
```sh
sudo nano /etc/ssh/sshd_config
```
Add the following line:
```sh
Port 1429
```
Restart the SSH service for the changes to take effect:
```sh
sudo systemctl restart ssh
sudo systemctl status ssh
exit
```
Reconnect to your server using the new SSH port:
```sh
ssh -p 1429 root@<ip-address>
```
For improved security, ensure that only SSH `Protocol 2` is used by editing the SSH configuration:
```sh
sudo nano /etc/ssh/sshd_config
```
Look for the Protocol directive and ensure it is set to `2`:
```sh
Protocol 2
```
Restart SSH service:
```sh
sudo systemctl restart ssh
```
For better security, it’s recommended to create a new user instead of using the root account. Create a new user and add them to the sudo group:
```sh
adduser gennadis
usermod -aG sudo gennadis
```
Reconnect to the server using this new user:
```sh
ssh -p 1429 gennadis@<ip-address>
```
Verify that the user has `sudo` privileges:
```sh
sudo ls -al /root
```
To enhance security, disable root SSH access by editing the SSH configuration file:
```sh
sudo nano /etc/ssh/sshd_config
```
Add or modify the following lines:
```sh
PermitRootLogin no
AllowUsers gennadis
```
Restart SSH to apply the changes:
```sh
sudo systemctl restart ssh
```
To further secure the server, use public key authentication instead of passwords. Authorize your public key by doing this from your **local** terminal:
```sh
cat ~/.ssh/id_ed25519.pub | ssh gennadis@<ip-config> "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"
```
Ensure the permissions for the authorized keys file are set correctly:
```sh
chmod 0600 /home/gennadis/.ssh/authorized_keys
```
Close the current SSH connection and login with the key:
```sh
ssh -i gennadis/.ssh/id_ed25519 -p 1429 gennadis@<ip-address>
```
**Note:** If everything is OK, the client opens the connection without asking for a password.

Now disable password-based authentication by editing the SSH configuration:
```sh
sudo nano /etc/ssh/sshd_config
```
Set `PasswordAuthentication` to `no`:
```sh
PasswordAuthentication no
```
Restart the SSH service:
```sh
sudo systemctl restart ssh
```

### 5. Simplify SSH Access (Client-Side)
To avoid typing in all the details each time you connect, you can simplify your SSH access by creating a configuration file on your local machine:
```sh
nano ~/.ssh/config
```
Add the following lines:
```sh
Host <ip-address>
  User gennadis
  IdentityFile ~/.ssh/id_ed25519
  IdentitiesOnly yes
  Port 1429
  ForwardAgent yes
  AddKeysToAgent yes
  UseKeychain yes
```
You can now reconnect using a simple SSH command:
```sh
ssh <ip-address>
```

### 6. Firewall Configuration
Install the `Uncomplicated Firewall` to protect your server:
```sh
sudo apt install ufw
```
Limit SSH access to the custom port `1429`:
```sh
sudo ufw limit 1429/tcp comment SSH
```
Check the current firewall rules:
```sh
sudo ufw show added
```
Enable UFW and verify its status:
```sh
sudo ufw enable
sudo ufw status
```

### 7. Fail2Ban Setup
`Fail2Ban` protects your server from brute-force attacks by banning IP addresses that exhibit suspicious behavior. Install `Fail2Ban`:
```sh
sudo apt install fail2ban -y
```
Enable Fail2Ban to start on boot:
```sh
sudo systemctl enable fail2ban
```
Create a local jail configuration file:
```sh
sudo touch /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```
Insert the following configuration to protect SSH:
```sh
[DEFAULT]
banaction = ufw
[sshd]
enabled = true
port=1429
maxretry = 3
findtime = 3600
bantime = 24h
ignoreip = 127.0.0.1
```
Restart `Fail2Ban` and check its status:
```sh
sudo systemctl restart fail2ban
sudo systemctl status fail2ban
```
These notes cover the essentials for setting up a secure and efficient Ubuntu server. Following these steps ensures your server is updated, SSH access is hardened, and both firewall and brute-force attack protection are in place.

### 8. [OPTIONAL] System Monitoring with journalctl for Systemd Logs
For services managed by `systemd`, logs are stored in a binary format and accessed via the `journalctl` tool. Over time, these logs can grow large, potentially consuming significant disk space.
You can view the current size of the `systemd` journal logs with the following command:
```sh
sudo journalctl --disk-usage
```
This command will display how much disk space the `systemd` journal is currently using. If you notice the logs consuming too much space, you can set a size limit.
1. Open the `journald` configuration file:
```sh
sudo nano /etc/systemd/journald.conf
```
2. Add or modify the following line to set the size limit:
```sh
SystemMaxUse=250M
```
3. Save the file and exit the editor.
4. Restart the `journald` service to apply the changes:
```sh
sudo systemctl restart systemd-journald
```
This will ensure that the `systemd` logs do not exceed the defined size limit and old logs are automatically deleted when the limit is reached.
