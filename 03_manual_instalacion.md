## MANUAL DE INSTALACIÓN Y DESPLIEGUE
### Administración de Sistemas y DevOps
### SVED + BiometriX - Sistema de Votación Electrónica Descentralizado
<a href="https://postimg.cc/7J12TtPw" target="_blank"><img src="https://i.postimg.cc/pr4Y43gh/logo.png" alt="logo"></a>

---

## 1. PRERREQUISITOS DEL SISTEMA

### 1.1 Requisitos de Hardware

| Componente | Mínimo | Recomendado |
|------------|--------|-------------|
| **CPU** | 4 núcleos a 2.0 GHz | 8+ núcleos a 2.5+ GHz |
| **RAM** | 8 GB | 16+ GB |
| **Almacenamiento** | 50 GB SSD | 100+ GB SSD |
| **Red** | 100 Mbps | 1 Gbps |

### 1.2 Requisitos de Software por Componente

| Componente | Tecnología | Versión | Notas |
|------------|------------|---------|-------|
| **Sistema Operativo** | Ubuntu / Debian | 22.04 LTS / 24.04 LTS | Recomendado para producción |
| **JDK** | Java | 21 LTS | OpenJDK o Oracle JDK |
| **Spring Boot** | Framework | 3.x | Incluido en el código |
| **Node.js** | JavaScript Runtime | 18.x / 20.x LTS | Para Angular frontend |
| **Angular CLI** | Framework Frontend | 15+ | Instalación global opcional |
| **PostgreSQL** | Base de Datos | 15.x | Para sved_db y biometrix_db |
| **Redis** | Cache / Tokens | 7.x | Para session tokens |
| **Hyperledger Fabric** | Blockchain | 2.5.x | Red descentralizada |
| **HashiCorp Vault** | Gestión de Secretos | 1.13+ | Almacenamiento de claves |
| **Python** | Motor de IA | 3.10+ | Para sidecar de embeddings |
| **Docker** | Contenedores | 24.x | Para despliegue (opcional) |
| **Docker Compose** | Orquestación | 2.x | Para entornos de desarrollo |
| **Git** | Control de Versiones | 2.x | Para clonar el repositorio |

### 1.3 Puertos de Red Requeridos

| Servicio | Puerto | Protocolo | Descripción |
|----------|--------|-----------|-------------|
| SVED Core | 8080 | HTTP/HTTPS | API Gateway y endpoints |
| BiometriX BaaS | 8081 | HTTP/HTTPS | API REST de autenticación biométrica |
| Angular Frontend | 4200 | HTTP | Desarrollo (producción usa Nginx) |
| PostgreSQL (sved_db) | 5432 | TCP | Base de datos de SVED |
| PostgreSQL (biometrix_db) | 5433 | TCP | Base de datos de BiometriX |
| Redis | 6379 | TCP | Cache de session tokens |
| Hyperledger Fabric Peers | 7051, 7052, 7053 | GRPC | Comunicación blockchain |
| Hyperledger Fabric Orderer | 7050 | GRPC | Servicio de consenso |
| HashiCorp Vault | 8200 | HTTP/HTTPS | API de gestión de secretos |
| Python Sidecar (gRPC) | 50051 | GRPC | Extracción de embeddings |

---

## 2. VARIABLES DE ENTORNO (.env.example)

### 2.1 SVED Core (.env.sved)

