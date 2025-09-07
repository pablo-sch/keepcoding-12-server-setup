# Projektabgabe: Server-Konfiguration und Deployment

`>` **KeepCoding Projekte - Web 18:** 📁 [repos-kc-web-18.md](https://github.com/pablo-sch/pablo-sch/blob/main/docs/repos-kc-web-18.md)

`>` **Sprache wählen:** [Englisch](README.md) 🔄 [Spanish](README.es.md)

`>` **Zusätzliche Dokumente:** [Automatisierungsskript](AUTOMATION.md) 🔄 [Docker-Konfiguration](DOCKER.md)

## Projektziel

- Konfiguration eines Ubuntu-Servers auf AWS EC2 von Grund auf für das Deployment mehrerer Webanwendungen
- Implementierung einer Microservices-Architektur mit Nginx als Reverse-Proxy
- Etablierung grundlegender Sicherheitsmaßnahmen (SSL, Fail2ban, MongoDB-Authentifizierung)
- Dokumentation des kompletten Prozesses für zukünftige Referenz und kontinuierliches Lernen
- Erstellung einer skalierbaren und wartbaren Infrastruktur für moderne Webanwendungen

## Erlernte und Angewandte Kenntnisse

- **Linux-Systemadministration**: Benutzerverwaltung, Berechtigungen, Prozesse und Dienste auf Ubuntu Server
- **Webserver-Konfiguration**: Setup und Optimierung von Nginx für statische Sites und Reverse-Proxy
- **Node.js-Prozessverwaltung**: Implementierung von Supervisor zur Aufrechterhaltung von Node.js-Anwendungen
- **NoSQL-Datenbanken**: Installation, Konfiguration und Sicherung von MongoDB
- **Backend as a Service**: Deployment und Konfiguration von Parse Server
- **Server-Sicherheit**: Implementierung von SSL/TLS mit Certbot, Fail2ban und SSH-Härtung
- **Infrastructure as Code**: Kommando-Dokumentation für Reproduzierbarkeit
- **Grundlegende DevOps**: Deployment-Automatisierung und Service-Management

## Projektdetails

Dieses Projekt dokumentiert die komplette Konfiguration eines Produktionsservers mit folgenden Charakteristika:

- **Hauptserver**: Ubuntu 24.04 LTS auf AWS EC2
- **Deployierte Anwendungen**:
  - Statische Website mit Bootstrap
  - React-Anwendung mit Redux
  - Echtzeit-Chat mit Node.js und Socket.io
  - REST-API mit Parse Server
- **Architektur**:
  - Nginx als Haupt-Webserver und Reverse-Proxy
  - MongoDB als Hauptdatenbank
  - Supervisor für Node.js-Prozessverwaltung
  - SSL/TLS für alle Anwendungen
- **Konfigurierte Domains**: Mehrere Subdomains mit unabhängigen SSL-Zertifikaten

## Verwendete Technologien

- **Betriebssystem**: Ubuntu 24.04 LTS
- **Webserver**: Nginx
- **JavaScript Runtime**: Node.js (LTS-Versionen)
- **Datenbank**: MongoDB (neueste stabile Version)
- **Backend Framework**: Parse Server
- **Prozessverwaltung**: Supervisor
- **SSL-Zertifikate**: Let's Encrypt (Certbot)
- **Sicherheit**: Fail2ban
- **Versionskontrolle**: Git
- **Cloud Provider**: AWS EC2

## Sicherheit - Checkliste

- [ ] Standard-SSH-Port ändern
- [ ] Firewall konfigurieren (UFW)
- [ ] Fail2ban installieren und konfigurieren
- [ ] MongoDB-Authentifizierung aktivieren
- [ ] SSL/TLS für alle Sites konfigurieren
- [ ] Spezifische Benutzer für jeden Service erstellen
- [ ] SSH-Root-Login deaktivieren
- [ ] Automatische Backups konfigurieren
- [ ] Logs regelmäßig überwachen
- [ ] System aktuell halten

## Schritte zur Erstellung des AWS-Servers

### 1. SSH-Konfiguration und erste Sicherheit

```bash
# SSH-Schlüssel kopieren und Berechtigungen konfigurieren
cp "path/to/your-key.pem" ~/
chmod 600 ~/your-key.pem
ssh -i ~/your-key.pem ubuntu@YOUR_SERVER_IP

# SSH-Passwort konfigurieren
passwd

# System aktualisieren
sudo apt update && sudo apt upgrade -y

# Passwort-Authentifizierung aktivieren (falls nötig)
sudo nano /etc/ssh/sshd_config.d/enable-password-auth.conf
# Hinzufügen:
# PasswordAuthentication yes
# KbdInteractiveAuthentication yes

# SSH-Port ändern (optional)
sudo nano /etc/ssh/sshd_config
# Port YOUR_CUSTOM_PORT

sudo systemctl reload ssh
sudo systemctl status ssh
```

### 2. Systembenutzer erstellen

```bash
# Benutzer für jeden Service erstellen
sudo adduser react
sudo adduser node
sudo adduser parse

# Passwörter konfigurieren und Login sperren
sudo passwd node  # YOUR_SECURE_PASSWORD
sudo passwd parse    # YOUR_SECURE_PASSWORD
sudo passwd -l node
sudo passwd -l parse

# Verzeichnisberechtigungen konfigurieren
sudo chmod o+rx /home/node
sudo chmod o+rx /home/react
sudo chmod o+rx /home/parse
sudo chmod o+rx /home/ubuntu/
```

### 3. Basis-Services Installation

```bash
# Nginx und Tools installieren
sudo apt install nginx unzip git -y

# Nginx starten und aktivieren
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

### 4. Nginx-Konfiguration - Standard-Site

```bash
# Statische Site herunterladen und konfigurieren
wget https://github.com/startbootstrap/startbootstrap-personal/archive/gh-pages.zip
unzip gh-pages.zip
sudo mv startbootstrap-personal-gh-pages/* /var/www/html/

# Standard-Site konfigurieren
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

### 5. Node.js-Installation mit NVM

```bash
# Als node-Benutzer
sudo -u node -i
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install --lts
nvm use --lts
logout

# Als parse-Benutzer
sudo -u parse -i
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install 18
nvm alias default 18
logout
```

### 6. Anwendungs-Deployment

#### React-Anwendung

```bash
# Auf lokaler Maschine - React-App kompilieren
git clone https://github.com/git-user/react-app.git
cd react-app
npm install
npm run build

# Auf Server hochladen und konfigurieren
scp -r -i ~/your-key.pem dist/ ubuntu@YOUR_SERVER_IP:~/temp-dist/
ssh -i ~/your-key.pem ubuntu@YOUR_SERVER_IP

# Auf Server - in korrektes Verzeichnis verschieben
sudo mkdir -p /home/react
sudo mv ~/temp-dist /home/react/dist
sudo chown -R react:react /home/react/dist
sudo chmod o+rx /home/react

# Nginx für React konfigurieren
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

#### Node.js-Anwendung

```bash
# Als node-Benutzer
sudo -u node -i
git clone https://github.com/git-user/node-app
cd node-app
npm install
logout

# Nginx für Node.js konfigurieren
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

### 7. MongoDB-Installation und -Konfiguration

```bash
# MongoDB installieren
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org

# MongoDB starten
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod

# Admin-Benutzer erstellen
mongosh
```

```javascript
// In MongoDB-Shell - Admin-Benutzer erstellen
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
# Authentifizierung aktivieren
sudo nano /etc/mongod.conf
```

```yaml
# In mongod.conf hinzufügen
security:
  authorization: enabled
```

```bash
# MongoDB neu starten
sudo systemctl restart mongod

# Benutzer für Parse erstellen
mongosh --authenticationDatabase admin -u admin -p
```

```javascript
// In authentifizierter MongoDB-Shell
use parsedb
db.createUser({
  user: "parse-user",
  pwd: "YOUR_PARSE_DB_PASSWORD",
  roles: [{ role: "readWrite", db: "parsedb" }]
})
exit
```

### 8. Parse Server-Konfiguration

```bash
# Als parse-Benutzer
sudo -u parse -i
git clone YOUR_PARSE_REPOSITORY
cd parse-app
git checkout 1.3.1
npm install

# Umgebungsvariablen konfigurieren
export DATABASE_URI=mongodb://parse-user:YOUR_PARSE_DB_PASSWORD@localhost:27017/parsedb
npm start
# Überprüfen ob es funktioniert, dann Ctrl+C
logout

# Nginx für Parse konfigurieren
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

### 9. Supervisor-Konfiguration

```bash
# Supervisor installieren
sudo apt install supervisor -y

# Node-Prozess konfigurieren
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
# Parse-Prozess konfigurieren
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
# Nginx-Sites aktivieren
sudo ln -s /etc/nginx/sites-available/react /etc/nginx/sites-enabled/react
sudo ln -s /etc/nginx/sites-available/node /etc/nginx/sites-enabled/node
sudo ln -s /etc/nginx/sites-available/parse /etc/nginx/sites-enabled/parse

# Services neu starten
sudo supervisorctl reread
sudo supervisorctl update
sudo nginx -t
sudo systemctl reload nginx
```

### 10. SSL-Konfiguration mit Certbot

```bash
# Certbot installieren
sudo apt install certbot python3-certbot-nginx -y

# SSL-Zertifikate erhalten
sudo certbot --nginx -d your-domain.com
sudo certbot --nginx -d react.your-domain.com
sudo certbot --nginx -d node.your-domain.com
sudo certbot --nginx -d parse.your-domain.com

# Automatische Erneuerung überprüfen
sudo certbot renew --dry-run
```

### 11. Fail2ban-Konfiguration

```bash
# Fail2ban installieren
sudo apt install fail2ban -y

# Fail2ban konfigurieren
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
# Fail2ban starten
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

## Nützliche Befehle

```bash
# Service-Status überprüfen
sudo systemctl status nginx
sudo systemctl status mongod
sudo supervisorctl status

# Logs anzeigen
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/supervisor/*.log

# Services neu starten
sudo supervisorctl restart node
sudo supervisorctl restart parse
sudo systemctl reload nginx

# SSL-Zertifikate überprüfen
sudo certbot certificates

# Mit MongoDB mit Authentifizierung verbinden
mongosh --authenticationDatabase admin -u admin -p
mongosh --authenticationDatabase parsedb -u parse-user -p

# Prozesse überprüfen
ps aux | grep node
ps aux | grep nginx

# Verzeichnisberechtigungen überprüfen
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

## Datei-Konfigurationen

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
directory=/home/parse/your-parse-app
autostart=true
autorestart=true
environment=DATABASE_URI="mongodb://parse-user:YOUR_PARSE_DB_PASSWORD@localhost:27017/parsedb",PATH="/home/parse/.nvm/versions/node/lts/bin:/usr/bin"
```

## Beiträge und Lizenzen

Projekt unter MIT-Lizenz. Freie Nutzung und Verteilung mit Namensnennung. Externe Beiträge werden nicht akzeptiert, Vorschläge sind jedoch willkommen.
