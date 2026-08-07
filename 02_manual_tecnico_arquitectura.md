## MANUAL TÉCNICO DEL SISTEMA
### Arquitectura e Implementación Interna
### SVED + BiometriX - Sistema de Votación Electrónica Descentralizado
<a href="https://postimg.cc/7J12TtPw" target="_blank"><img src="https://i.postimg.cc/pr4Y43gh/logo.png" alt="logo"></a>

---

## 1. ARQUITECTURA DEL SISTEMA

### 1.1 Capas de la Arquitectura

El sistema sigue una arquitectura de 5 capas con separación estricta de responsabilidades y datos:

| Capa | Componentes | Tecnología | Responsabilidad |
|------|-------------|------------|-----------------|
| **1. Cliente** | Terminal de Votación, Panel Administrativo, Portal de Auditoría | Angular | Interfaz de usuario para votantes, operadores y auditores |
| **2. Orquestador** | SVED Core | Spring Boot 3 | Lógica de negocio, orquestación de votación, API Gateway |
| **3. Autenticación Biométrica** | BiometriX BaaS | Spring Boot 3 | Autenticación 1:1, liveness detection, almacenamiento de plantillas |
| **4. Motor de IA** | Sidecar Python | Python 3.10+ | Extracción de embeddings faciales (FaceNet/InsightFace) |
| **5. Persistencia** | PostgreSQL, Hyperledger Fabric, Vault/HSM | PostgreSQL 15, Fabric 2.5, Vault 1.13 | Almacenamiento de datos, blockchain inmutable, gestión de claves |

### 1.2 Invariante Fundamental

**LAS DOS BASES DE DATOS NUNCA SE COMUNICAN ENTRE SÍ**

| Regla | Descripción |
|-------|-------------|
| **SVED Core** | No puede leer `biometrix_db` |
| **BiometriX** | No puede leer `sved_db` |
| **Coordinación** | Toda comunicación pasa por la API REST con mTLS + API Key |
| **Motivo** | Aislamiento de identidad (DUI) de datos biométricos (plantillas) |

### 1.3 Flujo de Comunicación (21 Mensajes)

El proceso completo de votación consta de 21 intercambios de mensajes organizados en 3 fases:

| Fase | Mensajes | Descripción |
|------|----------|-------------|
| **Fase 1: Identificación** | 1-2 | El operador ingresa DUI, SVED consulta el padrón localmente |
| **Fase 2: Autenticación Biométrica** | 3-10 | SVED solicita Session Token, captura muestra, BiometriX verifica |
| **Fase 3: Voto Anónimo** | 11-21 | Emisión de JWT anónimo, cifrado ElGamal, registro en Fabric, recibo |

**Secuencia detallada:**

| # | Emisor | Receptor | Mensaje |
|---|--------|----------|---------|
| 1 | Frontend | SVED | DUI del ciudadano |
| 2 | SVED | SVED | Consulta local en sved_db |
| 3 | SVED | BiometriX | Solicitar Session Token |
| 4 | BiometriX | SVED | Session Token (TTL 5 min) |
| 5 | SVED | Frontend | Proceder a captura biométrica |
| 6 | Frontend | SVED | Muestra biométrica (Base64) |
| 7 | SVED | BiometriX | POST /api/v1/verify |
| 8 | BiometriX | BiometriX | Liveness Detection + Matching |
| 9 | BiometriX | SVED | Decisión (match/no match) |
| 10 | SVED | Frontend | Resultado autenticación |
| 11 | SVED | SVED | Emitir JWT de Voto (anónimo) |
| 12 | SVED | Frontend | JWT de Voto |
| 13 | Frontend | Frontend | Cifrado ElGamal + Blind Signature |
| 14 | Frontend | SVED | Voto cifrado + ZKP |
| 15 | SVED | SVED | Validar JWT + consumir JTI |
| 16 | SVED | Fabric | Enviar transacción al chaincode |
| 17 | Fabric | Fabric | Endorsement (2 de 3 organizaciones) |
| 18 | Fabric | SVED | Transaction ID |
| 19 | SVED | SVED | Actualizar estado a VOTADO |
| 20 | SVED | Frontend | Confirmación + Transaction ID |
| 21 | Frontend | Ciudadano | Recibo impreso o digital |