```env
# ========================================================
# SVED CORE - VARIABLES DE ENTORNO
# ========================================================

# SPRING CONFIGURATION
SPRING_PROFILES_ACTIVE=production
SERVER_PORT=8080
SERVER_SSL_ENABLED=true
SERVER_SSL_KEY_STORE=/etc/sved/certs/keystore.p12
SERVER_SSL_KEY_STORE_PASSWORD=${KEYSTORE_PASSWORD}
SERVER_SSL_KEY_ALIAS=sved-server

# SVED DATABASE (sved_db)
SVED_DB_HOST=localhost
SVED_DB_PORT=5432
SVED_DB_NAME=sved_db
SVED_DB_USER=sved_user
SVED_DB_PASSWORD=${SVED_DB_PASSWORD}

# BIOMETRIX INTEGRATION
BIOMETRIX_API_URL=https://localhost:8081/api/v1
BIOMETRIX_API_KEY=${BIOMETRIX_API_KEY}
BIOMETRIX_MTLS_ENABLED=true
BIOMETRIX_MTLS_KEY_STORE=/etc/sved/certs/biometrix-client.p12
BIOMETRIX_MTLS_KEY_STORE_PASSWORD=${BIOMETRIX_MTLS_PASSWORD}
BIOMETRIX_MTLS_TRUST_STORE=/etc/sved/certs/truststore.p12
BIOMETRIX_MTLS_TRUST_STORE_PASSWORD=${TRUSTSTORE_PASSWORD}

# HYPERLEDGER FABRIC
FABRIC_CHANNEL_NAME=electoral-channel-2025
FABRIC_CHAINCODE_NAME=voting-contract
FABRIC_ORDERER_URL=grpcs://localhost:7050
FABRIC_PEER_TSE_URL=grpcs://localhost:7051
FABRIC_PEER_FISCALIA_URL=grpcs://localhost:9051
FABRIC_PEER_OBSERVADOR_URL=grpcs://localhost:11051
FABRIC_TLS_CA_CERT=/etc/sved/fabric/tls-ca-cert.pem
FABRIC_USER_CERT=/etc/sved/fabric/admin-cert.pem
FABRIC_USER_KEY=/etc/sved/fabric/admin-key.pem

# JWT CONFIGURATION
JWT_SECRET=${JWT_SECRET}
JWT_EXPIRATION_MINUTES=5

# REDIS (Session Tokens)
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=${REDIS_PASSWORD}
REDIS_SSL=false

# LOGGING
LOG_LEVEL=INFO
LOG_FILE=/var/log/sved/sved.log
```

### 2.2 BiometriX  (.env.biometrix)

```env
# ========================================================
# BIOMETRIX - VARIABLES DE ENTORNO
# ========================================================

# SPRING CONFIGURATION
SPRING_PROFILES_ACTIVE=production
SERVER_PORT=8081
SERVER_SSL_ENABLED=true
SERVER_SSL_KEY_STORE=/etc/biometrix/certs/keystore.p12
SERVER_SSL_KEY_STORE_PASSWORD=${KEYSTORE_PASSWORD}
SERVER_SSL_KEY_ALIAS=biometrix-server

# BIOMETRIX DATABASE (biometrix_db)
BIOMETRIX_DB_HOST=localhost
BIOMETRIX_DB_PORT=5433
BIOMETRIX_DB_NAME=biometrix_db
BIOMETRIX_DB_USER=biometrix_user
BIOMETRIX_DB_PASSWORD=${BIOMETRIX_DB_PASSWORD}

# TENANT CONFIGURATION
DEFAULT_TENANT_NAME=SVED
DEFAULT_TENANT_API_KEY=${DEFAULT_TENANT_API_KEY}
DEFAULT_TENANT_MATCH_THRESHOLD=0.85
DEFAULT_TENANT_RATE_LIMIT=100

# VAULT (Claves AES-256)
VAULT_URL=https://localhost:8200
VAULT_TOKEN=${VAULT_TOKEN}
VAULT_APPROLE_ROLE_ID=${VAULT_ROLE_ID}
VAULT_APPROLE_SECRET_ID=${VAULT_SECRET_ID}
VAULT_SSL_ENABLED=true
VAULT_KEY_PATH=/biometrix/keys

# PYTHON SIDECAR (gRPC)
SIDECAR_GRPC_HOST=localhost
SIDECAR_GRPC_PORT=50051
SIDECAR_GRPC_TIMEOUT_MS=5000
SIDECAR_CIRCUIT_BREAKER_FAILURES=5
SIDECAR_CIRCUIT_BREAKER_TIMEOUT_MS=30000

# REDIS (Session Tokens)
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=${REDIS_PASSWORD}
REDIS_SSL=false

# LIVENESS THRESHOLDS
LIVENESS_FINGERPRINT_THRESHOLD=0.90
LIVENESS_FACIAL_THRESHOLD=0.90
LIVENESS_VOICE_THRESHOLD=0.85

# QUALITY THRESHOLDS
QUALITY_MIN_DPI=500
QUALITY_MIN_FACIAL_RESOLUTION=640x480
QUALITY_MIN_FACE_RATIO=0.30

# LOCKOUT CONFIGURATION
LOCKOUT_MAX_ATTEMPTS=3
LOCKOUT_DURATION_MINUTES=15

# LOGGING
LOG_LEVEL=INFO
LOG_FILE=/var/log/biometrix/biometrix.log
```

### 2.3 Python Sidecar (.env.python)

