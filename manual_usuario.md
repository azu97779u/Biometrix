# MANUAL DE USUARIO FINAL
## Sistema de Votación Electrónica Descentralizado (SVED + BiometriX)

<a href="https://postimg.cc/7J12TtPw" target="_blank"><img src="https://i.postimg.cc/pr4Y43gh/logo.png" alt="logo"></a>

---

## GUÍA DE INICIO RÁPIDO (QUICKSTART)

Siga estos 3 pasos para comenzar a usar el sistema el día de la votación:

### Paso 1: Acceder al sistema
- Diríjase al centro de votación que le corresponde.
- El operador de mesa iniciará la sesión en la Terminal de Votación.
- **No necesita** usuario ni contraseña. Su identidad será verificada con su DUI y su biometría.

### Paso 2: Identificarse y autenticarse
- Entregue su **DUI físico** al operador. Él lo ingresará en el sistema.
- Cuando la pantalla lo indique, coloque su dedo en el sensor (o mire a la cámara).
- Espere la confirmación de identidad (tarda solo unos segundos).

### Paso 3: Votar y obtener su comprobante
- Seleccione a su candidato en la pantalla táctil.
- Confirme su selección.
- **Guarde el recibo** con el código Transaction ID que aparecerá en pantalla.

**¡Listo! Su voto ha sido emitido de forma segura, anónima e inmutable.**

---

## ÍNDICE GENERAL

1.  **INTRODUCCIÓN AL SISTEMA**
    1.1 Propósito del Manual
    1.2 Principios de Seguridad Fundamentales

2.  **PERFILES DE USUARIO Y SUS RESPONSABILIDADES**
    2.1 Ciudadano Votante
    2.2 Operador de Mesa Electoral (Supervisor)
    2.3 Administrador Electoral
    2.4 Auditor Ciudadano

3.  **MANUAL DEL CIUDADANO — PROCESO COMPLETO DE VOTACIÓN**
    3.1 Llegada al Centro de Votación
    3.2 Paso 1: Identificación Visual por DUI
    3.3 Paso 2: Verificación en el Padrón Electoral
    3.4 Paso 3: Solicitud de Sesión Biométrica
    3.5 Paso 4: Captura de la Muestra Biométrica
    3.6 Paso 5: Procesamiento Interno de Verificación
    3.7 Paso 6: Decisión de Autenticación y Manejo de Fallos
    3.8 Paso 7: Emisión del JWT de Voto
    3.9 Paso 8: Selección del Candidato
    3.10 Paso 9: Cifrado del Voto y Firma Ciega
    3.11 Paso 10: Prueba de Conocimiento Cero (ZKP)
    3.12 Paso 11: Registro en Hyperledger Fabric
    3.13 Paso 12: Entrega del Recibo y Transaction ID
    3.14 Casos Especiales: Token Expirado y Bloqueo Temporal

4.  **MANUAL DEL OPERADOR DE MESA (SUPERVISOR)**
    4.1 Panel de Control del Supervisor
    4.2 Gestión de Identificación del Ciudadano
    4.3 Manejo de Bloqueos Temporales (Lockout)
    4.4 Manejo de Tokens de Voto Expirados
    4.5 Alerta de Fraude o Incidente
    4.6 Cierre de la Jornada Electoral

5.  **MANUAL DEL ADMINISTRADOR ELECTORAL**
    5.1 Preparación del Padrón Electoral
    5.2 Proceso de Inscripción Biométrica (Enrollment)
    5.3 Configuración de Elecciones y Claves Criptográficas
    5.4 Despliegue del Chaincode en Fabric
    5.5 Gestión de Claves en Vault
    5.6 Apertura Oficial de la Elección

6.  **PORTAL DE AUDITORÍA PÚBLICA**
    6.1 Acceso al Portal
    6.2 Verificación por Transaction ID
    6.3 Interpretación del Resultado

