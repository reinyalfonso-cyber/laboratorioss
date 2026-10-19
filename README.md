<div align="center">

# 🛡️ VulnHub Boot2Root — Write-ups & Pentesting Labs

![VulnHub](https://img.shields.io/badge/Plataforma-VulnHub-f37021?style=for-the-badge)
![Pentesting](https://img.shields.io/badge/Enfoque-Red_Team_%26_Defensa-6f42c1?style=for-the-badge)
![Entorno](https://img.shields.io/badge/Entorno-Linux_%7C_VirtualBox-0078D6?style=for-the-badge&logo=linux&logoColor=white)
![Estado](https://img.shields.io/badge/Laboratorios-Documentados_Paso_a_Paso-2ea44f?style=for-the-badge)

**Repositorio técnico dedicado a la documentación exhaustiva de auditorías de seguridad, pruebas de penetración (Boot2Root), explotación de vulnerabilidades y escalada de privilegios sobre entornos vulnerables de VulnHub.**

</div>

---

## 📌 Descripción del Proyecto

Este repositorio centraliza las resoluciones técnicas completas (*Write-ups*) de máquinas vulnerables desplegadas en laboratorio local. Cada informe está redactado con un enfoque práctico y estructurado, integrando **evidencias gráficas paso a paso dentro de la misma explicación**, análisis detallado de los vectores de ataque utilizados, comandos ejecutados y recomendaciones de mitigación para corregir las vulnerabilidades expuestas.

### 🎯 Objetivos del Repositorio
* **Trazabilidad Completa:** Documentar cada fase de la intrusión de principio a fin, permitiendo replicar la auditoría sin pasos omitidos.
* **Evidencia Integrada:** Incluir capturas de pantalla directamente en cada comando y hallazgo crítico para validar visualmente el proceso.
* **Visión Ofensiva y Defensiva:** No limitarse únicamente a obtener las banderas (*flags*), sino comprender la causa raíz del fallo y cómo proteger el activo afectado.

---

## 🗂️ Catálogo de Máquinas Resueltas (VulnHub)

Haz clic en el enlace de cada máquina para acceder a su carpeta con la explicación completa, comandos y capturas paso a paso:

| # | Máquina | Plataforma | Sistema Operativo | Objetivo | Estado | Documentación Técnica |
| :---: | :--- | :---: | :---: | :---: | :---: | :---: |
| **01** | **C-1** | VulnHub | Linux | Boot2Root (User & Root) | ✅ Completado | [📖 Ver Explicación Completa](./VulnHub/C-1/) |
| **02** | **C-2** | VulnHub | Linux | Boot2Root (User & Root) | ✅ Completado | [📖 Ver Explicación Completa](./VulnHub/C-2/) |
| **03** | **Jangow** | VulnHub | Linux | Boot2Root (User & Root) | ✅ Completado | [📖 Ver Explicación Completa](./VulnHub/Jangow/) |
| **04** | **Planet** | VulnHub | Linux | Boot2Root (User & Root) | ✅ Completado | [📖 Ver Explicación Completa](./VulnHub/Planet/) |

---

## 🧭 Metodología de Trabajo Documentada en cada Laboratorio

Cada máquina dentro de la carpeta [`./VulnHub/`](./VulnHub/) sigue un flujo estandarizado de 5 fases técnicas:

### 1️⃣ Reconocimiento y Enumeración de Red
* Descubrimiento de hosts activos en el segmento de red local (`arp-scan`, `netdiscover`).
* Identificación de puertos abiertos, versiones de servicios y sistema operativo (`nmap`).
* Análisis de superficie de ataque expuesta.

### 2️⃣ Enumeración de Servicios y Análisis de Vulnerabilidades
* Fuerza bruta de directorios, archivos y rutas ocultas en servidores web (`gobuster`, `dirb`, `wfuzz`, `ffuf`).
* Inspección de código fuente, cabeceras HTTP, tecnologías subyacentes y versiones de CMS/Plugins.
* Comprobación de servicios adicionales como FTP, SSH, SMB o bases de datos expuestas.

### 3️⃣ Explotación y Acceso Inicial (*Foothold*)
* Identificación y validación de vulnerabilidades (ejecución remota de comandos, inyecciones SQL, inclusión de archivos locales/remotos, carga arbitraria de archivos o credenciales débiles).
* Manipulación de peticiones mediante `Burp Suite` o `curl`.
* Obtención de consola interactiva (*Reverse Shell*) y estabilización de la terminal (`TTY`).

### 4️⃣ Post-Explotación y Escalada de Privilegios (*Privilege Escalation*)
* Enumeración interna del sistema comprometido (`sudo -l`, permisos SUID/SGID, capacidades de binarios, tareas programadas en `cron`, variables de entorno `PATH` y búsqueda de credenciales en configuraciones).
* Análisis de vulnerabilidades de kernel o abuso de binarios del sistema (*GTFOBins*).
* Obtención de privilegios de superusuario (`root`) y validación de compromisos.

### 5️⃣ Mitigación y Remediación (*Hardening*)
* Análisis de la causa raíz que permitió tanto el acceso inicial como la escalada de privilegios.
* Recomendaciones técnicas de configuración segura, aplicación de parches y principio de mínimo privilegio.

---

## 🛠️ Arsenal de Herramientas Utilizadas

| Fase | Herramientas Principales |
| :--- | :--- |
| **Descubrimiento y Escaneo** | `Nmap`, `arp-scan`, `netdiscover`, `whatweb` |
| **Enumeración Web** | `Gobuster`, `Dirb`, `FFUF`, `WPScan`, `Nikto`, `Curl` |
| **Análisis y Explotación** | `Burp Suite`, `SQLMap`, `Searchsploit`, `Netcat`, `Hydra`, `John the Ripper` |
| **Post-Explotación (Linux)** | `LinPEAS`, `GTFOBins`, `find`, `pspy`, `strings`, `Python3 / Bash` |
| **Virtualización y Entorno** | `VirtualBox`, `Kali Linux / Ubuntu` |

---

## 🏗️ Estructura del Repositorio

El repositorio está organizado de forma directa para facilitar la lectura continua de cada laboratorio con sus imágenes integradas en el propio documento:

```text
laboratorioss/
│
├── README.md                     # Portada general, metodología e índice de máquinas
│
└── VulnHub/                      # Directorio de la plataforma VulnHub
    ├── C-1/
    │   └── README.md             # Write-up paso a paso con capturas integradas de C-1
    ├── C-2/
    │   └── README.md             # Write-up paso a paso con capturas integradas de C-2
    ├── Jangow/
    │   └── README.md             # Write-up paso a paso con capturas integradas de Jangow
    └── Planet/
        └── README.md             # Write-up paso a paso con capturas integradas de Planet
