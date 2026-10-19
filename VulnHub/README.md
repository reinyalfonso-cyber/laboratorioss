# VulnHub

Índice de los informes Boot2Root documentados en este repositorio. Cada carpeta contiene el write-up completo de una máquina: reconocimiento, explotación, escalada de privilegios y remediación, con capturas integradas paso a paso.

Para el contexto general del proyecto —metodología, stack de herramientas, clasificación de hallazgos y convenciones de documentación— consulta el [README principal](../README.md).

&nbsp;

---

&nbsp;

## Máquinas documentadas

| N.º | Máquina | Sistema | Objetivo | Estado | Informe |
| :-: | :-- | :-: | :-- | :-: | :-- |
| 01 | C-1 | Linux | Boot2Root (user y root) | Completada | [Ver informe](./C-1/) |
| 02 | C-2 | Linux | Boot2Root (user y root) | Completada | [Ver informe](./C-2/) |
| 03 | Jangow | Linux | Boot2Root (user y root) | Completada | [Ver informe](./Jangow/) |
| 04 | Planet | Linux | Boot2Root (user y root) | Completada | [Ver informe](./Planet/) |

&nbsp;

---

&nbsp;

## Qué incluye cada informe

Todos los write-ups de esta carpeta siguen el mismo esquema para facilitar la lectura y la comparación entre máquinas:

- Ficha técnica de la máquina y resumen de la cadena de ataque.
- Reconocimiento y enumeración de servicios.
- Explotación y obtención de acceso inicial.
- Post-explotación y escalada de privilegios.
- Evidencia final de las banderas obtenidas.
- Análisis de causa raíz y recomendaciones de remediación.