7.  **PREGUNTAS FRECUENTES (FAQ)**
    7.1 ¿Qué hago si olvidé mi DUI?
    7.2 ¿Qué pasa si la huella no es reconocida?
    7.3 ¿Puedo votar en otro centro?
    7.4 ¿Cómo sé que mi voto fue registrado?
    7.5 ¿Qué hago si el sistema se apaga?
    7.6 ¿Puedo cambiar mi voto después de confirmarlo?
    7.7 ¿El sistema sabe por quién voté?
    7.8 ¿Qué hago si se me venció el tiempo para votar?

8.  **GLOSARIO DE TÉRMINOS**

---

## 1. INTRODUCCIÓN AL SISTEMA

### 1.1 Propósito del Manual

Este manual le guiará en el uso del **Sistema de Votación Electrónica Descentralizado (SVED)**. Está diseñado para que cualquier persona, sin importar su conocimiento técnico, pueda votar de forma segura y sencilla.

**Garantías del sistema:**
- **Secreto:** Nadie sabrá por quién votó.
- **Inmutable:** Su voto no se puede alterar ni borrar.
- **Verificable:** Usted puede comprobar que su voto fue contado.
- **Seguro:** Su identidad está protegida en todo momento.

### 1.2 Principios de Seguridad Fundamentales

El sistema ha sido diseñado con varias capas de protección:

| Principio | ¿Qué significa para usted? |
|-----------|----------------------------|
| **Separación de datos** | Su información personal (DUI) está almacenada en un lugar diferente a sus datos biométricos (huella/rostro). |
| **Anonimato** | En el momento de votar, el sistema ya no sabe quién es usted. |
| **Cifrado de datos** | Su huella o rostro se almacenan como números cifrados que nadie puede leer. |
| **Detección de vida** | El sistema verifica que usted sea una persona real, no una foto o video. |

---

## 2. PERFILES DE USUARIO Y SUS RESPONSABILIDADES

<a href='https://postimg.cc/47PZ0j56' target='_blank'><img src='https://i.postimg.cc/VvQ6J80V/inicio.jpg' border='0' alt='Pantalla de inicio del sistema'></a>
*Figura 1: Pantalla de inicio del sistema. El operador de mesa inicia sesión aquí.*

### 2.1 Ciudadano Votante

- **Responsabilidad:** Ejercer su derecho al voto de forma libre y secreta.
- **Interacción:** Con la Terminal de Votación.
- **Acciones:** Entregar su DUI, colocar su dedo o rostro en el sensor, seleccionar su candidato y guardar su recibo.

### 2.2 Operador de Mesa Electoral (Supervisor)

- **Responsabilidad:** Gestionar el flujo de votantes y resolver incidencias técnicas.
- **Interacción:** Con la Terminal de Votación (modo supervisor).
- **Acciones:** Ingresar el DUI del ciudadano, autorizar reintentos, gestionar bloqueos y tokens expirados.

### 2.3 Administrador Electoral

- **Responsabilidad:** Configurar todo el sistema antes de la elección.
- **Interacción:** Con el Panel Administrativo y sistemas backend.
- **Acciones:** Importar el padrón, inscribir ciudadanos biométricamente, configurar elecciones.

### 2.4 Auditor Ciudadano (Portal Público)

- **Responsabilidad:** Verificar la integridad del proceso electoral.
- **Interacción:** Con el Portal de Auditoría desde su navegador web.
- **Acciones:** Introducir el Transaction ID de su recibo para confirmar su voto.

---

## 3. MANUAL DEL CIUDADANO — PROCESO COMPLETO DE VOTACIÓN

<a href='https://postimg.cc/2bxKCc1p' target='_blank'><img src='https://i.postimg.cc/Gt0wJ6rp/inicio.jpg' border='0' alt='inicio'></a>
*Figura 2: Pantalla de bienvenida al ciudadano. Espere a que el operador ingrese su DUI.*

---

### 3.1 Llegada al Centro de Votación

- Preséntese en el centro de votación que le corresponde.
- Debe estar inscrito biométricamente (inscripción realizada en semanas previas).

