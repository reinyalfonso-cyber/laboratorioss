# Informes Técnicos de Pruebas de Penetración — VulnHub

Repositorio de auditorías de seguridad ofensiva realizadas sobre entornos controlados de VulnHub. Cada informe documenta el ciclo completo de evaluación técnica, desde la fase de reconocimiento hasta la obtención de privilegios de superusuario (`root`) y las recomendaciones de remediación asociadas.

## Resumen Ejecutivo

El propósito de este repositorio es documentar de forma estructurada y verificable la identificación, explotación y mitigación de vulnerabilidades en sistemas Linux. Cada laboratorio se aborda bajo un enfoque de auditoría técnica reproducible:

- **Trazabilidad técnica:** Registro secuencial de comandos, parámetros y resultados obtenidos en cada fase de la prueba.
- **Evidencia integrada:** Inclusión de capturas de pantalla directamente en el flujo de ejecución para respaldar cada hallazgo.
- **Análisis de causa raíz y remediación:** Identificación del fallo técnico subyacente acompañada de medidas correctivas orientadas a la mitigación del riesgo.

## Catálogo de Informes

| ID | Activo Evaluado | Sistema Operativo | Enfoque | Estado | Documentación |
| :---: | :--- | :---: | :--- | :---: | :--- |
| 01 | **C-1** | Linux | Compromiso inicial y escalada de privilegios | Completado | [Ver informe](./VulnHub/C-1/) |
| 02 | **C-2** | Linux | Compromiso inicial y escalada de privilegios | Completado | [Ver informe](./VulnHub/C-2/) |
| 03 | **Jangow** | Linux | Compromiso inicial y escalada de privilegios | Completado | [Ver informe](./VulnHub/Jangow/) |
| 04 | **Planet** | Linux | Compromiso inicial y escalada de privilegios | Completado | [Ver informe](./VulnHub/Planet/) |

## Metodología de Evaluación

Las pruebas siguen un flujo de trabajo estructurado en cinco fases basadas en estándares de la industria (PTES y NIST SP 800-115):

1. **Reconocimiento y Descubrimiento:** Identificación de activos en el segmento de red local, escaneo de puertos activos y detección de versiones de servicios expuestos (`arp-scan`, `nmap`).
2. **Enumeración y Análisis de Superficie:** Descubrimiento de rutas web, análisis de cabeceras HTTP, inspección de componentes vulnerables y evaluación de servicios de red (`gobuster`, `ffuf`, `whatweb`, `nikto`).
3. **Explotación y Acceso Inicial:** Validación de vulnerabilidades, manipulación de peticiones mediante `Burp Suite` o `curl`, ejecución de pruebas de concepto y obtención de acceso interactivo al sistema.
4. **Post-Explotación y Escalada de Privilegios:** Auditoría interna de permisos (`sudo`), binarios con bit SUID/SGID, capacidades del sistema, tareas programadas (`cron`) y configuraciones inseguras para alcanzar privilegios de administración (`root`).
5. **Remediación y Endurecimiento:** Definición de controles técnicos, aplicación del principio de mínimo privilegio y corrección de las configuraciones vulnerables identificadas.

## Estructura del Repositorio

```text
laboratorioss/
├── README.md                  Documento principal y catálogo de informes
└── VulnHub/
    ├── C-1/
    │   ├── img/               Evidencias gráficas de la auditoría C-1
    │   └── README.md          Informe técnico del activo C-1
    ├── C-2/
    │   ├── img/               Evidencias gráficas de la auditoría C-2
    │   └── README.md          Informe técnico del activo C-2
    ├── Jangow/
    │   ├── img/               Evidencias gráficas de la auditoría Jangow
    │   └── README.md          Informe técnico del activo Jangow
    └── Planet/
        ├── img/               Evidencias gráficas de la auditoría Planet
        └── README.md          Informe técnico del activo Planet
