# VulnHub Boot2Root | Informes de Pruebas de Penetración

Repositorio técnico de auditorías ofensivas sobre entornos vulnerables de VulnHub. Cada informe documenta el ciclo completo de una intrusión controlada, desde el reconocimiento hasta la remediación, con evidencia gráfica integrada y análisis de causa raíz.

&nbsp;

| | |
| :-- | :-- |
| **Alcance** | Máquinas Boot2Root de VulnHub en laboratorio local aislado |
| **Enfoque** | Seguridad ofensiva con recomendaciones defensivas |
| **Formato** | Informe técnico por máquina, reproducible paso a paso |
| **Marcos de referencia** | PTES, OWASP Testing Guide, NIST SP 800-115, MITRE ATT&CK, CVSS v3.1 |
| **Entorno** | Linux, VirtualBox, red aislada |

&nbsp;

---

&nbsp;

## Índice

1. [Resumen ejecutivo](#resumen-ejecutivo)
2. [Alcance y reglas de compromiso](#alcance-y-reglas-de-compromiso)
3. [Catálogo de máquinas](#catálogo-de-máquinas)
4. [Metodología](#metodología)
5. [Alineación con MITRE ATT&CK](#alineación-con-mitre-attck)
6. [Stack tecnológico y herramientas](#stack-tecnológico-y-herramientas)
7. [Entorno de laboratorio](#entorno-de-laboratorio)
8. [Estructura de los informes](#estructura-de-los-informes)
9. [Clasificación de hallazgos](#clasificación-de-hallazgos)
10. [Estructura del repositorio](#estructura-del-repositorio)
11. [Convenciones de documentación](#convenciones-de-documentación)
12. [Aviso legal](#aviso-legal)

&nbsp;

---

&nbsp;

## Resumen ejecutivo

Este repositorio centraliza las auditorías completas realizadas sobre máquinas vulnerables, tratadas como si fueran activos reales de un cliente. El objetivo no se limita a obtener las flags: cada informe busca demostrar un proceso de trabajo profesional, verificable y orientado a la mejora de la seguridad del activo analizado.

Principios de trabajo:

- **Trazabilidad total.** Cada fase queda documentada sin pasos omitidos, de forma que cualquier revisor pueda reproducir el resultado.
- **Evidencia integrada.** Las capturas acompañan a cada comando y a cada hallazgo relevante, en el mismo punto del documento donde se explican.
- **Causa raíz antes que explotación.** Cada fallo se analiza hasta su origen técnico y se acompaña de una medida correctiva concreta.
- **Doble perspectiva.** Todo vector ofensivo se cierra con su contramedida defensiva.

&nbsp;

---

&nbsp;

## Alcance y reglas de compromiso

| Elemento | Definición |
| :-- | :-- |
| **Objetivos en alcance** | Máquinas virtuales de VulnHub desplegadas localmente |
| **Fuera de alcance** | Cualquier sistema, red o servicio ajeno al laboratorio |
| **Tipo de prueba** | Caja negra, sin credenciales ni información previa |
| **Meta de cada ejercicio** | Acceso inicial, escalada a `root` y obtención de `user.txt` y `root.txt` |
| **Aislamiento** | Red local sin salida a Internet ni a redes de producción |

&nbsp;

---

&nbsp;

## Catálogo de máquinas

| N.º | Máquina | Sistema | Objetivo | Estado | Informe |
| :-: | :-- | :-: | :-- | :-: | :-- |
| 01 | C-1 | Linux | Boot2Root (user y root) | Completada | [Ver informe](./VulnHub/C-1/) |
| 02 | C-2 | Linux | Boot2Root (user y root) | Completada | [Ver informe](./VulnHub/C-2/) |
| 03 | Jangow | Linux | Boot2Root (user y root) | Completada | [Ver informe](./VulnHub/Jangow/) |
| 04 | Planet | Linux | Boot2Root (user y root) | Completada | [Ver informe](./VulnHub/Planet/) |

&nbsp;

---

&nbsp;

## Metodología

Todas las máquinas siguen un flujo estandarizado de cinco fases, inspirado en PTES y en NIST SP 800-115. Esto permite comparar resultados entre laboratorios y mantener un nivel de calidad homogéneo.

&nbsp;

### Fase 1. Reconocimiento y descubrimiento de red

- Descubrimiento de hosts activos en el segmento local (`arp-scan`, `netdiscover`).
- Identificación de puertos abiertos, versiones de servicios y sistema operativo (`nmap`).
- Delimitación de la superficie de ataque expuesta.

&nbsp;

### Fase 2. Enumeración de servicios y análisis de vulnerabilidades

- Descubrimiento de directorios, archivos y rutas ocultas en servidores web (`gobuster`, `dirb`, `wfuzz`, `ffuf`).
- Inspección de código fuente, cabeceras HTTP, tecnologías subyacentes y versiones de CMS y plugins (`whatweb`, `wpscan`, `nikto`).
- Revisión de servicios adicionales: FTP, SSH, SMB y bases de datos expuestas.

&nbsp;

### Fase 3. Explotación y acceso inicial (foothold)

- Identificación y validación de vulnerabilidades: ejecución remota de comandos, inyección SQL, inclusión de archivos locales y remotos, carga arbitraria de archivos y credenciales débiles.
- Manipulación y repetición de peticiones con `Burp Suite` y `curl`.
- Obtención de shell inversa y estabilización de la TTY.

&nbsp;

### Fase 4. Post-explotación y escalada de privilegios

- Enumeración interna del sistema: `sudo -l`, permisos SUID y SGID, capabilities de binarios, tareas `cron`, variable `PATH` y credenciales en ficheros de configuración.
- Análisis de vulnerabilidades de kernel y abuso de binarios legítimos (GTFOBins).
- Obtención de privilegios de `root` y validación del compromiso total.

&nbsp;

### Fase 5. Mitigación y remediación (hardening)

- Análisis de la causa raíz del acceso inicial y de la escalada de privilegios.
- Recomendaciones de configuración segura, gestión de parches y principio de mínimo privilegio.

&nbsp;

---

&nbsp;

## Alineación con MITRE ATT&CK

Correspondencia general entre las fases de trabajo y las tácticas del marco MITRE ATT&CK for Enterprise.

| Fase | Tácticas ATT&CK relacionadas |
| :-- | :-- |
| 1. Reconocimiento | Reconnaissance (TA0043), Discovery (TA0007) |
| 2. Enumeración | Discovery (TA0007), Reconnaissance (TA0043) |
| 3. Explotación | Initial Access (TA0001), Execution (TA0002), Credential Access (TA0006) |
| 4. Escalada de privilegios | Privilege Escalation (TA0004), Persistence (TA0003), Credential Access (TA0006) |
| 5. Remediación | Mitigaciones asociadas a cada técnica identificada |

&nbsp;

---

&nbsp;

## Stack tecnológico y herramientas

### Plataforma y entorno

| Componente | Uso |
| :-- | :-- |
| **Kali Linux / Ubuntu** | Sistema operativo de la máquina atacante |
| **VirtualBox** | Virtualización de atacante y objetivos, con red aislada |
| **Bash** | Automatización de tareas y análisis en consola |
| **Python 3** | Scripting, servidores HTTP temporales y adaptación de exploits |
| **Markdown** | Documentación de cada informe |
| **Git y GitHub** | Control de versiones y publicación |

&nbsp;

### Herramientas por fase

| Fase | Herramienta | Propósito |
| :-- | :-- | :-- |
| Descubrimiento | `arp-scan`, `netdiscover` | Localización de hosts en la red local |
| Escaneo | `nmap` | Puertos, versiones de servicios y detección de SO |
| Fingerprinting web | `whatweb`, `nikto`, `curl` | Tecnologías, cabeceras y configuraciones inseguras |
| Fuerza bruta de rutas | `gobuster`, `dirb`, `ffuf`, `wfuzz` | Directorios, archivos y parámetros ocultos |
| CMS | `wpscan` | Enumeración de versiones, plugins y usuarios en WordPress |
| Análisis de tráfico web | `Burp Suite` | Interceptación, modificación y repetición de peticiones |
| Explotación | `sqlmap`, `searchsploit`, `netcat` | Inyección SQL, búsqueda de exploits y shells inversas |
| Ataque a credenciales | `hydra`, `john` | Fuerza bruta de servicios y crackeo de hashes |
| Post-explotación | `LinPEAS`, `pspy`, `find`, `strings` | Enumeración local, procesos, binarios y secretos |
| Escalada | GTFOBins | Abuso de binarios legítimos del sistema |

&nbsp;

---

&nbsp;

## Entorno de laboratorio

| Componente | Detalle |
| :-- | :-- |
| Hipervisor | VirtualBox |
| Máquina atacante | Kali Linux o Ubuntu |
| Máquinas objetivo | Imágenes OVA de VulnHub |
| Red | Segmento local aislado (host-only o red interna) |
| Conectividad externa | Ninguna durante las pruebas |

&nbsp;

---

&nbsp;

## Estructura de los informes

Cada `README.md` de máquina mantiene el mismo esquema, lo que facilita la lectura y la comparación.

1. **Ficha técnica.** Nombre, sistema operativo, objetivo y dirección IP en el laboratorio.
2. **Reconocimiento y enumeración.** Comandos, salidas relevantes y capturas.
3. **Acceso inicial.** Vulnerabilidad identificada, prueba de concepto y obtención de shell.
4. **Escalada de privilegios.** Vector utilizado y evidencia de acceso como `root`.
5. **Evidencia final.** Obtención de `user.txt` y `root.txt`.
6. **Causa raíz y remediación.** Qué falló, por qué y cómo corregirlo.
7. **Lecciones aprendidas.** Conclusiones técnicas del ejercicio.

&nbsp;

---

&nbsp;

## Clasificación de hallazgos

Los hallazgos se valoran con CVSS v3.1 y se agrupan según el siguiente criterio.

| Severidad | Rango CVSS | Criterio |
| :-- | :-: | :-- |
| Crítica | 9.0 - 10.0 | Compromiso total del sistema sin autenticación o con mínima interacción |
| Alta | 7.0 - 8.9 | Acceso no autorizado con impacto significativo |
| Media | 4.0 - 6.9 | Exposición de información o acceso limitado |
| Baja | 0.1 - 3.9 | Debilidad de configuración con impacto reducido |
| Informativa | 0.0 | Observación sin impacto directo en seguridad |

&nbsp;

---

&nbsp;

## Estructura del repositorio

```text
laboratorioss/
├── README.md                  Portada, metodología e índice general
└── VulnHub/
    ├── C-1/
    │   └── README.md          Informe completo de C-1
    ├── C-2/
    │   └── README.md          Informe completo de C-2
    ├── Jangow/
    │   └── README.md          Informe completo de Jangow
    └── Planet/
        └── README.md          Informe completo de Planet
```

&nbsp;

---

&nbsp;

## Convenciones de documentación

- Los comandos se presentan en bloques de código, indicando si se ejecutan en la máquina atacante o en el objetivo.
- Las capturas se incluyen en línea, inmediatamente después del comando o hallazgo que documentan.
- Las direcciones IP corresponden al laboratorio local y pueden variar entre despliegues.
- Cada informe distingue entre lo observado, lo explotado y lo recomendado.
- Las rutas y nombres de ficheros se escriben con tipografía monoespaciada.

&nbsp;

---

&nbsp;

## Aviso legal

Todas las pruebas se han realizado exclusivamente sobre máquinas diseñadas para ser vulnerables, en una red de laboratorio aislada y bajo control del autor.

Este contenido tiene fines educativos y de investigación en seguridad. Aplicar estas técnicas sobre sistemas sin autorización expresa y por escrito de su propietario es ilegal. El autor no se responsabiliza del uso indebido de esta información.
