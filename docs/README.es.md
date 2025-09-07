# Entrega Proyecto de Configuración de Servidores y Despliegue

`>` **Proyectos KeepCoding - Web 18:** 📁 [repos-kc-web-18.md](https://github.com/pablo-sch/pablo-sch/blob/main/docs/repos-kc-web-18.md)

`>` **Selecciona tu Idioma:** [Inglés](README.md) 🔄 [Alemán](README.de.md)

`>` **Documentos Adicionales:** [Script de Automatización](AUTOMATION.md) 🔄 [Configuración Docker](DOCKER.md)

## Objetivo del Proyecto

- Configurar un servidor Ubuntu en AWS EC2 desde cero para el despliegue de múltiples aplicaciones web
- Implementar arquitectura de microservicios con Nginx como proxy reverso
- Establecer medidas de seguridad básicas (SSL, Fail2ban, autenticación MongoDB)
- Documentar el proceso completo para futuras referencias y aprendizaje continuo
- Crear una infraestructura escalable y mantenible para aplicaciones web modernas

## Conocimientos Aprendidos y Trabajados

- **Administración de Sistemas Linux**: Gestión de usuarios, permisos, procesos y servicios en Ubuntu Server
- **Configuración de Servidores Web**: Setup y optimización de Nginx para sitios estáticos y proxy reverso
- **Gestión de Procesos Node.js**: Implementación de Supervisor para mantener aplicaciones Node.js en ejecución
- **Bases de Datos NoSQL**: Instalación, configuración y securización de MongoDB
- **Backend as a Service**: Despliegue y configuración de Parse Server
- **Seguridad de Servidores**: Implementación de SSL/TLS con Certbot, Fail2ban, y hardening de SSH
- **Infraestructura como Código**: Documentación de comandos para reproducibilidad
- **DevOps Básico**: Automatización de despliegues y gestión de servicios

## Detalles del Proyecto

Este proyecto documenta la configuración completa de un servidor de producción con las siguientes características:

- **Servidor Principal**: Ubuntu 24.04 LTS en AWS EC2
- **Aplicaciones Desplegadas**:
  - Sitio web estático con Bootstrap
  - Aplicación React con Redux
  - Chat en tiempo real con Node.js y Socket.io
  - API REST con Parse Server
- **Arquitectura**:
  - Nginx como servidor web principal y proxy reverso
  - MongoDB como base de datos principal
  - Supervisor para gestión de procesos Node.js
  - SSL/TLS para todas las aplicaciones
- **Dominios Configurados**: Múltiples subdominios con certificados SSL independientes

## Tecnologías Utilizadas

- **Sistema Operativo**: Ubuntu 24.04 LTS
- **Servidor Web**: Nginx
- **Runtime JavaScript**: Node.js (versiones LTS)
- **Base de Datos**: MongoDB (última versión estable)
- **Backend Framework**: Parse Server
- **Gestión de Procesos**: Supervisor
- **Certificados SSL**: Let's Encrypt (Certbot)
- **Seguridad**: Fail2ban
- **Control de Versiones**: Git
- **Cloud Provider**: AWS EC2

## Seguridad - Checklist

- [ ] Cambiar puerto SSH por defecto
- [ ] Configurar firewall (UFW)
- [ ] Instalar y configurar Fail2ban
- [ ] Habilitar autenticación en MongoDB
- [ ] Configurar SSL/TLS para todos los sitios
- [ ] Crear usuarios específicos para cada servicio
- [ ] Deshabilitar login root SSH
- [ ] Configurar backups automáticos
- [ ] Monitorear logs regularmente
- [ ] Mantener sistema actualizado

## Pasos para Crear el Servidor AWS

### 1. Configuración SSH y Seguridad Inicial

```bash
# Copiar clave SSH y configurar permisos
cp "path/to/your-key.pem" ~/
chmod 600 ~/your-key.pem
ssh -i ~/your-key.pem ubuntu@YOUR_SERVER_IP

# Configurar contraseña SSH
passwd

# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Habilitar autenticación por contraseña (si es necesario)
sudo nano /etc/ssh/sshd_config.d/enable-password-auth.conf
# Agregar:
# PasswordAuthentication yes
# KbdInteractiveAuthentication yes

# Cambiar puerto SSH (opcional)
sudo nano /etc/ssh/sshd_config
# Port YOUR_CUSTOM_PORT

sudo systemctl reload ssh
sudo systemctl status ssh
```

### 2. Crear Usuarios del Sistema

```bash
# Crear usuarios para cada servicio
sudo adduser react
sudo adduser node
sudo adduser parse

# Configurar contraseñas y bloquear login
sudo passwd node  # YOUR_SECURE_PASSWORD
sudo passwd parse    # YOUR_SECURE_PASSWORD
sudo passwd -l node
sudo passwd -l parse

# Configurar permisos de directorio
sudo chmod o+rx /home/node
sudo chmod o+rx /home/react
sudo chmod o+rx /home/parse
sudo chmod o+rx /home/ubuntu/
```

### 3. Instalación de Servicios Base

```bash
# Instalar Nginx y herramientas
sudo apt install nginx unzip git -y

# Iniciar y habilitar Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

### 4. Configuración Nginx - Sitio por Defecto

```bash
# Descargar y configurar sitio estático
wget https://github.com/startbootstrap/startbootstrap-personal/archive/gh-pages.zip
unzip gh-pages.zip
sudo mv startbootstrap-personal-gh-pages/* /var/www/html/

# Configurar sitio por defecto
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

### 5. Instalación Node.js con NVM

```bash
# Como usuario node
sudo -u node -i
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install --lts
nvm use --lts
logout

# Como usuario parse
sudo -u parse -i
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install 18
nvm alias default 18
logout
```

### 6. Despliegue de Aplicaciones

#### Aplicación React

```bash
# En máquina local - compilar React app
git clone https://github.com/git-user/react-app.git
cd react-app
npm install
npm run build

# Subir al servidor y configurar
scp -r -i ~/your-key.pem dist/ ubuntu@YOUR_SERVER_IP:~/temp-dist/  # CAMBIADO: build → dist
ssh -i ~/your-key.pem ubuntu@YOUR_SERVER_IP

# En el servidor - mover a directorio correcto
sudo mkdir -p /home/react
sudo mv ~/temp-dist /home/react/dist
sudo chown -R react:react /home/react/dist
sudo chmod o+rx /home/react

# Configurar Nginx para React
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

#### Aplicación Node.js

```bash
# Como usuario node
sudo -u node -i
git clone https://github.com/git-user/node-app
cd node-app
npm install
logout

# Configurar Nginx para Node.js
sudo nano /etc/nginx/sites-available/node
```

```nginx
server {
    listen 80;
    server_name node.your-domain.com;

    location ~ ^/(css|fonts|img|js|sounds) {
        root /home/node/node-app/public;  # CAMBIADO: /node → /node-app
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

### 7. Instalación y Configuración MongoDB

```bash
# Instalar MongoDB
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org

# Iniciar MongoDB
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod

# Crear usuario administrador
mongosh
```

```javascript
// En MongoDB shell - Crear usuario admin
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
# Habilitar autenticación
sudo nano /etc/mongod.conf
```

```yaml
# Agregar en mongod.conf
security:
  authorization: enabled
```

```bash
# Reiniciar MongoDB
sudo systemctl restart mongod

# Crear usuario para Parse
mongosh --authenticationDatabase admin -u admin -p
```

```javascript
// En MongoDB shell autenticado
use parsedb
db.createUser({
  user: "parse-user",
  pwd: "YOUR_PARSE_DB_PASSWORD",
  roles: [{ role: "readWrite", db: "parsedb" }]
})
exit
```

### 8. Configuración Parse Server

```bash
# Como usuario parse
sudo -u parse -i
git clone YOUR_PARSE_REPOSITORY
cd parse-app
git checkout 1.3.1
npm install

# Configurar variables de entorno
export DATABASE_URI=mongodb://parse-user:YOUR_PARSE_DB_PASSWORD@localhost:27017/parsedb
npm start
# Verificar que funcione, luego Ctrl+C
logout

# Configurar Nginx para Parse
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

### 9. Configuración Supervisor

```bash
# Instalar Supervisor
sudo apt install supervisor -y

# Configurar proceso node
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
# Configurar proceso parse
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
# Habilitar sitios Nginx
sudo ln -s /etc/nginx/sites-available/react /etc/nginx/sites-enabled/react
sudo ln -s /etc/nginx/sites-available/node /etc/nginx/sites-enabled/node
sudo ln -s /etc/nginx/sites-available/parse /etc/nginx/sites-enabled/parse

# Reiniciar servicios
sudo supervisorctl reread
sudo supervisorctl update
sudo nginx -t
sudo systemctl reload nginx
```

### 10. Configuración SSL con Certbot

```bash
# Instalar Certbot
sudo apt install certbot python3-certbot-nginx -y

# Obtener certificados SSL
sudo certbot --nginx -d your-domain.com
sudo certbot --nginx -d react.your-domain.com
sudo certbot --nginx -d node.your-domain.com
sudo certbot --nginx -d parse.your-domain.com

# Verificar renovación automática
sudo certbot renew --dry-run
```

### 11. Configuración Fail2ban

```bash
# Instalar Fail2ban
sudo apt install fail2ban -y

# Configurar Fail2ban
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
# Iniciar Fail2ban
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

## Comandos Útiles

```bash
# Verificar estado de servicios
sudo systemctl status nginx
sudo systemctl status mongod
sudo supervisorctl status

# Ver logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/supervisor/*.log

# Reiniciar servicios
sudo supervisorctl restart node
sudo supervisorctl restart parse
sudo systemctl reload nginx

# Verificar certificados SSL
sudo certbot certificates

# Conectar a MongoDB con autenticación
mongosh --authenticationDatabase admin -u admin -p
mongosh --authenticationDatabase parsedb -u parse-user -p

# Verificar procesos
ps aux | grep node
ps aux | grep nginx

# Verificar permisos de directorio
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

## Configuraciones de Archivos

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

## Contribuciones y Licencias

Proyecto bajo licencia MIT. Uso y distribución libres con atribución. No se aceptan contribuciones externas, pero las sugerencias son bienvenidas.
