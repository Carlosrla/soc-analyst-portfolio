# Triage de alertas — Semana 2, Módulo 1

**Fecha:** 07/09/2026
**Analista:** Carlos

---

## Alerta 1

**Alerta:** Inicio de sesión de Administrador de dominio desde una IP externa a las 03:14 AM, geolocalizada en un país donde la empresa no opera.

**Host/Usuario:** Administrador de dominio (cuenta, sin más detalle en el enunciado)

**Contexto recogido:**
- Comparar contra el histórico/baseline de esa cuenta: ¿ha iniciado sesión antes desde esa ubicación o a esa hora? ¿Es un patrón conocido o una primera vez?
- Al ser una cuenta de Administrador de dominio (la más crítica de una red Windows, con capacidad de crear usuarios y moverse por toda la red), comprobar qué hizo la sesión una vez dentro: ¿solo inició sesión sin más actividad, o además creó cuentas, accedió a otros servidores, descargó algo?

**Evaluación ATT&CK:** Initial Access — es el momento de usar una cuenta válida para entrar, no el momento de haberla conseguido (eso sería Credential Access, una fase anterior no visible en esta alerta).

**Decisión:** Escalado

**Justificación:** Hay suficientes indicadores anómalos (geolocalización fuera de la operativa de la empresa, hora inusual, cuenta crítica de administrador) que no puedo descartar por mí mismo sin más contexto, así que escalo para que se compruebe el histórico de la cuenta y la actividad realizada durante la sesión.

---

## Alerta 2

**Alerta:** Usuario descarga 40 archivos de una carpeta compartida en 2 minutos, fuera de su patrón habitual.

**Host/Usuario:** No identificado en el enunciado.

**Contexto recogido:**
- Qué tipo de archivos son (documentación de RRHH, relativamente inofensiva, frente a bases de datos o archivos con información de clientes, mucho más sensibles).
- Quién es el usuario y si su rol/histórico justifica esa descarga (por ejemplo, alguien de Business Intelligence generando un informe periódico, frente a alguien sin relación aparente con esos datos).

**Evaluación ATT&CK:** Collection (con posible escalada a Exfiltration si posteriormente se confirma que los archivos salen de la red de la empresa).

**Decisión:** Escalado

**Justificación:** Con los datos disponibles no tengo contexto suficiente para cerrar el caso por mí mismo — desconozco el tipo de archivos y si el usuario tiene una razón laboral legítima para esta descarga. El propio patrón (fuera de su comportamiento habitual, detectado por el sistema) es una señal objetiva que no puedo descartar sin esa información adicional.

---

## Alerta 3

**Alerta:** Proceso `powershell.exe` ejecutado con el parámetro `-EncodedCommand` desde Word (`winword.exe` como proceso padre).

**Host/Usuario:** No identificado en el enunciado.

**Contexto recogido:**
- Word se usa normalmente para redacción de texto, no para lanzar procesos del sistema — que aparezca como proceso padre de PowerShell es anómalo. Lo más probable es que exista un documento con una macro maliciosa (código VBA que se ejecuta automáticamente al abrir el archivo), que es la que lanza PowerShell como proceso hijo.
- El parámetro `-EncodedCommand` codifica el comando en Base64, ocultando su contenido real. El objetivo del atacante es dificultar la lectura inmediata del log y ralentizar la detección.
- A diferencia de otras alertas, aquí no hace falta contexto externo (histórico de usuario, rol, etc.): el propio patrón técnico (Word lanzando PowerShell con un comando ofuscado) prácticamente no tiene explicación legítima en un entorno de oficina normal.

**Evaluación ATT&CK:** Execution — es el momento en que el código se ejecuta dentro del sistema; la alerta no indica, de momento, ninguna comunicación hacia el exterior.

**Decisión:** Escalado

**Justificación:** El patrón (Word lanzando PowerShell con un comando ofuscado) no tiene explicación legítima habitual en un entorno de oficina, por lo que constituye evidencia suficiente por sí sola para escalar como probable ejecución de código malicioso, pendiente de análisis del documento de origen.

---

## Alerta 4

**Alerta:** Múltiples intentos fallidos de inicio de sesión (15) seguidos de uno exitoso en la cuenta de un empleado del departamento de contabilidad.

**Host/Usuario:** Empleado de contabilidad, sin nombre especificado en el enunciado.

**Contexto recogido:**
- El patrón (15 fallos seguidos de 1 éxito, en poco tiempo, contra la misma cuenta) es característico de un ataque de fuerza bruta.
- Existen dos explicaciones posibles: (1) el propio empleado, que olvidó su contraseña y fue probando variaciones hasta acertar; (2) un atacante realizando fuerza bruta hasta dar con la combinación correcta.
- La comprobación clave para distinguir ambos casos es el origen (geolocalización/IP) de los 15 intentos y del inicio de sesión exitoso: si provienen de la red habitual del empleado (oficina, VPN corporativa), apunta a error propio; si provienen de una IP externa desconocida, apunta a fuerza bruta real.

**Evaluación ATT&CK:** Credential Access (los intentos fallidos, buscando dar con la contraseña) e Initial Access (el inicio de sesión exitoso final).

**Decisión:** Escalado

**Justificación:** Se escala la alerta al existir una señal objetiva compatible con fuerza bruta (patrón de fallos repetidos seguido de éxito); falta el dato de geolocalización de los intentos para poder descartar o confirmar con garantías, por lo que no puedo cerrar el caso por mí mismo.

---

## Alerta 5

**Alerta:** Un servidor interno realiza conexiones salientes periódicas cada 60 segundos exactos a un dominio registrado hace 3 días.

**Host/Usuario:** No especificado en el enunciado.

**Contexto recogido:**
- Un patrón de conexión cada 60 segundos exactos, con precisión de reloj, no es propio de comportamiento humano ni de la mayoría de aplicaciones legítimas. Este comportamiento se conoce como *beaconing*: una máquina comprometida se comunica periódicamente con un controlador externo para comprobar si tiene nuevas instrucciones — es característico de la táctica Command and Control.
- Un dominio registrado hace solo 3 días es coherente con infraestructura de "usar y tirar", típica de campañas maliciosas que renuevan su infraestructura conforme se detecta y bloquea. No es una prueba definitiva por sí sola (existen dominios legítimos nuevos, y también casos de suplantación de dominios reconocidos), pero estadísticamente es mucho menos probable que un dominio tan reciente sea legítimo.
- La combinación de ambas señales (beaconing + dominio recién registrado) es lo bastante específica como para no requerir contexto externo adicional: el patrón técnico ya constituye evidencia suficiente.

**Evaluación ATT&CK:** Command and Control.

**Decisión:** Escalado

**Justificación:** El patrón combinado (beaconing + dominio recién registrado) no tiene explicación legítima habitual, por lo que constituye evidencia suficiente para escalar como probable actividad de Command and Control, pendiente de confirmación técnica (análisis del dominio, aislamiento del host).
