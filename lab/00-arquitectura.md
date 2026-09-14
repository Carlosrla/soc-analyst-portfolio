# Arquitectura del laboratorio — Estado inicial (Módulo 0)

**Fecha:** 03/09/2026

## Topología

Red interna Host-Only (vboxnet0) en 10.10.10.0/24, con tres VMs operativas y una reservada:

- WIN11-CLIENT — 10.10.10.10
- UBUNTU-SRV — 10.10.10.20
- UBUNTU-DESKTOP — 10.10.10.30
- WIN-SERVER (DC) — 10.10.10.5 (reservada, se despliega en el Módulo 5)

Cada VM tiene un segundo adaptador NAT, exclusivamente de salida, para actualizaciones del sistema operativo. El laboratorio no es accesible desde la red doméstica del host (adaptador Host-Only, no Bridge).

## Máquinas virtuales

| VM | SO | RAM | vCPU | Disco | IP (Host-Only) | Estado |
|---|---|---|---|---|---|---|
| WIN11-CLIENT | Windows 11 Enterprise (eval) | 4-6 GB | 2 | 60 GB | 10.10.10.10 | Operativa |
| UBUNTU-SRV | Ubuntu Server 24.04.1 LTS | 2 GB | 2 | 25 GB | 10.10.10.20 | Operativa |
| UBUNTU-DESKTOP | Ubuntu Desktop 24.04.1 LTS | 4 GB | 2 | 30 GB | 10.10.10.30 | Operativa |
| WIN-SERVER (DC) | Windows Server 2025 (eval) | — | — | — | 10.10.10.5 (reservada) | Pendiente — Módulo 5 |

## Hipervisor

VirtualBox 7.0, con red interna `vboxnet0` (10.10.10.0/24, sin DHCP — IPs asignadas manualmente en cada VM).

## Verificación de conectividad

Ping cruzado confirmado entre las tres VMs operativas (10.10.10.10 <-> 10.10.10.20 <-> 10.10.10.30). Fue necesario abrir una regla ICMP en el Firewall de Windows Defender de WIN11-CLIENT, que por defecto bloquea ICMP entrante:

netsh advfirewall firewall add rule name="ICMP Allow" protocol=icmpv4 dir=in action=allow


## Política de snapshots

- `00-clean-install`: tomado en las tres VMs tras confirmar red, Guest Additions y herramientas base instaladas.
- Convención: `NN-descripcion-corta`, numeración incremental por VM.
- Backup real (.ova) al cierre de cada bloque del programa, a almacenamiento externo.

## Caja de herramientas instalada

- **Host:** VS Code, Git, Wireshark.
- **WIN11-CLIENT:** Guest Additions, Sysinternals Suite (C:\Sysinternals), PowerShell 7.
- **UBUNTU-SRV / UBUNTU-DESKTOP:** Guest Additions, htop, net-tools, curl, jq, python3-venv, git.

## Próximos pasos

Módulo 5 desplegará WIN-SERVER como Domain Controller en 10.10.10.5, ampliando esta topología con Active Directory.