---

## 2. MODELO DE DATOS

### 2.1 biometrix_db (Tablas)

#### Tabla: tenants

| Campo | Tipo | Nulable | PK/FK | Descripción |
|-------|------|---------|-------|-------------|
| id | BIGSERIAL | NO | PK | Identificador único del tenant |
| tenant_name | VARCHAR(100) | NO | - | Nombre del cliente/tenant |
| api_key_hash | VARCHAR(64) | NO | - | SHA-256 de la API Key |
| encryption_key_ref | VARCHAR(255) | NO | - | Referencia a la clave en Vault |
| match_threshold | DECIMAL(3,2) | NO | - | Umbral de coincidencia (0.85 por defecto) |
| rate_limit | INTEGER | NO | - | Límite de peticiones por minuto |
| created_at | TIMESTAMP | NO | - | Fecha de creación |

#### Tabla: templates

| Campo | Tipo | Nulable | PK/FK | Descripción |
|-------|------|---------|-------|-------------|
| id | BIGSERIAL | NO | PK | Identificador único |
| tenant_id | BIGINT | NO | FK (tenants.id) | Referencia al tenant |
| external_id | VARCHAR(36) | NO | - | UUID del ciudadano |
| modality | VARCHAR(20) | NO | - | fingerprint / facial / voice |
| encrypted_template | BYTEA | NO | - | IV + Cyphertext + GCM Tag |
| quality_score | DECIMAL(3,2) | NO | - | 0.0 - 1.0 |
| created_at | TIMESTAMP | NO | - | Fecha de creación |
| updated_at | TIMESTAMP | NO | - | Fecha de actualización |

**Restricción UNIQUE:** (tenant_id, external_id, modality)

#### Tabla: sessions

| Campo | Tipo | Nulable | PK/FK | Descripción |
|-------|------|---------|-------|-------------|
| id | BIGSERIAL | NO | PK | Identificador único |
| tenant_id | BIGINT | NO | FK (tenants.id) | Referencia al tenant |
| token_hash | VARCHAR(64) | NO | - | SHA-256 del session token |
| external_id | VARCHAR(36) | NO | - | UUID del ciudadano |
| is_consumed | BOOLEAN | NO | - | false = disponible, true = usado |
| expires_at | TIMESTAMP | NO | - | TTL 5 minutos |
| created_at | TIMESTAMP | NO | - | Fecha de creación |

#### Tabla: audit_log

| Campo | Tipo | Nulable | PK/FK | Descripción |
|-------|------|---------|-------|-------------|
| id | BIGSERIAL | NO | PK | Identificador único |
| tenant_id | BIGINT | NO | FK (tenants.id) | Referencia al tenant |
| external_id_hash | VARCHAR(64) | NO | - | SHA-256(external_id) |
| operation | VARCHAR(20) | NO | - | ENROLL / VERIFY |
| result | VARCHAR(20) | NO | - | SUCCESS / FAILED |
| quality_score | DECIMAL(3,2) | SI | - | Score de calidad |
| ip_address | VARCHAR(45) | SI | - | IPv4 o IPv6 |
| created_at | TIMESTAMP | NO | - | Fecha de creación |

**Restricción:** Tabla INSERT-only (trigger previene UPDATE y DELETE)

#### Tabla: lockouts

| Campo | Tipo | Nulable | PK/FK | Descripción |
|-------|------|---------|-------|-------------|
| id | BIGSERIAL | NO | PK | Identificador único |
| tenant_id | BIGINT | NO | FK (tenants.id) | Referencia al tenant |
| external_id | VARCHAR(36) | NO | - | UUID del ciudadano |
| failed_attempts | INTEGER | NO | - | Conteo actual de fallos |
| locked_until | TIMESTAMP | SI | - | NULL si no está bloqueado |
| total_lockouts_24h | INTEGER | NO | - | Número de bloqueos en 24h |
| created_at | TIMESTAMP | NO | - | Fecha de creación |
| updated_at | TIMESTAMP | NO | - | Fecha de actualización |

