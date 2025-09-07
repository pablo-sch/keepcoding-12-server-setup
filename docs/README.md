# Server Configuration and Deployment Submission

`>` **KeepCoding Projects - Web 18:** 📁 [repos-kc-web-18.md](https://github.com/pablo-sch/pablo-sch/blob/main/docs/repos-kc-web-18.md)

`>` **Select Your Language:** [Spanish](README.es.md) 🔄 [German](README.de.md)

`>` **Additional Documents:** [Automation Script](AUTOMATION.md) 🔄 [Docker Configuration](DOCKER.md)

## Project Objective

- Configure an Ubuntu server on AWS EC2 from scratch for deploying multiple web applications
- Implement microservices architecture with Nginx as reverse proxy
- Establish basic security measures (SSL, Fail2ban, MongoDB authentication)
- Document the complete process for future reference and continuous learning
- Create a scalable and maintainable infrastructure for modern web applications

## Knowledge Learned and Applied

- **Linux System Administration**: User management, permissions, processes and services on Ubuntu Server
- **Web Server Configuration**: Setup and optimization of Nginx for static sites and reverse proxy
- **Node.js Process Management**: Implementation of Supervisor to keep Node.js applications running
- **NoSQL Databases**: Installation, configuration and securing of MongoDB
- **Backend as a Service**: Deployment and configuration of Parse Server
- **Server Security**: Implementation of SSL/TLS with Certbot, Fail2ban, and SSH hardening
- **Infrastructure as Code**: Command documentation for reproducibility
- **Basic DevOps**: Deployment automation and service management

## Project Details

This project documents the complete configuration of a production server with the following characteristics:

- **Main Server**: Ubuntu 24.04 LTS on AWS EC2
- **Deployed Applications**:
  - Static website with Bootstrap
  - React application with Redux
  - Real-time chat with Node.js and Socket.io
  - REST API with Parse Server
- **Architecture**:
  - Nginx as main web server and reverse proxy
  - MongoDB as main database
  - Supervisor for Node.js process management
  - SSL/TLS for all applications
- **Configured Domains**: Multiple subdomains with independent SSL certificates

## Technologies Used

- **Operating System**: Ubuntu 24.04 LTS
- **Web Server**: Nginx
- **JavaScript Runtime**: Node.js (LTS versions)
- **Database**: MongoDB (latest stable version)
- **Backend Framework**: Parse Server
- **Process Management**: Supervisor
- **SSL Certificates**: Let's Encrypt (Certbot)
- **Security**: Fail2ban
- **Version Control**: Git
- **Cloud Provider**: AWS EC2

## Security - Checklist

- [ ] Change default SSH port
- [ ] Configure firewall (UFW)
- [ ] Install and configure Fail2ban
- [ ] Enable MongoDB authentication
- [ ] Configure SSL/TLS for all sites
- [ ] Create specific users for each service
- [ ] Disable SSH root login
- [ ] Configure automatic backups
- [ ] Monitor logs regularly
- [ ] Keep system updated

## Steps to Create AWS Server

### 1. SSH Configuration and Initial Security

```bash
# Copy SSH key and configure permissions
cp "path/to/your-key.pem" ~/
chmod 600 ~/your-key.pem
ssh -i ~/your-key.pem ubuntu@YOUR_SERVER_IP

# Configure SSH password
passwd

# Update system
sudo apt update && sudo apt upgrade -y

# Enable password authentication (if necessary)
sudo nano /etc/ssh/sshd_config.d/enable-password-auth.conf
# Add:
# PasswordAuthentication yes
# KbdInteractiveAuthentication yes

# Change SSH port (optional)
sudo nano /etc/ssh/sshd_config
# Port YOUR_CUSTOM_PORT

sudo systemctl reload ssh
sudo systemctl status ssh
```

### 2. Create System Users

```bash
# Create users for each service
sudo adduser react
sudo adduser node
sudo adduser parse

# Configure passwords and lock login
sudo passwd node  # YOUR_SECURE_PASSWORD
sudo passwd parse    # YOUR_SECURE_PASSWORD
sudo passwd -l node
sudo passwd -l parse

# Configure directory permissions
sudo chmod o+rx /home/node
sudo chmod o+rx /home/react
sudo chmod o+rx /home/parse
sudo chmod o+rx /home/ubuntu/
```

### 3. Base Services Installation

```bash
# Install Nginx and tools
sudo apt install nginx unzip git -y

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

### 4. Nginx Configuration - Default Site

```bash
# Download and configure static site
wget https://github.com/startbootstrap/startbootstrap-personal/archive/gh-pages.zip
unzip gh-pages.zip
sudo mv startbootstrap-personal-gh-pages/* /var/www/html/

# Configure default site
sudo nano /etc/nginx/sites-available/default
```

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
    server_name your-domain.com;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 5. Node.js Installation with NVM

```bash
# As node user
sudo -u node -i
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install --lts
nvm use --lts
logout

# As parse user
sudo -u parse -i
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install 18
nvm alias default 18
logout
```

### 6. Application Deployment

#### React Application

```bash
# On local machine - compile React app
git clone https://github.com/git-user/react-app.git
cd react-app
npm install
npm run build

# Upload to server and configure
scp -r -i ~/your-key.pem dist/ ubuntu@YOUR_SERVER_IP:~/temp-dist/
ssh -i ~/your-key.pem ubuntu@YOUR_SERVER_IP

# On server - move to correct directory
sudo mkdir -p /home/react
sudo mv ~/temp-dist /home/react/dist
sudo chown -R react:react /home/react/dist
sudo chmod o+rx /home/react

# Configure Nginx for React
sudo nano /etc/nginx/sites-available/react
```

```nginx
server {
    server_name react.your-domain.com;
    listen 80;
    root /home/react/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

#### Node.js Application

```bash
# As node user
sudo -u node -i
git clone https://github.com/git-user/node-app
cd node-app
npm install
logout

# Configure Nginx for Node.js
sudo nano /etc/nginx/sites-available/node
```

```nginx
server {
    listen 80;
    server_name node.your-domain.com;

    location ~ ^/(css|fonts|img|js|sounds) {
        root /home/node/node-app/public;
        access_log off;
        expires max;
        add_header X-Owner your-username;
    }

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass http://127.0.0.1:3000;
        proxy_redirect off;
    }
}
```

### 7. MongoDB Installation and Configuration

```bash
# Install MongoDB
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org

# Start MongoDB
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod

# Create admin user
mongosh
```

```javascript
// In MongoDB shell - Create admin user
use admin
db.createUser({
  user: "admin",
  pwd: "YOUR_ADMIN_PASSWORD",
  roles: [
    { role: "userAdminAnyDatabase", db: "admin" },
    { role: "readWriteAnyDatabase", db: "admin" }
  ]
})
exit
```

```bash
# Enable authentication
sudo nano /etc/mongod.conf
```

```yaml
# Add to mongod.conf
security:
  authorization: enabled
```

```bash
# Restart MongoDB
sudo systemctl restart mongod

# Create user for Parse
mongosh --authenticationDatabase admin -u admin -p
```

```javascript
// In authenticated MongoDB shell
use parsedb
db.createUser({
  user: "parse-user",
  pwd: "YOUR_PARSE_DB_PASSWORD",
  roles: [{ role: "readWrite", db: "parsedb" }]
})
exit
```

### 8. Parse Server Configuration

```bash
# As parse user
sudo -u parse -i
git clone YOUR_PARSE_REPOSITORY
cd parse-app
git checkout 1.3.1
npm install

# Configure environment variables
export DATABASE_URI=mongodb://parse-user:YOUR_PARSE_DB_PASSWORD@localhost:27017/parsedb
npm start
# Verify it works, then Ctrl+C
logout

# Configure Nginx for Parse
sudo nano /etc/nginx/sites-available/parse
```

```nginx
server {
    listen 80;
    server_name parse.your-domain.com;

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass http://127.0.0.1:1337/parse/;
        proxy_redirect off;
    }
}
```

### 9. Supervisor Configuration

```bash
# Install Supervisor
sudo apt install supervisor -y

# Configure node process
sudo nano /etc/supervisor/conf.d/node.conf
```

```ini
[program:node]
command=npm start
user=node
directory=/home/node/node-app
autostart=true
autorestart=true
environment=PATH="/home/node/.nvm/versions/node/lts/bin"
```

```bash
# Configure parse process
sudo nano /etc/supervisor/conf.d/parse.conf
```

```ini
[program:parse]
command=npm start
user=parse
directory=/home/parse/parse-app
autostart=true
autorestart=true
environment=DATABASE_URI="mongodb://parse-user:YOUR_PARSE_DB_PASSWORD@localhost:27017/parsedb",PATH="/home/parse/.nvm/versions/node/lts/bin:/usr/bin"
```

```bash
# Enable Nginx sites
sudo ln -s /etc/nginx/sites-available/react /etc/nginx/sites-enabled/react
sudo ln -s /etc/nginx/sites-available/node /etc/nginx/sites-enabled/node
sudo ln -s /etc/nginx/sites-available/parse /etc/nginx/sites-enabled/parse

# Restart services
sudo supervisorctl reread
sudo supervisorctl update
sudo nginx -t
sudo systemctl reload nginx
```

### 10. SSL Configuration with Certbot

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Obtain SSL certificates
sudo certbot --nginx -d your-domain.com
sudo certbot --nginx -d react.your-domain.com
sudo certbot --nginx -d node.your-domain.com
sudo certbot --nginx -d parse.your-domain.com

# Verify automatic renewal
sudo certbot renew --dry-run
```

### 11. Fail2ban Configuration

```bash
# Install Fail2ban
sudo apt install fail2ban -y

# Configure Fail2ban
sudo nano /etc/fail2ban/jail.local
```

```ini
[DEFAULT]
bantime = 10m
findtime = 10m
maxretry = 5

[sshd]
enabled = true
port = YOUR_SSH_PORT
filter = sshd
logpath = /var/log/auth.log

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /var/log/nginx/error.log

[nginx-limit-req]
enabled = true
filter = nginx-limit-req
port = http,https
logpath = /var/log/nginx/error.log
```

```bash
# Start Fail2ban
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

## Useful Commands

```bash
# Check service status
sudo systemctl status nginx
sudo systemctl status mongod
sudo supervisorctl status

# View logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/supervisor/*.log

# Restart services
sudo supervisorctl restart node
sudo supervisorctl restart parse
sudo systemctl reload nginx

# Check SSL certificates
sudo certbot certificates

# Connect to MongoDB with authentication
mongosh --authenticationDatabase admin -u admin -p
mongosh --authenticationDatabase parsedb -u parse-user -p

# Check processes
ps aux | grep node
ps aux | grep nginx

# Check directory permissions
ls -l /home/
```

## MongoDB Init Script (mongo-init.js)

```javascript
db = db.getSiblingDB("parsedb");

db.createUser({
  user: "parse-user",
  pwd: process.env.PARSE_PASSWORD,
  roles: [
    {
      role: "readWrite",
      db: "parsedb",
    },
  ],
});

db.createCollection("_dummy");
```

## File Configurations

### /etc/nginx/sites-available/default

```nginx
server {
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
    server_name your-domain.com;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### /etc/nginx/sites-available/react

```nginx
server {
    server_name react.your-domain.com;
    root /home/react/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}

server {
    listen 80;
    server_name YOUR_SERVER_IP;

    root /home/react/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### /etc/nginx/sites-available/node

```nginx
server {
    server_name node.your-domain.com;

    location ~ ^/(stylesheets|avatars|photos|images) {
        root /home/node/node-app/public/;
        access_log off;
        expires max;
        add_header X-Owner your-username;
    }

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass http://127.0.0.1:3000;
        proxy_redirect off;
    }
}
```

### /etc/nginx/sites-available/parse

```nginx
server {
    server_name parse.your-domain.com;
    location / {
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass http://127.0.0.1:1337/parse/;
        proxy_redirect off;
    }
}
```

### /etc/supervisor/conf.d/node.conf

```ini
[program:node]
command=npm start
user=node
directory=/home/node/node-app
autostart=true
autorestart=true
environment=PATH="/home/node/.nvm/versions/node/lts/bin"
```

### /etc/supervisor/conf.d/parse.conf

```ini
[program:parse]
command=npm start
user=parse
directory=/home/parse/parse-app
autostart=true
autorestart=true
environment=DATABASE_URI="mongodb://parse-user:YOUR_PARSE_DB_PASSWORD@localhost:27017/parsedb",PATH="/home/parse/.nvm/versions/node/lts/bin:/usr/bin"
```

## Contributions and Licenses

Project under MIT license. Free use and distribution with attribution. External contributions are not accepted, but suggestions are welcome.