```env
# ========================================================
# PYTHON SIDECAR - VARIABLES DE ENTORNO
# ========================================================

# gRPC SERVER
GRPC_HOST=0.0.0.0
GRPC_PORT=50051
GRPC_MAX_WORKERS=4
GRPC_MAX_MESSAGE_SIZE=104857600  # 100 MB

# MODELS
FACE_MODEL_PATH=/opt/sved/python/models/facenet_model.pb
FACE_MODEL_TYPE=facenet  # facenet / insightface
EMBEDDING_DIMENSION=512

# LIVENESS MODELS
LIVENESS_MODEL_PATH=/opt/sved/python/models/pad_model.onnx
LIVENESS_FACIAL_MODEL=/opt/sved/python/models/pad_facial.onnx

# LOGGING
LOG_LEVEL=INFO
LOG_FILE=/var/log/sved/python-sidecar.log
```

---

## 3. PASO A PASO DE INSTALACIÓN

### 3.1 Instalación de Dependencias Base

**Paso 1:** Actualizar el sistema operativo

```bash
sudo apt update
sudo apt upgrade -y
```

**Paso 2:** Instalar dependencias básicas

```bash
sudo apt install -y wget curl git gnupg lsb-release ca-certificates apt-transport-https
```

**Paso 3:** Instalar Docker y Docker Compose

```bash
# Instalar Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Instalar Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 3.2 Instalación de PostgreSQL

**Paso 1:** Agregar el repositorio de PostgreSQL

```bash
# Instalar PostgreSQL 15 para Ubuntu 22.04
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt update
sudo apt install -y postgresql-15 postgresql-contrib-15
```

**Paso 2:** Iniciar y habilitar PostgreSQL

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

**Paso 3:** Crear bases de datos y usuarios

```bash
# Acceder a PostgreSQL
sudo -u postgres psql

# Crear usuario y base de datos para SVED
CREATE USER sved_user WITH PASSWORD 'sved_secure_password';
CREATE DATABASE sved_db OWNER sved_user;
GRANT ALL PRIVILEGES ON DATABASE sved_db TO sved_user;

# Crear usuario y base de datos para BiometriX
CREATE USER biometrix_user WITH PASSWORD 'biometrix_secure_password';
CREATE DATABASE biometrix_db OWNER biometrix_user;
GRANT ALL PRIVILEGES ON DATABASE biometrix_db TO biometrix_user;

# Salir de psql
\q
```

**Paso 4:** Configurar conexiones remotas (opcional)

```bash
sudo nano /etc/postgresql/15/main/postgresql.conf
# Cambiar: listen_addresses = '*'

sudo nano /etc/postgresql/15/main/pg_hba.conf
# Agregar: host all all 0.0.0.0/0 md5

sudo systemctl restart postgresql
```

### 3.3 Instalación de Redis

```bash
# Instalar Redis
sudo apt install -y redis-server

# Configurar Redis (opcional, habilitar persistencia)
sudo nano /etc/redis/redis.conf
# Cambiar: supervised systemd
# Cambiar: requirepass redis_secure_password

# Iniciar Redis
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

### 3.4 Instalación de HashiCorp Vault

**Paso 1:** Descargar e instalar Vault

```bash
# Descargar Vault
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install -y vault

# Iniciar Vault en modo desarrollo (NO para producción)
vault server -dev -dev-listen-address="0.0.0.0:8200" -dev-root-token-id="root-token"
```

**Paso 2:** Configurar Vault para BiometriX (producción)

```bash
# Configurar Vault
vault kv put -mount=secret biometrix/keys/tenant-sved \
  aes_key="aes-256-gcm-key-base64-encoded" \
  salt="unique-salt-per-tenant"
```

### 3.5 Instalación de Hyperledger Fabric

**Paso 1:** Descargar binarios de Fabric

```bash
# Descargar fabric-samples
git clone https://github.com/hyperledger/fabric-samples.git
cd fabric-samples

# Descargar binarios
curl -sSL https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh | bash -s -- binary

# Agregar binarios al PATH
export PATH=$PWD/bin:$PATH
```

**Paso 2:** Generar certificados y canal

```bash
# Crear estructura de directorios
mkdir -p /opt/sved/fabric/crypto-config
mkdir -p /opt/sved/fabric/channel-artifacts

# Generar certificados con cryptogen
cryptogen generate --config=./crypto-config.yaml --output=/opt/sved/fabric/crypto-config

# Crear genesis block
configtxgen -profile ElectoralChannelGenesis -outputBlock /opt/sved/fabric/channel-artifacts/genesis.block -channelID electoral-channel-2025

# Crear channel transaction
configtxgen -profile ElectoralChannel -outputCreateChannelTx /opt/sved/fabric/channel-artifacts/electoral-channel-2025.tx -channelID electoral-channel-2025

# Crear anchor peer updates
configtxgen -profile ElectoralChannel -outputAnchorPeersUpdate /opt/sved/fabric/channel-artifacts/TSEAnchor.tx -channelID electoral-channel-2025 -asOrg TSE
configtxgen -profile ElectoralChannel -outputAnchorPeersUpdate /opt/sved/fabric/channel-artifacts/FiscaliaAnchor.tx -channelID electoral-channel-2025 -asOrg Fiscalia
```

