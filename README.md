# 📦 Inventory App – Despliegue en AWS EC2 (AWS Academy Lab)

Este proyecto muestra cómo desplegar la aplicación **Inventory** en una instancia **Amazon EC2** dentro del entorno de laboratorio de **AWS Academy**.  

---

## 🚀 Tecnologías
- **AWS EC2 (Amazon Linux 2023)**
- **Node.js 18**
- **Nginx** como reverse proxy
- **Systemd** para levantar la app como servicio
- **AWS Academy Learner Lab** (sandbox educativo)

---

## 🔑 Pasos realizados

### 1. Ingreso al entorno de AWS Academy
1. Desde el **Learner Lab**, abrí el panel **AWS Details**.  
2. Descargué y abrí la URL de **AWS SSO**, que me llevó a la **AWS Management Console**.  
3. Usé la región **us-east-1** (la habilitada por el lab).  

---

### 2. Creación de la instancia EC2
1. En la consola, busqué **EC2** y entré a **Instances → Launch instances**.  
2. Configuración seleccionada:
   - **AMI**: Amazon Linux 2023  
   - **Tipo de instancia**: `t3.micro`  
   - **Par de claves**: omitido (no lo usé porque todo se configuró con *user data*)  
   - **Firewall (Security Group)**:  
     - ✅ SSH  
     - ✅ HTTP 

---

### 3. Configuración con *User Data*
En la sección **Advanced details → User data** agregué el siguiente script:  

```bash
#!/bin/bash
set -euxo pipefail

REPO_URL="https://github.com/daanimelian/inventory"

# Instalar dependencias
dnf update -y
dnf install -y git
curl -fsSL https://rpm.nodesource.com/setup_18.x | bash -
dnf install -y nodejs nginx

# Descargar aplicación
git clone "$REPO_URL" /opt/inventory || { rm -rf /opt/inventory; git clone "$REPO_URL" /opt/inventory; }
cd /opt/inventory
npm ci --only=production || npm install --production

# Crear servicio systemd
cat >/etc/systemd/system/inventory.service <<'UNIT'
[Unit]
Description=Inventory Node App
After=network.target

[Service]
User=nobody
WorkingDirectory=/opt/inventory
ExecStart=/usr/bin/node server.js
Restart=always
Environment=PORT=3000 NODE_ENV=production

[Install]
WantedBy=multi-user.target
UNIT

systemctl daemon-reload
systemctl enable --now inventory

# Configurar Nginx como proxy en 80
cat >/etc/nginx/conf.d/inventory.conf <<'NG'
server {
  listen 80;
  server_nam_