---

### 3.2 Paso 1: Identificación Visual por DUI

<a href="https://postimg.cc/JtbFqhKC" target="_blank"><img src="https://i.postimg.cc/RC8zv6Hh/identificacion-con-dui.jpg" alt="identificacion-con-dui"></a>
*Figura 3: El operador de mesa ingresa su número de DUI en el campo señalado con la **flecha roja**. Luego presiona el botón **"Verificar"** (recuadro rojo).*

**Acción del Ciudadano:**
- Entregue su **Documento Único de Identidad (DUI)** físico al Operador de Mesa.

**Acción del Operador:**
- Introduce el número de DUI en la Terminal.
- Presiona el botón **"Verificar"** (recuadro rojo en la Figura 3).

---

### 3.3 Paso 2: Verificación en el Padrón Electoral

<a href="https://postimg.cc/2LpP5bwv" target="_blank"><img src="https://i.postimg.cc/sx1yw5Gn/identidad-verificada.jpg" alt="identidad-verificada"></a>
*Figura 4: Pantalla de confirmación de identidad. Aparecerá el nombre del ciudadano (recuadro verde). Si no se cumplen las condiciones, aparecerá un mensaje de error (recuadro rojo).*

**Resultado para el Ciudadano:**
- **Éxito:** La terminal mostrará su nombre y el mensaje: *"Ciudadano habilitado. Proceda a la autenticación biométrica."*
- **Fallo:** Aparecerá un mensaje de error:
    - *"DUI no encontrado en el padrón."*
    - *"Este centro no corresponde a su lugar de votación."*
    - *"Este ciudadano ya ha votado."*

**¿Qué hacer si falla?**
- Pida ayuda al supervisor de mesa. Él verificará su información y le orientará.

---

### 3.4 Paso 3: Solicitud de Sesión Biométrica

- **El ciudadano no ve este paso.** Ocurre automáticamente en el sistema.
- Es una medida de seguridad para evitar que alguien capture su biometría y la use después.

---

### 3.5 Paso 4: Captura de la Muestra Biométrica

**Acción del Ciudadano:**
- Siga las instrucciones en la pantalla de la Terminal:

| Modalidad | Instrucción |
|-----------|-------------|
| **Huella dactilar** | Coloque el dedo indicador sobre el sensor. No lo mueva hasta que la pantalla indique "Listo". |
| **Reconocimiento facial** | Mire fijamente a la cámara durante 2-3 segundos. No use gafas oscuras. |
| **Voz** | Repita la frase que aparece en pantalla en voz clara. |

> **Importante:** Su muestra biométrica nunca se almacena en el sistema. Solo se usa para verificar su identidad y luego se elimina.

---

### 3.6 Paso 5: Procesamiento Interno de Verificación

- **El ciudadano no ve este paso.** Es un proceso automático que toma menos de 1 segundo.
- El sistema compara su muestra en vivo con la plantilla que se registró cuando se inscribió.
- También verifica que usted sea una persona real (no una foto o video) usando tecnología de detección de vida.

---

### 3.7 Paso 6: Decisión de Autenticación y Manejo de Fallos

**Caso 1: Autenticación Exitosa**
- La pantalla mostrará: *"Identidad verificada. Proceda a votar."*
- El sistema pasa al siguiente paso.

**Caso 2: Fallo por Calidad o No Reconocimiento**
- La pantalla mostrará: *"Verificación fallida. Intento X de 3."*
- **Repita el Paso 4** (coloque su dedo o mire a la cámara de nuevo).
- Si falla 3 veces, el sistema lo **bloqueará por 15 minutos** (ver Casos Especiales).

**Caso 3: Bloqueo Temporal (Lockout)**
- La pantalla mostrará: *"Ha superado el número máximo de intentos. Su cuenta ha sido bloqueada temporalmente por 15 minutos. Contacte al supervisor de mesa."*
- **Espere 15 minutos** o pida ayuda al supervisor.

---

### 3.8 Paso 7: Emisión del JWT de Voto

