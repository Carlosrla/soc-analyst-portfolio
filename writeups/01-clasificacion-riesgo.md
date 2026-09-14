# Clasificación de riesgo — Clínica dental

**Fecha:** 07/09/2026
**Analista:** Carlos

**Escenario analizado:** Una pequeña clínica dental usa un servidor Windows Server 2016 sin actualizar desde hace 2 años, expuesto a Internet vía RDP para que el dentista acceda desde casa. Almacena historiales médicos de pacientes.

## 1. Vulnerabilidades identificadas

- **Sistema operativo desactualizado (2 años sin parchear):** aumenta la superficie de ataque, ya que expone al servidor a vulnerabilidades conocidas y públicamente documentadas que ya tienen exploits disponibles.
- **RDP expuesto directamente a Internet, sin VPN por delante:** sin ese filtro previo, cualquier persona o bot con acceso a Internet puede intentar conectarse y atacar el servicio directamente (fuerza bruta, escaneo automatizado), en vez de tener que superar primero una capa de autenticación adicional.

## 2. Amenazas plausibles

- **Amenaza real:** un grupo organizado dedicado a la extorsión mediante ransomware, especializado en atacar pequeños negocios con poca o nula capacidad defensiva y datos sensibles que puedan secuestrar (como historiales médicos). Tiene tanto la capacidad técnica como el interés específico en este tipo de objetivo.
- **Amenaza poco probable:** un atacante oportunista sin objetivo definido ("el curioso"), que puede tener nociones básicas de escaneo de puertos pero carece de interés real en atacar clínicas dentales — se limitaría a detectar el servicio expuesto, sin explotarlo activamente.

## 3. Valoración del riesgo

**Riesgo: alto.** Existen vulnerabilidades claras y explotables (sistema desactualizado, RDP expuesto), y a la vez existe una amenaza real y plausible con capacidad e interés en este tipo de objetivo (grupos de ransomware especializados en el sector salud). Al coincidir ambas piezas —vulnerabilidad y amenaza— el riesgo se dispara; si faltara cualquiera de las dos, el riesgo sería bajo.

## 4. Controles recomendados

- **Preventivo:** actualizar el sistema operativo a una versión soportada, y desplegar una VPN delante del servicio RDP para que ya no sea accesible directamente desde Internet.
- **Detectivo:** vigilar los logs de autenticación en busca de patrones de intentos fallidos repetidos en poco tiempo (indicio de fuerza bruta), con una herramienta que analice esos logs y genere una alerta automática.
- **Correctivo:** mantener copias de seguridad de los historiales médicos fuera de la red principal (backup offline/air-gapped), para poder restaurar el servicio sin depender de pagar un rescate si el sistema llega a ser comprometido.

## 5. Conclusión

La prioridad inmediata es cerrar la exposición directa del RDP a Internet, ya que es el vector de entrada más claro y de donde probablemente vendría la mayor filtración de datos. Desplegar una VPN delante de ese servicio obliga a cualquier atacante a superar una barrera adicional antes de poder siquiera intentar acceder al puerto, reduciendo drásticamente la superficie de ataque expuesta.