### 2.2 sved_db (Tablas)

#### Tabla: centros_votacion

| Campo | Tipo | Nulable | PK/FK | Descripción |
|-------|------|---------|-------|-------------|
| id | BIGSERIAL | NO | PK | Identificador único |
| nombre | VARCHAR(200) | NO | - | Nombre del centro |
| direccion | VARCHAR(300) | NO | - | Dirección física |
| departamento | VARCHAR(50) | NO | - | Departamento |
| municipio | VARCHAR(50) | NO | - | Municipio |
| capacidad | INTEGER | NO | - | Capacidad máxima |
| created_at | TIMESTAMP | NO | - | Fecha de creación |

#### Tabla: ciudadanos

| Campo | Tipo | Nulable | PK/FK | Descripción |
|-------|------|---------|-------|-------------|
| id | BIGSERIAL | NO | PK | Identificador único |
| centro_id | BIGINT | NO | FK (centros_votacion.id) | Centro asignado |
| dui | VARCHAR(10) | NO | - | Documento Único de Identidad |
| nombres | VARCHAR(100) | NO | - | Nombres del ciudadano |
| apellidos | VARCHAR(100) | NO | - | Apellidos del ciudadano |
| biometrix_external_id | VARCHAR(36) | NO | - | UUID para BiometriX |
| estado_voto | VARCHAR(20) | NO | - | Ver lista de estados |
| failed_attempts | INTEGER | NO | - | Intentos fallidos consecutivos |
| lockout_until | TIMESTAMP | SI | - | NULL si no bloqueado |
| created_at | TIMESTAMP | NO | - | Fecha de creación |
| updated_at | TIMESTAMP | NO | - | Fecha de actualización |

#### Tabla: elecciones

| Campo | Tipo | Nulable | PK/FK | Descripción |
|-------|------|---------|-------|-------------|
| id | BIGSERIAL | NO | PK | Identificador único |
| election_id | VARCHAR(50) | NO | - | ID único de la elección |
| nombre | VARCHAR(200) | NO | - | Nombre de la elección |
| fecha_apertura | TIMESTAMP | NO | - | Fecha y hora de apertura |
| fecha_cierre | TIMESTAMP | NO | - | Fecha y hora de cierre |
| estado | VARCHAR(20) | NO | - | CONFIGURANDO / ABIERTA / CERRADA |
| candidatos | JSONB | NO | - | Lista de candidatos [{id, nombre, partido}] |
| clave_publica_elgamal | TEXT | NO | - | Clave pública Y |
| clave_privada_shamir | TEXT | NO | - | Partes Shamir (5 partes) |
| created_at | TIMESTAMP | NO | - | Fecha de creación |

#### Tabla: tokens_consumidos

| Campo | Tipo | Nulable | PK/FK | Descripción |
|-------|------|---------|-------|-------------|
| id | BIGSERIAL | NO | PK | Identificador único |
| jti | VARCHAR(36) | NO | - | JWT ID único |
| election_id | VARCHAR(50) | NO | FK (elecciones.election_id) | ID de la elección |
| consumed_at | TIMESTAMP | NO | - | Fecha de consumo |

### 2.3 Estados del Ciudadano

| Estado | Descripción | Transición |
|--------|-------------|------------|
| **SIN_REGISTRO** | Ciudadano en padrón sin plantilla biométrica | Enrollment exitoso → INSCRITO |
| **INSCRITO** | Plantilla biométrica almacenada | Inicio jornada → PENDIENTE |
| **PENDIENTE** | Habilitado para votar | Autenticación exitosa → EN_PROCESO |
| **EN_PROCESO** | JWT emitido, voto en curso | Voto registrado → VOTADO |
| **VOTADO** | Voto registrado en Fabric (final) | Estado terminal, no transiciona |
| **BLOQUEADO_TEMP** | Lockout por 3 fallos biométricos | 15 min o desbloqueo manual → PENDIENTE |
| **TOKEN_EXPIRADO** | JWT expiró (5 min) | Reautenticación autorizada → EN_PROCESO |
| **INHABILITADO** | Bloqueo administrativo | Rehabilitación manual → PENDIENTE |