- **El ciudadano no ve este paso.**
- Es el momento crítico donde el sistema **separa su identidad de su voto**.
- A partir de aquí, el sistema no sabe quién es usted. Solo sabe que un voto anónimo va a ser emitido.

---

### 3.9 Paso 8: Selección del Candidato en la Papeleta

<a href='https://postimg.cc/7bL5Xfyw' target='_blank'><img src='https://i.postimg.cc/9Xdym78R/listado-de-postulantes.jpg' border='0' alt='Listado de postulantes'></a>
*Figura 5: Pantalla de selección de candidato. Toque el nombre o foto del candidato de su preferencia (recuadro verde). Luego presione el botón **"Confirmar Voto"** (flecha roja).*

**Acción del Ciudadano:**
1.  Toque la pantalla sobre el nombre o foto del candidato que desea elegir.
2.  Verifique que el candidato seleccionado esté resaltado (recuadro verde en Figura 5).
3.  Presione el botón **"Confirmar Voto"** (flecha roja en Figura 5).

---

### 3.10 Paso 9: Cifrado del Voto y Firma Ciega

- **El ciudadano no ve este paso.**
- Su voto se convierte en un código secreto que nadie puede leer, ni siquiera el sistema.
- La autoridad electoral firma este código sin saber qué candidato eligió.

**¿Por qué es importante?**
- Si alguien le obliga a decir por quién votó, usted no puede demostrarlo porque ni siquiera la autoridad lo sabe.

---

### 3.11 Paso 10: Prueba de Conocimiento Cero

- **El ciudadano no ve este paso.**
- El sistema genera una prueba matemática que demuestra que su voto es válido sin revelar el candidato.

**¿Por qué es importante?**
- Asegura que su voto cuenta para un candidato real, no para una opción inválida.

---

### 3.12 Paso 11: Registro del Voto en Hyperledger Fabric

<a href='https://postimg.cc/B8Gg12kP' target='_blank'><img src='https://i.postimg.cc/DzSYHgc6/el-voto-fue-registrado.jpg' border='0' alt='Voto registrado exitosamente'></a>
*Figura 6: Pantalla de confirmación de registro. El sistema muestra el mensaje de éxito (recuadro verde) y el **Transaction ID** (flecha roja).*

**Acción del Sistema:**
- Su voto cifrado se escribe en una **blockchain** (cadena de bloques).
- Esto es como un libro de contabilidad público que no se puede modificar.
- Participan 3 organizaciones independientes (TSE, Fiscalía, Observadores) que verifican y firman la transacción.

---

### 3.13 Paso 12: Entrega del Recibo y Transaction ID

**Acción del Sistema (Frontend):**
- Muestra el mensaje: **"Su voto ha sido registrado exitosamente."**
- Muestra el **Transaction ID** en pantalla (flecha roja en Figura 6).
- Opcionalmente, imprime un recibo físico.

**Acción del Ciudadano:**
- **Guarde el Transaction ID.** Es su comprobante.
- Con este código podrá verificar en el Portal de Auditoría que su voto fue registrado.

---

### 3.14 Casos Especiales: Token Expirado y Bloqueo Temporal

**A. Token de Voto Expirado**

**Situación:** Después de autenticarse, pasaron más de 5 minutos y no seleccionó su candidato.

**La pantalla mostrará:** *"El tiempo para votar ha expirado. Por favor, contacte al supervisor."*

**Qué hacer:**
1.  Llame al supervisor de mesa.
2.  Él autorizará un nuevo intento desde su panel.
3.  No necesita volver a mostrar su DUI, solo autenticarse biométricamente de nuevo.

**B. Bloqueo Temporal por Fallos Biométricos**

**Situación:** Falló la autenticación biométrica 3 veces seguidas.

**La pantalla mostrará:** *"Bloqueado por 15 minutos. Contacte al supervisor."*

