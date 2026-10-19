# VulnHub Boot2Root: Write-ups de Pentesting

Documentación técnica de auditorías ofensivas sobre máquinas vulnerables de VulnHub, desde el reconocimiento inicial hasta la escalada de privilegios y la remediación.

Cada informe es reproducible de principio a fin, incluye la evidencia gráfica junto a cada comando y analiza la causa raíz de cada fallo, no solo la forma de explotarlo.

&nbsp;

---

&nbsp;

## Índice

1. [Resumen](#resumen)
2. [Aviso legal y ético](#aviso-legal-y-ético)
3. [Catálogo de máquinas](#catálogo-de-máquinas)
4. [Metodología](#metodología)
5. [Estructura de cada informe](#estructura-de-cada-informe)
6. [Herramientas](#herramientas)
7. [Entorno de laboratorio](#entorno-de-laboratorio)
8. [Estructura del repositorio](#estructura-del-repositorio)
9. [Convenciones](#convenciones)

&nbsp;

---

&nbsp;

## Resumen

Este repositorio reúne las resoluciones completas de máquinas Boot2Root desplegadas en un laboratorio local aislado. El objetivo no es únicamente capturar las flags, sino demostrar un proceso de auditoría riguroso y trazable:

- **Trazabilidad completa.** Cada fase de la intrusión queda documentada sin pasos omitidos, de modo que cualquier lector pueda replicar el resultado.
- **Evidencia integrada.** Las capturas de pantalla acompañan a cada comando y a cada hallazgo relevante, en el mismo punto del documento donde se explica.
- **Perspectiva ofensiva y defensiva.** Cada informe cierra con el análisis de la causa raíz y las medidas concretas de mitigación que habrían evitado el compromiso.

&nbsp;

---

&nbsp;

## Aviso legal y ético

Todas las pruebas se han realizado exclusivamente sobre máquinas virtuales diseñadas para ser vulnerables, dentro de una red de laboratorio aislada y bajo control del autor.

El contenido de este repositorio tiene fines educativos y de investigación en seguridad. Aplicar estas técnicas sobre sistemas sin autorización expresa y por escrito de su propietario es ilegal. El autor no se responsabiliza del uso indebido de esta información.

&nbsp;

---

&nbsp;

## Catálogo de máquinas

| N.º | Máquina | Sistema | Objetivo | Estado | Informe |
| :-: | :------ | :-----: | :------- | :----: | :------ |
| 01 | C-1 | Linux | Boot2Root (user y root) | Completada | [Ver informe](./VulnHub/C-1/) |
| 02 | C-2 | Linux | Boot2Root (user y root) | Completada | [Ver informe](./VulnHub/C-2/) |
| 03 | Jangow | Linux | Boot2Root (user y root) | Completada | [Ver informe](./VulnHub/Jangow/) |
| 04 | Planet | Linux | Boot2Root (user y root) | Completada | [Ver informe](./VulnHub/Planet/) |

&nbsp;

---

&nbsp;

## Metodología

Todas las máquinas siguen el mismo flujo de cinco fases, lo que permite comparar resultados y mantener un estándar de calidad homogéneo.

| Fase | Descripción | Herramientas habituales |
| :--- | :---------- | :---------------------- |
| **1. Reconocimiento** | Descubrimiento de hosts activos, puertos abiertos, versiones de servicios y sistema operativo. Delimitación de la superficie de ataque. | `arp-scan`, `netdiscover`, `nmap` |
| **2. Enumeración** | Descubrimiento de rutas y archivos ocultos, inspección de cabeceras y código fuente, identificación de CMS y plugins, revisión de FTP, SSH, SMB y bases de datos expuestas. | `gobuster`, `dirb`, `ffuf`, `wfuzz`, `whatweb`, `nikto` |
| **3. Explotación** | Validación de la vulnerabilidad (RCE, inyección SQL, LFI/RFI, subida de archivos, credenciales débiles), obtención de reverse shell y estabilización de la TTY. | `Burp Suite`, `curl`, `sqlmap`, `netcat`, `hydra` |
| **4. Escalada de privilegios** | Enumeración interna: `sudo -l`, SUID/SGID, capabilities, tareas `cron`, variable `PATH`, credenciales en ficheros de configuración, vulnerabilidades de kernel y abuso de binarios. | `LinPEAS`, `pspy`, `GTFOBins`, `searchsploit` |
| **5. Remediación** | Análisis de la causa raíz del acceso inicial y de la escalada, con recomendaciones de configuración segura, parcheo y mínimo privilegio. | Documentación y buenas prácticas |

&nbsp;

---

&nbsp;

## Estructura de cada informe

Cada `README.md` de máquina mantiene el mismo esquema:

1. **Ficha técnica:** nombre, sistema operativo, objetivo y dirección IP en el laboratorio.
2. **Reconocimiento y enumeración:** comandos, salidas relevantes y capturas.
3. **Acceso inicial:** vulnerabilidad identificada, prueba de concepto y obtención de shell.
4. **Escalada de privilegios:** vector utilizado y evidencia de acceso como `root`.
5. **Flags:** evidencia de `user.txt` y `root.txt`.
6. **Análisis de causa raíz y remediación:** qué falló, por qué y cómo corregirlo.
7. **Lecciones aprendidas:** conclusiones técnicas del laboratorio.

&nbsp;

---

&nbsp;

## Herramientas

| Categoría | Herramientas |
| :-------- | :----------- |
| Descubrimiento y escaneo | `Nmap`, `arp-scan`, `netdiscover`, `whatweb` |
| Enumeración web | `Gobuster`, `Dirb`, `FFUF`, `WPScan`, `Nikto`, `curl` |
| Análisis y explotación | `Burp Suite`, `SQLMap`, `Searchsploit`, `Netcat`, `Hydra`, `John the Ripper` |
| Post-explotación en Linux | `LinPEAS`, `GTFOBins`, `pspy`, `find`, `strings`, `Python 3`, `Bash` |

&nbsp;

---

&nbsp;

## Entorno de laboratorio

| Componente | Detalle |
| :--------- | :------ |
| Virtualización | VirtualBox |
| Máquina atacante | Kali Linux / Ubuntu |
| Máquinas objetivo | Imágenes de VulnHub |
| Red | Segmento local aislado, sin salida a Internet ni a redes de producción |

&nbsp;

---

&nbsp;

## Estructura del repositorio

```text
laboratorioss/
├── README.md                 Portada, metodología e índice general
└── VulnHub/
    ├── C-1/
    │   └── README.md         Informe completo de C-1
    ├── C-2/
    │   └── README.md         Informe completo de C-2
    ├── Jangow/
    │   └── README.md         Informe completo de Jangow
    └── Planet/
        └── README.md         Informe completo de Planet
```

&nbsp;

---

&nbsp;

## Convenciones

- Los comandos se muestran en bloques de código con el prompt original para distinguir el contexto (atacante u objetivo).
- Las capturas se incluyen en línea, inmediatamente después del comando o hallazgo que documentan.
- Las direcciones IP corresponden al laboratorio local y pueden variar entre despliegues.
- Cada informe distingue entre lo que se **observa**, lo que se **explota** y lo que se **recomienda corregir**.

&nbsp;

---

&nbsp;

<sub>Proyecto con fines educativos. Uso exclusivo en entornos propios o con autorización expresa.</sub>