---

## 3. ESPECIFICACIÓN DE APIs

### 3.1 BiometriX BaaS - API REST

#### POST /api/v1/enroll - Registro Biométrico

| Aspecto | Detalle |
|---------|---------|
| **Método** | POST |
| **Endpoint** | `/api/v1/enroll` |
| **Headers** | `X-API-Key: {api_key_del_tenant}`<br>`Content-Type: application/json` |

**Request Body:**
```json
{
  "external_id": "550e8400-e29b-41d4-a716-446655440000",
  "modality": "fingerprint",
  "sample": "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
}
```

**Response (201 Created):**
```json
{
  "status": "enrolled",
  "enrollment_id": "enr_a7f3d-8b2c-4d5e-9f1a-2b3c4d5e6f7g",
  "external_id": "550e8400-e29b-41d4-a716-446655440000",
  "modality": "fingerprint",
  "quality_score": 0.94,
  "enrolled_at": "2025-07-01T09:30:00Z"
}
```

**Códigos de Error:**

| Código HTTP | Código Interno | Descripción |
|-------------|----------------|-------------|
| 400 | - | Payload malformado o faltan campos obligatorios |
| 401 | BX-001 | API Key inválida o expirada |
| 409 | BX-003 | Ciudadano ya registrado para esta modalidad |
| 422 | BX-004 | Liveness Failed (muestra no viva, score < 0.90) |
| 422 | BX-005 | Quality Insufficient (muestra de baja calidad) |
| 429 | BX-002 | Rate limit excedido |

---

#### POST /api/v1/session - Solicitar Session Token

| Aspecto | Detalle |
|---------|---------|
| **Método** | POST |
| **Endpoint** | `/api/v1/session` |
| **Headers** | `X-API-Key: {api_key_del_tenant}` |