**Qué hacer:**
1.  **Opción 1 (Recomendada):** Espere 15 minutos. El sistema lo desbloqueará automáticamente.
2.  **Opción 2:** Si cree que fue un error técnico (sensor sucio, dedo mojado), pida al supervisor que lo desbloquee manualmente.

> **Nota:** Los desbloqueos manuales quedan registrados para garantizar la transparencia.

---

## 4. MANUAL DEL OPERADOR DE MESA (SUPERVISOR)

<a href='https://postimg.cc/GB5m5jR9' target='_blank'><img src='https://i.postimg.cc/bJzSd6F0/dashboard.jpg' border='0' alt='Dashboard del supervisor'></a>
*Figura 7: Panel de control del supervisor. Las alertas de bloqueo aparecen en el área marcada con **recuadro rojo**.*

---

### 4.1 Panel de Control del Supervisor

<a href="https://postimg.cc/q6tGhj11" target="_blank"><img src="https://i.postimg.cc/J0xYFFZC/ultimos-eventos.jpg" alt="ultimos-eventos"></a>

- Acceda al panel especial en la Terminal (requiere credenciales de supervisor).
- En el panel visualizará:
    - Estado actual del centro (número de votantes, incidencias).
    - Lista de ciudadanos en estado `PENDIENTE`, `EN_PROCESO`, `BLOQUEADO_TEMP` o `TOKEN_EXPIRADO`.
    - Alertas activas (bloqueos, fallos).

---

### 4.2 Gestión de Identificación del Ciudadano

**Flujo estándar:**
1.  Reciba el DUI del ciudadano.
2.  Escanéelo o ingréselo en el campo correspondiente.
3.  Presione **"Verificar"** (recuadro rojo en Figura 3).

**Si el sistema rechaza al ciudadano:**
1.  Verifique que el DUI sea correcto (sin errores de digitación).
2.  Confirme que el ciudadano esté en el centro correcto.
3.  Si es un error administrativo, contacte al Administrador Electoral para rectificar.

---

### 4.3 Manejo de Bloqueos Temporales (Lockout)

**Alerta de bloqueo:** La terminal notifica: *"Ciudadano [Nombre] bloqueado por 15 minutos."*

**Evaluación:**
1.  Verifique la identidad del ciudadano (compare con su DUI físico).
2.  Pregunte si tiene dificultades (dedo mojado, lentes, etc.).

**Decisiones:**

| Opción | Acción | Cuándo usarla |
|--------|--------|---------------|
| **A (Recomendada)** | Esperar 15 minutos. El sistema desbloquea automáticamente. | Siempre que sea posible. |
| **B (Excepcional)** | Desbloqueo manual desde el panel: botón **"Desbloquear Ciudadano"**. Ingrese su contraseña. | Si el problema fue técnico (sensor defectuoso). |

**Registro de auditoría:** Cada desbloqueo manual queda registrado (quién, cuándo, motivo).

---

### 4.4 Manejo de Tokens de Voto Expirados

**Alerta:** *"El tiempo para votar ha expirado para el ciudadano [Nombre]."*

**Evaluación:**
1.  Confirme que el ciudadano sigue en la fila y desea votar.
2.  Asegúrese de que no se haya ido y regresado horas después.

**Acción:**
1.  En su panel, seleccione al ciudadano con estado `TOKEN_EXPIRADO`.
2.  Presione **"Reiniciar Autenticación Biométrica"**.
3.  El ciudadano autentica nuevamente y se emite un nuevo JWT.

**Nota:** El ciudadano no tiene que pasar por la identificación DUI nuevamente.

---

### 4.5 Alerta de Fraude o Incidente

**Botón de "Alerta" en el panel:**
- **Úselo si detecta:**
    - Intento de suplantación de identidad.
    - Ciudadano tratando de votar dos veces.
    - Comportamiento sospechoso.

**Acción del sistema:**
- Notifica al Administrador Electoral y al centro de monitoreo.
- Registra el incidente en la auditoría inmutable.

---

### 4.6 Cierre de la Jornada Electoral

