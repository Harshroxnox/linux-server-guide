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
For Nginx server run 
```bash
ufw allow "Nginx Full"
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
for https 
```bash
ufw allow 443
```
## `Python Virtual Env`
suppose you have python3.10 then run
```bash
apt install python3.10-virtualenv
```
```bash
python3 -m venv venv
```
```bash
source venv/bin/activate
```
## `Nodejs Install NVM`
```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```
reload bash
```bash
source ~/.bashrc
```
suppose you want to install lts/iron
```bash
nvm install lts/iron
```
You can also use `nvm ls` and `nvm install v16.20.0`
## `Docker Setup`
Installing steps on ubuntu 22.04 jammy
```bash
sudo apt update
```
```bash
sudo apt install -y apt-transport-https ca-certificates curl gnupg
```
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/dockerce.gpg
```
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/dockerce.gpg] https://download.docker.com/linux/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/dockerce.list > /dev/null
```
```bash
sudo apt update
```
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
## `Setting Up Systemd Services`
create a service
```bash
nano /etc/systemd/system/your_service.service
```
```bash
[Unit]
Description=<description about this service>

[Service]
User=<user e.g. root>
WorkingDirectory=<directory_of_script e.g. /root>
ExecStart=<script which needs to be executed>

[Install]
WantedBy=multi-user.target
```
For python venv scripts
```bash
[Unit]
Description=<project description>

[Service]
User=<user e.g. root>
WorkingDirectory=<path to your project directory containing your python script>
ExecStart=/home/user/.virtualenv/bin/python main.py
# replace /home/user/.virtualenv/bin/python with your virtualenv and main.py with your script

[Install]
WantedBy=multi-user.target
```
reload daemon
```bash
sudo systemctl daemon-reload
```
Managing services
```bash
sudo systemctl start your-service.service
```
```bash
sudo systemctl stop your-service.service
```
```bash
sudo systemctl status your-service.service
```
```bash
sudo systemctl enable your-service.service
```
```bash
sudo systemctl restart your-service.service
```
## `NGINX`
```bash
apt install nginx
```
```bash
ufw allow "Nginx Full"
```
Build the frontend static files and paste that folder(in this case dist) inside /var/www/ 
Delete the default file inside sites-available and sites-enabled
Edit Nginx configuration file 
```bash
nano /etc/nginx/sites-available/filename
```
Here is a sample nginx file having both frontend and backend routes set up
```bash
server {
  listen 80;
  # This is for the frontend serving the built files inside /var/www/dist
  location / {
        root /var/www/dist;
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }

  # This is for all the backend routes running on localhost:5000 having routes /question /registration etc.
  location /question {
        proxy_pass http://127.0.0.1:5000/question;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
  }

  location /registration {
        proxy_pass http://127.0.0.1:5000/registration;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
  }
}
```
This assumes backend is running on localhost:5000 and has routes /question /registration. Modify accordingly.
```bash
ln -s /etc/nginx/sites-available/filename /etc/nginx/sites-enabled/filename
```
```bash
systemctl restart nginx
```
## `SSL Certification`
```bash
apt install certbot python3-certbot-nginx
```

Make sure that Nginx Full rule is available
```bash
ufw status
```

```bash
certbot --nginx -d example.com -d www.example.com
```

Let’s Encrypt’s certificates are only valid for ninety days. To set a timer to validate automatically:
```bash
systemctl status certbot.timer
```