### 3.6 Instalación de Java y Node.js

```bash
# Instalar Java 21
sudo apt install -y openjdk-21-jdk
java -version

# Instalar Node.js 18 (LTS)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
npm -v

# Instalar Angular CLI
sudo npm install -g @angular/cli@15
```

### 3.7 Clonar y Configurar el Código Fuente

**Paso 1:** Clonar el repositorio

```bash
git clone https://github.com/your-org/sved-electoral-system.git
cd sved-electoral-system
```

**Paso 2:** Estructura del repositorio

```
sved-electoral-system/
├── docs/
│   ├── 01_srs_requisitos.md
│   ├── 02_manual_tecnico_arquitectura.md
│   ├── 03_manual_instalacion_sysadmin.md
│   └── 04_manual_usuario_final.md
├── backend/
│   ├── sved-core/
│   │   ├── src/
│   │   ├── pom.xml
│   │   └── .env.sved
│   ├── biometrix-baas/
│   │   ├── src/
│   │   ├── pom.xml
│   │   └── .env.biometrix
│   └── python-sidecar/
│       ├── main.py
│       ├── requirements.txt
│       └── .env.python
├── frontend/
│   ├── terminal-votacion/
│   ├── panel-administrativo/
│   └── portal-auditoria/
├── fabric/
│   ├── chaincode/
│   │   └── VotingContract.java
│   ├── crypto-config/
│   └── channel-artifacts/
├── docker/
│   ├── docker-compose.yml
│   └── Dockerfile.sved
└── scripts/
    ├── init-db.sh
    └── deploy-fabric.sh
```

### 3.8 Compilación y Despliegue de Backend

#### 3.8.1 Compilar SVED Core

```bash
cd backend/sved-core
mvn clean package -DskipTests
```

#### 3.8.2 Compilar BiometriX BaaS

```bash
cd backend/biometrix-baas
mvn clean package -DskipTests
```

#### 3.8.3 Configurar Python Sidecar

```bash
cd backend/python-sidecar
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 3.9 Migración de Bases de Datos

**Paso 1:** Ejecutar scripts SQL

```bash
# Para sved_db
psql -U sved_user -d sved_db -f backend/sved-core/src/main/resources/db/migration/V1__init_schema.sql
psql -U sved_user -d sved_db -f backend/sved-core/src/main/resources/db/migration/V2__seed_data.sql

# Para biometrix_db
psql -U biometrix_user -d biometrix_db -f backend/biometrix-baas/src/main/resources/db/migration/V1__init_schema.sql
psql -U biometrix_user -d biometrix_db -f backend/biometrix-baas/src/main/resources/db/migration/V2__seed_tenants.sql
```

### 3.10 Iniciar Servicios

**Paso 1:** Iniciar Python Sidecar (gRPC)

```bash
cd backend/python-sidecar
source venv/bin/activate
python main.py
```

**Paso 2:** Iniciar BiometriX BaaS

```bash
cd backend/biometrix-baas
export $(cat .env.biometrix | xargs)
java -jar target/biometrix-baas-1.0.0.jar
```

**Paso 3:** Iniciar SVED Core

```bash
cd backend/sved-core
export $(cat .env.sved | xargs)
java -jar target/sved-core-1.0.0.jar
```

### 3.11 Despliegue del Frontend

**Paso 1:** Compilar Angular

```bash
cd frontend/terminal-votacion
npm install
ng build --configuration=production