**Al finalizar la jornada:**
1.  Verifique que no haya ciudadanos en `EN_PROCESO` o `TOKEN_EXPIRADO`.
2.  Cierre la sesión del centro.
3.  El sistema generará un **reporte de cierre** con:
    - Total de votantes.
    - Total de votos registrados en Fabric.
    - Incidencias reportadas.
    - Conteo local vs. blockchain.

4.  Firme digitalmente el reporte.

---

## 5. MANUAL DEL ADMINISTRADOR ELECTORAL

### 5.1 Preparación del Padrón Electoral

**Acción del Administrador:**
1.  **Importar el padrón:**
    - Desde el Panel Administrativo, importe el archivo CSV/XML del padrón electoral oficial.
2.  **Carga automática:**
    - El sistema crea un registro por cada ciudadano con estado `SIN_REGISTRO`.
    - Genera un **External_ID** (código secreto) para cada ciudadano.

3.  **Asignación de centros:**
    - Verifique y corrija la asignación de centros de votación.

---

### 5.2 Proceso de Inscripción Biométrica (Enrollment)

**Este proceso ocurre semanas antes de la elección.**

**Acción del Administrador:**
1.  El ciudadano se presenta en el centro de inscripción.
2.  Capture sus datos y muestra biométrica (huella, rostro o voz).
3.  El sistema envía la muestra a `BiometriX` para su procesamiento.
4.  `BiometriX` verifica:
    - Que la muestra sea de una persona viva.
    - Que tenga calidad suficiente.
    - Cifra y almacena la plantilla.
5.  El estado del ciudadano cambia a `INSCRITO`.

---

### 5.3 Configuración de Elecciones y Claves Criptográficas

**Acción del Administrador:**
1.  **Crear elección:**
    - Defina nombre, fecha, hora de apertura y cierre.
    - Ingrese la lista de candidatos.

2.  **Generar claves ElGamal:**
    - Presione **"Generar Claves Criptográficas"**.
    - La clave pública se publica. La clave privada se divide en 5 partes (Shamir 3-de-5).
    - Se necesitan 3 de 5 custodios para descifrar los totales.

3.  **Configurar umbral biométrico:**
    - Ajuste el valor de sensibilidad (0.85 por defecto).
    - Más alto = más estricto. Más bajo = más permisivo.

---

### 5.4 Despliegue del Chaincode en Fabric

**Acción del Administrador:**
1.  Verifique que todos los peers estén activos.
2.  Instale el chaincode en todos los peers.
3.  Cada organización aprueba el chaincode.
4.  Realice el commit del chaincode.
5.  Ejecute una transacción de prueba para confirmar su correcto funcionamiento.

---

### 5.5 Gestión de Claves en Vault

**Acción del Administrador:**
1.  La clave AES-256 se almacena en **HashiCorp Vault**.
2.  Solo `BiometriX` tiene permiso para leer esta clave.
3.  Los administradores humanos no tienen acceso directo a la clave.

---

### 5.6 Apertura Oficial de la Elección

**Acción del Administrador:**
1.  Verifique que todos los ciudadanos estén `INSCRITO` o `PENDIENTE`.
2.  Presione **"Abrir Elección"**.
3.  El sistema cambia el estado a `ABIERTA` en todos los componentes.
4.  Se envían notificaciones a todos los centros de votación.

---

## 6. PORTAL DE AUDITORÍA PÚBLICA

### 6.1 Acceso al Portal

- Ingrese a la URL pública del **Portal de Auditoría** desde cualquier navegador.
- No requiere usuario ni contraseña.

---

### 6.2 Verificación por Transaction ID

<a href='https://postimg.cc/t18fRbqt' target='_blank'><img src='https://i.postimg.cc/tTgH5R8f/identidad-verificada.jpg' border='0' alt='Portal de auditoría'></a>
*Figura 8: Portal de Auditoría. Ingrese el código Transaction ID en el campo señalado con **flecha roja** y presione **"Verificar"** (recuadro rojo).*

