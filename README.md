# Linux server guide
## `Update linux`
```bash
apt update
```
```bash
apt dist-upgrade
```
## `Using automatic updates`
```bash
apt install unattended-upgrades
```
```bash
dpkg-reconfigure --priority=low unattended-upgrades
```
## `Add a limited user account`
```bash
useradd -m -s /bin/bash jay && passwd jay
```
```bash
apt install sudo
```
```bash
visudo
```
Now search in this file for groups that can execute sudo commands(groups begin with %) and add your user in that group.
Suppose the group name is sudo. Then execute
```bash
usermod -aG sudo jay
```
See the groups for that user to verify
```bash
groups jay
```
Switch to that user
```bash
su - jay
```
## `Setup SSH login`
Execute the following commands on your `local linux terminal`. Make sure you don't have a key named id_rsa
```bash
cd .ssh && ssh-keygen
```
Send the .pub file to the cloud server
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub jay@1.2.3.4
```
Now `login to your cloud server` disable root login and add a new line for user-ssh-login in the sshd-config
```bash
sudo nano /etc/ssh/sshd_config
```
edit `PermitRootLogin no` and add `AllowUsers jay`
```bash
sudo systemctl restart sshd
```
## `Setup Firewall`
```bash
apt install ufw
```
```bash
ufw default allow outgoing
```
```bash
ufw default deny incoming
```
```bash
ufw allow ssh
```
```bash
ufw enable
```
check status
```bash
ufw status
```
for http 
```bash
ufw allow 80
```