# Los archivos compilados estarán en dist/
```

**Paso 2:** Servir con Nginx (producción)

```bash
sudo apt install -y nginx
sudo cp -r dist/* /var/www/html/sved-terminal/
sudo nano /etc/nginx/sites-available/sved-terminal
```

```nginx
server {
    listen 80;
    server_name terminal.eleccion2025.gob;
    root /var/www/html/sved-terminal;
    index index.html index.htm;
    location / {
        try_files $uri $uri/ /index.html;
    }
    location /api/ {
        proxy_pass https://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/sved-terminal /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

### 3.12 Despliegue de Hyperledger Fabric Chaincode

```bash
cd fabric/chaincode

# Instalar chaincode
peer lifecycle chaincode package voting.tar.gz --path . --lang java --label voting_v1

# Instalar en cada organización
peer lifecycle chaincode install voting.tar.gz

# Aprobar chaincode (TSE)
peer lifecycle chaincode approveformyorg \
  --channelID electoral-channel-2025 \
  --name voting-contract \
  --version 1.0 \
  --package-id voting_v1:hash \
  --sequence 1

# Aprobar chaincode (Fiscalía)
# ... repetir para cada organización

# Commit chaincode
peer lifecycle chaincode commit \
  --channelID electoral-channel-2025 \
  --name voting-contract \
  --version 1.0 \
  --sequence 1 \
  --signature-policy "AND('TSE.member', 'Fiscalia.member')"
```

---

## 4. SOLUCIÓN DE PROBLEMAS (TROUBLESHOOTING)

### 4.1 Error: PostgreSQL no inicia

| Síntoma | Causa | Solución |
|---------|-------|----------|
| `systemctl status postgresql` muestra `failed` | Puerto 5432 ocupado o permisos incorrectos | Verificar puerto libre: `sudo netstat -tulpn \| grep 5432`<br>Verificar logs: `sudo tail -f /var/log/postgresql/postgresql-15-main.log`<br>Reiniciar: `sudo systemctl restart postgresql` |

### 4.2 Error: Conexión a Redis falla

| Síntoma | Causa | Solución |
|---------|-------|----------|
| `Redis connection refused` | Redis no está corriendo o puerto bloqueado | Verificar: `sudo systemctl status redis-server`<br>Iniciar: `sudo systemctl start redis-server`<br>Verificar contraseña en `.env` coincide con `/etc/redis/redis.conf` |

### 4.3 Error: Vault no responde

| Síntoma | Causa | Solución |
|---------|-------|----------|
| `Vault connection error: 503` | Vault no iniciado o token inválido | Iniciar Vault: `vault server -dev -dev-listen-address="0.0.0.0:8200"`<br>Verificar token: `export VAULT_TOKEN=root-token`<br>Verificar ruta: `vault kv get secret/biometrix/keys/tenant-sved` |

### 4.4 Error: gRPC sidecar no responde

| Síntoma | Causa | Solución |
|---------|-------|----------|
| `gRPC connection timeout` | Sidecar no iniciado o puerto bloqueado | Verificar: `ps aux \| grep python`<br>Iniciar: `python main.py`<br>Verificar puerto: `netstat -tulpn \| grep 50051`<br>Revisar logs: `tail -f /var/log/sved/python-sidecar.log` |

### 4.5 Error: Fabric endorsement falla

| Síntoma | Causa | Solución |
|---------|-------|----------|
| `Endorsement policy failure` | Faltan peers o organizaciones | Verificar peers activos: `peer channel list`<br>Verificar policy: `peer lifecycle chaincode querycommitted --channelID electoral-channel-2025 --name voting-contract`<br>Esperar sincronización de peers |

---

## 5. ANEXO: SCRIPTS DE INICIO

### 5.1 Inicio de todos los servicios

```bash
#!/bin/bash
# start-services.sh

echo "Iniciando servicios del sistema electoral..."

# 1. PostgreSQL
sudo systemctl start postgresql
sudo systemctl status postgresql --no-pager

# 2. Redis
sudo systemctl start redis-server
sudo systemctl status redis-server --no-pager

# 3. Vault (desarrollo)
vault server -dev -dev-listen-address="0.0.0.0:8200" &
export VAULT_ADDR="http://localhost:8200"
export VAULT_TOKEN="root-token"

# 4. Python Sidecar
cd /opt/sved/backend/python-sidecar
source venv/bin/activate
nohup python main.py > /var/log/sved/python-sidecar.log 2>&1 &

# 5. BiometriX BaaS
cd /opt/sved/backend/biometrix-baas
export $(cat .env.biometrix | xargs)
nohup java -jar target/biometrix-baas-1.0.0.jar > /var/log/biometrix/biometrix.log 2>&1 &

# 6. SVED Core
cd /opt/sved/backend/sved-core
export $(cat .env.sved | xargs)
nohup java -jar target/sved-core-1.0.0.jar > /var/log/sved/sved.log 2>&1 &

# 7. Nginx (Frontend)
sudo systemctl start nginx

echo "Todos los servicios iniciados."
```

### 5.2 Detener todos los servicios

```bash
#!/bin/bash
# stop-services.sh

echo "Deteniendo servicios del sistema electoral..."

pkill -f "python main.py"
pkill -f "biometrix-baas-1.0.0.jar"
pkill -f "sved-core-1.0.0.jar"
pkill -f "vault server"
sudo systemctl stop nginx
sudo systemctl stop redis-server
sudo systemctl stop postgresql

echo "Todos los servicios detenidos."