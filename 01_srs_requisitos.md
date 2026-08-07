
## SRS - Especificación de Requisitos de Software
<a href="https://postimg.cc/7J12TtPw" target="_blank"><img src="https://i.postimg.cc/pr4Y43gh/logo.png" alt="logo"></a>
### Sistema de Votación Electrónica Descentralizado (SVED + BiometriX)

#### 1. Introducción

**1.1 Propósito**
El presente documento detalla los requisitos funcionales y no funcionales del **Sistema de Votación Electrónica Descentralizado (SVED)**. Su objetivo principal es servir como la fuente de verdad entre el equipo de desarrollo y las partes interesadas, garantizando que el producto final cumpla con los estándares de seguridad, anonimato e inmutabilidad requeridos para un proceso electoral.

**1.2 Alcance**
El sistema se compone de una arquitectura de 5 capas que incluye:
- **Capa Cliente (Frontend):** Angular para Terminales de Votación, Paneles Administrativos y Portal de Auditoría.
- **Núcleo (SVED Core):** Spring Boot 3, orquestador central de procesos, APIs y lógica de negocio.
- **Servicio Biométrico (BiometriX BaaS):** Servicio independiente para autenticación 1:1, aislamiento de plantillas y detección de vida (Liveness).
- **Motor de IA (Sidecar Python):** Extracción de embeddings vectoriales (FaceNet/InsightFace) en red interna.
- **Persistencia:** PostgreSQL (bases de datos aisladas), Hyperledger Fabric (Ledger inmutable) y HashiCorp Vault (Gestión de claves).

#### 2. Requisitos Funcionales (RF)

**Módulo 1: Autenticación y Gestión de Identidades (BiometriX)**

| ID | Requisito | Prioridad |
| :--- | :--- | :--- |
| **RF-01** | **Enrollment Biométrico:** El sistema debe permitir el registro de plantillas biométricas (huella, rostro, voz) asociadas a un `External_ID` (UUID), con cifrado AES-256-GCM y almacenamiento en `biometrix_db` [citation:1]. | Alta |
| **RF-02** | **Verificación 1:1:** El sistema debe comparar una muestra en vivo contra la plantilla de referencia de un ciudadano específico, calculando la distancia coseno y aplicando un threshold configurable por tenant (ej. 0.85) [citation:2]. | Alta |
| **RF-03** | **Detección de Vida (Liveness):** Previo a la comparación, el sistema debe validar que la muestra proviene de una persona viva (ISO/IEC 30107-3), analizando conductividad, textura o profundidad según la modalidad [citation:3]. | Alta |
| **RF-04** | **Anti-Replay:** El sistema debe implementar un mecanismo de Session Token (TTL 5 min) para prevenir ataques de repetición en las solicitudes de verificación [citation:4]. | Media |

**Módulo 2: Proceso de Votación (SVED Core)**

| ID | Requisito | Prioridad |
| :--- | :--- | :--- |
| **RF-05** | **Verificación de Padrón:** El sistema debe consultar `sved_db` para validar que el DUI exista, esté asignado al centro actual y tenga estado `PENDIENTE` antes de iniciar el flujo de votación [citation:5]. | Alta |
| **RF-06** | **Separación Identidad-Voto:** Tras la autenticación exitosa, el sistema debe emitir un JWT de Voto que excluya explícitamente datos personales (DUI, nombre), conteniendo solo `jti`, `election_id` y `exp` [citation:6]. | Crítica |
| **RF-07** | **Selección y Cifrado:** El Frontend debe cifrar la selección del candidato usando ElGamal Homomórfico y generar una Blind Signature, garantizando que la autoridad no vea el contenido del voto [citation:7]. | Alta |
| **RF-08** | **Registro en Blockchain:** SVED debe enviar la transacción (voto cifrado + ZKP) al chaincode en Hyperledger Fabric, exigiendo endorsements de al menos 2 de 3 organizaciones (TSE, Fiscalía, Observadores) [citation:8]. | Crítica |

**Módulo 3: Criptografía y Seguridad**

| ID | Requisito | Prioridad |
| :--- | :--- | :--- |
| **RF-09** | **Cifrado Homomórfico:** El sistema debe soportar el cifrado ElGamal sobre curvas elípticas para permitir el recuento de votos sin descifrar votos individuales [citation:9]. | Alta |
| **RF-10** | **Firma Ciega:** El sistema debe implementar el protocolo de Chaum para que la autoridad certifique la validez del voto sin conocer el candidato, previniendo la coerción [citation:10]. | Alta |
| **RF-11** | **ZKP (Zero-Knowledge Proof):** El Frontend debe generar una prueba de rango Schnorr (~256 bytes) para demostrar que el voto contiene una opción válida (sin revelar cuál), verificable en <10ms por el chaincode [citation:11]. | Alta |

**Módulo 4: Administración y Auditoría**

| ID | Requisito | Prioridad |
| :--- | :--- | :--- |
| **RF-12** | **Gestión de Claves:** Las claves AES (BiometriX) y las privadas ElGamal (Voto) deben almacenarse en HashiCorp Vault/HSM, con distribución Shamir 3-de-5 para la clave de descifrado de votos [citation:12]. | Crítica |
| **RF-13** | **Auditoría Inmutable:** Las tablas `audit_log` en ambas bases de datos deben ser de solo inserción (INSERT-only), registrando eventos con `External_ID` hasheado para preservar la privacidad [citation:13]. | Alta |
| **RF-14** | **Portal de Verificación:** El sistema debe proveer un Portal Público que consulte el Transaction ID en Hyperledger Fabric y confirme la existencia del voto sin revelar el candidato [citation:14]. | Media |

#### 3. Requisitos No Funcionales (RNF)

| ID | Requisito | Especificación / Meta |
| :--- | :--- | :--- |
| **RNF-01** | **Rendimiento (Latencia):** | El proceso completo de votación (autenticación + registro en Fabric) debe ejecutarse en **< 30 segundos**. La verificación biométrica debe consumir **< 800ms** [citation:15]. |
| **RNF-02** | **Seguridad (Comunicación):** | Toda comunicación entre componentes debe estar cifrada mediante **TLS 1.3** y autenticada por **mTLS** + API Key, con rate limiting por tenant [citation:16]. |
| **RNF-03** | **Disponibilidad:** | El sistema debe soportar la carga de la jornada electoral con un tiempo de actividad esperado del **99.9%**. El mecanismo de lockout (15 min) protege contra ataques de fuerza bruta [citation:17]. |
| **RNF-04** | **Mantenibilidad (Docs as Code):** | Toda la documentación técnica y de usuario debe mantenerse en Markdown (.md) dentro del repositorio `docs/`, siguiendo el estándar de nomenclatura [citation:18]. |
| **RNF-05** | **Portabilidad:** | Los servicios deben ser desplegables en contenedores, con variables de entorno claramente definidas en un archivo `.env.example` [citation:19]. |

#### 4. Restricciones Técnicas y Supuestos

1.  **Tecnología Base:** El backend se desarrolla sobre Java 21 + Spring Boot 3. La capa de persistencia usa PostgreSQL y Hyperledger Fabric 2.x.
2.  **Aislamiento de Datos:** Las bases de datos `sved_db` y `biometrix_db` **nunca deben comunicarse directamente**. La coordinación siempre se realiza a través de la API REST [citation:20].
3.  **Dependencias Externas:** El sistema depende del servicio `BiometriX` (desplegado como tenant independiente), del Microservicio Python para embeddings y de la red Hyperledger Fabric (con al menos 3 organizaciones).