**Request Body:**
```json
{
  "external_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response (200 OK):**
```json
{
  "session_token": "st_7h8g9f-3k4l-5m6n-7o8p-9q0r1s2t3u4v",
  "expires_at": "2025-11-02T08:20:00Z"
}
```

---

#### POST /api/v1/verify - Verificación Biométrica 1:1

| Aspecto | Detalle |
|---------|---------|
| **Método** | POST |
| **Endpoint** | `/api/v1/verify` |
| **Headers** | `X-API-Key: {api_key_del_tenant}`<br>`Content-Type: application/json` |

**Request Body:**
```json
{
  "external_id": "550e8400-e29b-41d4-a716-446655440000",
  "session_token": "st_7h8g9f-3k4l-5m6n-7o8p-9q0r1s2t3u4v",
  "sample": "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
}
```

**Response - Match (200 OK):**
```json
{
  "status": "success",
  "decision": "match",
  "confidence_score": 0.97,
  "liveness_score": 0.95,
  "transaction_id": "auth_8823xk-9n8m-7l6k-5j4h-3g2f1e0d9c8b",
  "timestamp": "2025-11-02T08:14:30Z"
}
```

**Response - Liveness Failed (422):**
```json
{
  "status": "failure",
  "error_code": "LIVENESS_FAILED",
  "liveness_score": 0.43,
  "attempts_remaining": 2
}
```

**Response - Match Failed (422):**
```json
{
  "status": "failure",
  "error_code": "MATCH_FAILED",
  "confidence_score": 0.52,
  "attempts_remaining": 2
}
```

**Response - Lockout Active (423):**
```json
{
  "status": "failure",
  "error_code": "LOCKOUT_ACTIVE",
  "locked_until": "2025-11-02T08:30:00Z",
  "minutes_remaining": 15
}
```

### 3.2 SVED Core - API Internas

#### POST /api/v1/auth/identify - Identificación por DUI

| Aspecto | Detalle |
|---------|---------|
| **Método** | POST |
| **Endpoint** | `/api/v1/auth/identify` |

**Request Body:**
```json
{
  "dui": "12345678-9",
  "centro_id": 101
}
```

**Response (200 OK):**
```json
{
  "status": "verified",
  "ciudadano": {
    "id": 1001,
    "nombres": "Juan Carlos",
    "apellidos": "Pérez Gómez",
    "estado": "PENDIENTE",
    "external_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

#### POST /api/v1/vote/emit - Emitir Voto

| Aspecto | Detalle |
|---------|---------|
| **Método** | POST |
| **Endpoint** | `/api/v1/vote/emit` |

**Request Body:**
```json
{
  "jwt_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "encrypted_vote": {
    "c1": "base64_encoded_point",
    "c2": "base64_encoded_point"
  },
  "blind_signature": "base64_encoded_signature",
  "zk_proof": "base64_encoded_proof"
}
```

**Response (200 OK):**
```json
{
  "status": "registered",
  "transaction_id": "7f8g9h0i1j2k3l4m5n6o7p8q9r0s1t2u3v4w5x6y7z8a9b0c",
  "block_number": 1042,
  "timestamp": "2025-11-02T08:15:45Z"
}
```

---

## 4. MANEJO DE EXCEPCIONES

### 4.1 Catálogo de Códigos de Error

| Código | HTTP | Descripción Técnica | Causa |
|--------|------|---------------------|-------|
| **SVED-001** | 400 | INVALID_DUI_FORMAT | El DUI no cumple formato (#########-#) |
| **SVED-002** | 404 | CITIZEN_NOT_FOUND | DUI no encontrado en el padrón electoral |
| **SVED-003** | 403 | WRONG_VOTING_CENTER | Ciudadano no asignado a este centro |
| **SVED-004** | 409 | ALREADY_VOTED | Ciudadano ya votó (estado VOTADO) |
| **SVED-005** | 423 | TEMPORARY_LOCKOUT | Ciudadano bloqueado por 15 minutos |
| **SVED-006** | 401 | INVALID_JWT | JWT de voto inválido o expirado |
| **SVED-007** | 409 | JTI_ALREADY_CONSUMED | JWT ID ya utilizado (doble voto) |
| **BX-001** | 401 | INVALID_API_KEY | API Key no existe, inactiva o revocada |
| **BX-002** | 429 | RATE_LIMIT_EXCEEDED | Límite de peticiones por minuto superado |
| **BX-003** | 409 | ALREADY_ENROLLED | Ciudadano ya tiene plantilla para esta modalidad |
| **BX-004** | 422 | LIVENESS_FAILED | Muestra no superó detección de vida (score < 0.90) |
| **BX-005** | 422 | QUALITY_INSUFFICIENT | Muestra no cumple estándares de calidad técnica |
| **BX-006** | 403 | SESSION_TOKEN_INVALID | Session Token no existe, expirado o consumido |
| **BX-007** | 423 | LOCKOUT_ACTIVE | Ciudadano en lockout por múltiples fallos |
| **FAB-001** | 500 | CHAINCODE_EXECUTION_ERROR | Error en ejecución del chaincode de Fabric |
| **FAB-002** | 500 | ENDORSEMENT_POLICY_FAILED | No se obtuvo endorsement mínimo (2/3) |
| **FAB-003** | 422 | INVALID_ZK_PROOF | Prueba de conocimiento cero inválida |
| **FAB-004** | 422 | INVALID_BLIND_SIGNATURE | Firma ciega no corresponde a la autoridad |
| **FAB-005** | 400 | ELECTION_NOT_OPEN | Elección no está en estado ABIERTA |

### 4.2 Ejemplo de Respuesta de Error (BiometriX)

```json
{
  "timestamp": "2025-11-02T08:14:30Z",
  "status": 422,
  "error": "Unprocessable Entity",
  "code": "BX-004",
  "message": "Liveness detection failed: sample does not meet minimum threshold of 0.90",
  "details": {
    "liveness_score": 0.43,
    "threshold": 0.90,
    "attempts_remaining": 2
  },
  "path": "/api/v1/verify"
}
```

### 4.3 Circuit Breaker (Resilience4j)

BiometriX implementa un Circuit Breaker para el microservicio Python (gRPC):

| Estado | Descripción | Comportamiento |
|--------|-------------|----------------|
| **CLOSED** | Operación normal | Se permiten todas las llamadas al microservicio Python |
| **OPEN** | Circuito abierto | Después de 5 fallos consecutivos, se retorna error graceful sin llamar a Python |
| **HALF-OPEN** | Prueba | Después de 30 segundos, se permite una llamada de prueba. Si es exitosa → CLOSED, si falla → OPEN |

---

## 5. SEGURIDAD Y DEFENSA EN PROFUNDIDAD

### 5.1 Capas de Seguridad (7 Capas)

| Capa | Tecnología | Descripción |
|------|------------|-------------|
| **1** | TLS 1.3 | Handshake 1-RTT. TLS 1.0/1.1/1.2 deshabilitados. Certificados PKI o Let's Encrypt |
| **2** | mTLS + API Key | Cliente y servidor presentan certificados. API Key hasheada (SHA-256) en BD. Tenant resuelto desde certificado |
| **3** | Anti-Replay Session Token | Token de un solo uso (TTL 5 min). JTI consumido en Redis. Timestamps verificados |
| **4** | Liveness Detection (ISO/IEC 30107-3) | Huella: conductividad + presión + elasticidad. Rostro: texturas + profundidad + microexpresiones. Score mínimo: 0.90 |
| **5** | Cifrado en Reposo AES-256-GCM | Salt único por tenant. Clave maestra en Vault/HSM, nunca en BD. IV único por operación |
| **6** | BioHashing (Cancelable Biometrics) | Transformación matemática irreversible. Plantillas cancelables / revocables. Si se filtra, se invalida y regenera |
| **7** | Auditoría Inmutable INSERT-only | Tabla de solo inserción (sin UPDATE/DELETE). External_ID hasheado (jamás en plaintext). Timestamps + IP + score + decisión. Capacidad forense completa |

### 5.2 SecurityFilterChain - Spring Security

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    return http
        .requiresChannel(c -> c.anyRequest().requiresSecure()) // TLS 1.3
        .addFilterBefore(apiKeyFilter, UsernamePasswordAuthenticationFilter.class)
        .addFilterBefore(mTLSFilter, ApiKeyFilter.class)
        .addFilterBefore(lockoutFilter, ApiKeyFilter.class)
        .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .build();
}
```

### 5.3 Trigger PostgreSQL - Audit Log INSERT-only

```sql
-- biometrix_db - Integridad de la tabla audit_log
-- Trigger PostgreSQL: audit_log es INSERT ONLY

CREATE OR REPLACE FUNCTION prevent_audit_modification() 
RETURNS TRIGGER AS $$ 
BEGIN 
    IF TG_OP IN ('UPDATE', 'DELETE') THEN 
        RAISE EXCEPTION 'La tabla audit_log es de solo inserción';
    END IF; 
    RETURN NEW; 
END; 
$$ LANGUAGE plpgsql; 

CREATE TRIGGER audit_insert_only 
BEFORE UPDATE OR DELETE ON audit_log 
FOR EACH ROW EXECUTE FUNCTION prevent_audit_modification();
```

---

## 6. DEPENDENCIAS TECNOLÓGICAS

| Componente | Tecnología | Versión |
|------------|------------|---------|
| **Lenguaje** | Java | 21 |
| **Framework Backend** | Spring Boot | 3.x |
| **Framework Frontend** | Angular | 15+ |
| **Base de Datos 1** | PostgreSQL | 15.x |
| **Base de Datos 2** | PostgreSQL | 15.x |
| **Blockchain** | Hyperledger Fabric | 2.5.x |
| **Gestión de Secretos** | HashiCorp Vault | 1.13+ |
| **Motor de IA** | Python | 3.10+ |
| **Biblioteca IA Facial** | FaceNet / InsightFace | Última estable |
| **Criptografía** | Bouncy Castle | 1.77+ |
| **Rate Limiting** | Bucket4j | 8.x |
| **Circuit Breaker** | Resilience4j | 2.x |
| **Cache / Tokens** | Redis | 7.x |

---

## 7. FLUJOS INTERNOS

### 7.1 Flujo de Enrollment (Registro Biométrico)

| Paso | Acción | Fallo | Éxito |
|------|--------|-------|-------|
| 1 | POST /api/v1/enroll | - | Continúa |
| 2 | Validar API Key | 401 Invalid API Key | Continúa |
| 3 | ¿Ya registrado? | 409 Conflict | Continúa |
| 4 | Seleccionar procesador (Strategy) | - | Continúa |
| 5 | Liveness Detection | 422 Liveness Failed | Continúa |
| 6 | Control de Calidad | 422 Quality Insufficient | Continúa |
| 7 | Extraer Embedding (Python sidecar) | Error gRPC | Continúa |
| 8 | Cifrar AES-256-GCM (Vault) | - | Continúa |
| 9 | Almacenar en PostgreSQL | - | Continúa |
| 10 | Audit Log (INSERT-only) | - | Continúa |
| 11 | Retornar 201 Created | - | **FIN** |

### 7.2 Flujo de Verificación Biométrica (1:1)

| Paso | Acción | Fallo | Éxito |
|------|--------|-------|-------|
| 1 | POST /api/v1/verify | - | Continúa |
| 2 | Validar Session Token | 403 Invalid Token | Continúa |
| 3 | Consumir Token (marcar usado) | - | Continúa |
| 4 | ¿Lockout Activo? | 423 Locked | Continúa |
| 5 | Ejecutar en Paralelo: Liveness + Recuperar Plantilla | - | Continúa |
| 6 | ¿Liveness OK? (score ≥ 0.90) | 422 Liveness Failed | Continúa |
| 7 | Matching Engine (Distancia Coseno) | - | Continúa |
| 8 | ¿Score ≥ Threshold? | 422 Match Failed | Continúa |
| 9 | Resetear failed_attempts | - | Continúa |
| 10 | Retornar 200 OK (Match) | - | **FIN** |

**Manejo de Match Failed:**
| Intentos Fallidos | Acción |
|-------------------|--------|
| 1-2 | Retornar attempts_remaining, ciudadano reintenta |
| ≥ 3 | Activar Lockout 15 min, retornar 423 Locked |

---

## 8. CRIPTOGRAFÍA APLICADA

### 8.1 Cifrado ElGamal Homomórfico

| Elemento | Descripción |
|----------|-------------|
| **Algoritmo** | ElGamal sobre curvas elípticas (EC-ElGamal) |
| **Propiedad** | Suma de cifrados = Cifrado de la suma |
| **Uso** | Cifrado de votos individuales y recuento homomórfico |
| **Parámetros** | G = punto generador, Y = clave pública (Y = xG) |
| **Cifrado** | C1 = rG, C2 = v·G + r·Y |
| **Descifrado (total)** | total·G = C2_total - x·C1_total |

### 8.2 Blind Signature (Firma Ciega de Chaum)

| Elemento | Descripción |
|----------|-------------|
| **Propósito** | Autoridad firma el voto sin ver el contenido |
| **Beneficio** | Previene coerción: ciudadano no puede probar a nadie cómo votó |
| **Mecanismo** | Ciudadano enmascara el voto → Autoridad firma → Ciudadano desenmascara |

### 8.3 Zero-Knowledge Proof (Schnorr)

| Elemento | Descripción |
|----------|-------------|
| **Propósito** | Demostrar que el voto contiene una opción válida sin revelar cuál |
| **Algoritmo** | Protocolo Schnorr |
| **Tamaño** | ~256 bytes |
| **Tiempo de Verificación** | < 10 ms |

### 8.4 Distribución de Clave Privada (Shamir 3-de-5)

| Elemento | Descripción |
|----------|-------------|
| **Propósito** | Evitar que una sola persona pueda descifrar los votos |
| **Esquema** | Shamir Secret Sharing |
| **Configuración** | 5 partes generadas, se necesitan 3 para reconstruir |
| **Custodios** | TSE, Fiscalía, Defensoría, Partidos Políticos, Observadores |