**Acción del Ciudadano:**
1.  Introduzca el **Transaction ID** de su recibo.
2.  Presione **"Verificar"**.

**Resultado:**
- **Éxito:** *"Voto registrado exitosamente en el bloque #XXXX, transacción #YYYY, a las HH:MM:SS del DD/MM/AAAA."*
- **Fallo:** *"No se encontró ninguna transacción con este ID."*

---

### 6.3 Interpretación del Resultado

- **Confirmación:** Su voto fue procesado y registrado de forma inmutable.
- **Privacidad:** El portal **no muestra** el candidato ni su identidad.
- **Transparencia:** Cualquier ciudadano puede verificar cualquier Transaction ID.

---

## 7. PREGUNTAS FRECUENTES (FAQ)

### 7.1 ¿Qué hago si olvidé traer mi DUI?

No podrá votar. El DUI es obligatorio para identificarse en el sistema. Regrese por su documento y vuelva a la fila.

---

### 7.2 ¿Qué pasa si el sensor biométrico no reconoce mi huella?

Tiene hasta 3 intentos. Si falla, el sistema lo bloqueará por 15 minutos. Solicite ayuda al supervisor de mesa. Él puede desbloquearlo manualmente si el problema es técnico (dedo mojado, sensor sucio).

---

### 7.3 ¿Puedo votar en otro centro de votación?

No. El sistema solo permite votar en el centro asignado en el padrón electoral. Si intenta votar en otro centro, aparecerá un mensaje de error.

---

### 7.4 ¿Cómo sé que mi voto fue registrado correctamente?

Al finalizar, recibirá un recibo con un **Transaction ID**. Ingrese ese código en el Portal de Auditoría (URL pública). Si aparece la transacción, su voto está registrado en la blockchain.

---

### 7.5 ¿Qué hago si el sistema se apaga o se congela durante mi votación?

No se preocupe. Informe inmediatamente al supervisor de mesa. El sistema tiene mecanismos de recuperación. Su sesión de votación no se perderá. El supervisor podrá reiniciar su autenticación biométrica.

---

### 7.6 ¿Puedo cambiar mi voto después de confirmarlo?

No. Una vez confirmado, el voto se cifra y se registra en la blockchain de forma inmutable. No es posible modificarlo ni anularlo.

---

### 7.7 ¿El sistema sabe por quién voté?

No. El sistema separa su identidad de su voto mediante un JWT anónimo. Ni SVED, ni BiometriX, ni la blockchain pueden saber por quién votó. Su voto es completamente secreto.

---

### 7.8 ¿Qué hago si se me venció el tiempo para votar?

Si pasaron más de 5 minutos después de autenticarse y no votó, llame al supervisor de mesa. Él podrá autorizar un nuevo intento de autenticación biométrica desde su panel. No necesita mostrar su DUI nuevamente.

---

## 8. GLOSARIO DE TÉRMINOS

| Término | Significado |
|---------|-------------|
| **Blockchain** | Un libro de contabilidad digital que no se puede modificar ni borrar. |
| **DUI** | Documento Único de Identidad. Su identificación oficial. |
| **Enrollment** | Proceso de inscripción biométrica (captura de huella/rostro) que ocurre antes de la elección. |
| **JWT** | Un "pase digital" temporal que el sistema genera para usted después de autenticarse. No contiene su identidad. |
| **Liveness Detection** | Tecnología que verifica que usted es una persona real, no una foto o video. |
| **Lockout** | Bloqueo temporal de su cuenta después de 3 intentos fallidos de autenticación. |
| **Transaction ID** | Código único que recibe después de votar. Sirve para verificar su voto en el Portal de Auditoría. |
| **ZKP** | Prueba matemática que demuestra que su voto es válido sin revelar el candidato. |

---

**FIN DEL MANUAL DE USUARIO FINAL**

---

*Este manual ha sido desarrollado conforme a la Guía de Especificaciones de Documentación de Software (Versión 2026) y los Diagramas Técnicos del Sistema SVED + BiometriX.*