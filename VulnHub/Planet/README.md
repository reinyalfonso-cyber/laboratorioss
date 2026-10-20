# Earth Secure Messaging \| Informe Técnico de Auditoría (Enumeración web, cifrado XOR y escalada de privilegios)

Informe técnico de la evaluación de seguridad Boot2Root realizada sobre
la máquina **Earth Secure Messaging**, del entorno VulnHub. La
investigación incluyó el reconocimiento de servicios, la enumeración de
recursos web, el análisis de pistas relacionadas con el cifrado XOR, el
acceso al panel administrativo como `terra`, la ejecución de comandos
como `apache` y el análisis del binario SUID `/usr/bin/reset_root`.

El reconocimiento y varias fases de la investigación están documentados
mediante capturas. **La escalada final a `root` no se considera
confirmada** hasta obtener una evidencia que muestre el contexto
privilegiado en la máquina objetivo y la lectura de la bandera final.

------------------------------------------------------------------------

## Ficha técnica

  -----------------------------------------------------------------------
  Campo                               Detalle
  ----------------------------------- -----------------------------------
  Nombre de la máquina                Earth Secure Messaging

  Plataforma                          VulnHub

  Dirección IP del objetivo           `10.0.2.6`

  Dirección IP de Kali Linux          `10.0.2.15`

  Dominios identificados              `earth.local`,
                                      `terratest.earth.local`

  Sistema / servidor web identificado Fedora, Apache httpd 2.4.51,
                                      mod_wsgi/4.7.1, Python 3.9

  Puertos expuestos                   `22/tcp` (SSH), `80/tcp` (HTTP),
                                      `443/tcp` (HTTPS)

  Usuario del panel administrativo    `terra`

  Usuario efectivo de la CLI web      `apache`

  Binario SUID investigado            `/usr/bin/reset_root`

  Técnicas principales                Nmap, Gobuster, análisis de XOR,
                                      enumeración SUID y `ltrace`

  Objetivo del ejercicio              Investigar el acceso inicial y
                                      verificar si es posible escalar
                                      privilegios hasta `root`
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## Resumen de la cadena de investigación

  -----------------------------------------------------------------------
  Fase                    Técnica                 Resultado
  ----------------------- ----------------------- -----------------------
  Reconocimiento          Escaneo de servicios    Identificación de SSH,
                          con `nmap`              HTTP y HTTPS en el
                                                  objetivo

  Enumeración web         `gobuster` y revisión   Identificación del
                          de `robots.txt`         panel `/admin/` y
                                                  recursos adicionales

  Análisis de pistas      Revisión de             Las notas mencionan el
                          `testingnotes.txt` y    uso de XOR y de
                          `testdata.txt`          `testdata.txt` durante
                                                  las pruebas

  Análisis de mensajes    Inspección del HTML y   Se identificaron
                          de cadenas              mensajes cifrados; el
                          hexadecimales           resultado del
                                                  descifrado debe
                                                  verificarse

  Acceso administrativo   Inicio de sesión en     Acceso al panel como
                          `/admin/`               `terra`

  Ejecución de comandos   CLI del panel y comando Confirmación del
                          `whoami`                usuario efectivo
                                                  `apache`

  Enumeración local       Búsqueda de binarios    Identificación de
                          SUID                    `/usr/bin/reset_root`

  Análisis del binario    Transferencia a Kali y  Identificación de tres
                          ejecución de `ltrace`   rutas comprobadas
                                                  mediante `access()`

  Escalada de privilegios Verificación de         Pendiente de
                          desencadenantes y nueva confirmación; el
                          ejecución               binario había indicado
                                                  que faltaban
                                                  desencadenantes
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 1. Reconocimiento y enumeración de servicios

### 1.1. Escaneo de puertos

Se ejecutó un escaneo de detección de servicios contra el objetivo:

``` bash
nmap -sCV 10.0.2.6
```

El resultado identificó los siguientes servicios:

  -----------------------------------------------------------------------
  Puerto                  Estado                  Servicio / información
  ----------------------- ----------------------- -----------------------
  `22/tcp`                Abierto                 OpenSSH 8.6

  `80/tcp`                Abierto                 Apache httpd 2.4.51
                                                  (Fedora),
                                                  mod_wsgi/4.7.1, Python
                                                  3.9

  `443/tcp`               Abierto                 HTTPS sobre Apache
                                                  httpd 2.4.51 (Fedora),
                                                  mod_wsgi/4.7.1, Python
                                                  3.9
  -----------------------------------------------------------------------

El certificado TLS presentó los nombres DNS `earth.local` y
`terratest.earth.local`.

![Escaneo de servicios con Nmap](./evidencias/01-nmap-servicios.png)

### 1.2. Enumeración de rutas web

Se enumeraron directorios y archivos en ambos dominios:

``` bash
gobuster dir -u https://earth.local -w /usr/share/wordlists/dirb/common.txt -k
gobuster dir -u https://terratest.earth.local -w /usr/share/wordlists/dirb/common.txt -k
```

En `earth.local` se identificó `/admin/`, que redirige a la ruta del
panel administrativo, y `cgi-bin/`, que devolvió un código `403`. En
`terratest.earth.local` se identificaron recursos como `robots.txt` e
`index.html`, además de rutas restringidas.

![Enumeración de rutas con
Gobuster](./evidencias/02-gobuster-enumeracion.png)

### 1.3. Revisión de `robots.txt` y archivos de prueba

Se consultaron los recursos encontrados:

``` bash
curl -k https://terratest.earth.local/robots.txt
curl -k https://terratest.earth.local/testingnotes.txt
curl -k https://terratest.earth.local/testdata.txt
```

Las notas de pruebas indican que el sistema utiliza XOR y que
`testdata.txt` se empleó para probar el cifrado. También identifican
`terra` como el nombre de usuario del panel administrativo. La página
principal contiene mensajes representados mediante cadenas
hexadecimales.

![Contenido de robots.txt](./evidencias/03-robots-txt.png)

![Notas de pruebas de la aplicación](./evidencias/04-testingnotes.png)

![Contenido de testdata.txt](./evidencias/05-testdata.png)

### 1.4. Análisis de los mensajes cifrados

Se guardó el HTML de la página principal para examinar las cadenas
hexadecimales:

``` bash
curl -k https://earth.local/ -o pagina-principal.html
```

Las notas apuntan a XOR como algoritmo utilizado. Para considerar el
descifrado demostrado, es necesario registrar la cadena de entrada, la
clave exacta, la representación de los bytes y una salida legible
reproducible. Las evidencias disponibles muestran las pistas y los
mensajes cifrados, pero no bastan por sí solas para confirmar una
contraseña concreta.

![Mensajes cifrados de la página
principal](./evidencias/06-mensajes-cifrados.png)

![Análisis XOR](./evidencias/07-analisis-xor.png)

------------------------------------------------------------------------

## 2. Acceso al panel administrativo

### 2.1. Inicio de sesión como `terra`

Se accedió a:

``` text
https://earth.local/admin/
```

El panel requiere autenticación. El usuario utilizado fue `terra`, de
acuerdo con la pista encontrada en los recursos de prueba. No se
incluyen contraseñas en claro en este informe.

![Inicio de sesión del panel
administrativo](./evidencias/08-login-admin.png)

### 2.2. Confirmación del usuario efectivo

Desde la herramienta **Admin Command Tool** se ejecutó:

``` bash
whoami
```

La salida fue:

``` text
apache
```

Esto confirma que los comandos enviados desde la CLI web se ejecutan
como el usuario de servicio `apache`, no como `root`.

![Confirmación de ejecución como
apache](./evidencias/09-cli-whoami-apache.png)

------------------------------------------------------------------------

## 3. Enumeración local y análisis de SUID

### 3.1. Enumeración de binarios SUID

Se ejecutó la búsqueda de archivos con el bit SUID:

``` bash
find / -type f -perm -4000 -ls 2>/dev/null
```

Entre los resultados aparecieron binarios del sistema como `sudo`,
`pkexec`, `passwd`, `su` y el ejecutable personalizado
`/usr/bin/reset_root`.

El binario `reset_root` llamó la atención porque pertenece a `root` y
tiene habilitado el bit SUID. Esto justifica analizar su comportamiento,
pero no demuestra por sí solo que sea explotable.

![Enumeración de binarios SUID](./evidencias/10-suid-enumeracion.png)

### 3.2. Ejecución inicial de `reset_root`

Se ejecutó:

``` bash
/usr/bin/reset_root
```

El programa mostró:

``` text
CHECKING IF RESET TRIGGERS PRESENT...
RESET FAILED, ALL TRIGGERS ARE NOT PRESENT.
```

Por tanto, la ejecución inicial no restableció la contraseña de `root`.

![Ejecución inicial fallida de
reset_root](./evidencias/11-reset-root-fallo.png)

------------------------------------------------------------------------

## 4. Transferencia y análisis de `reset_root`

### 4.1. Recepción del binario en Kali

Se preparó un listener en Kali para recibir el archivo:

``` bash
nc -lvnp 3333 > reset_root
```

La salida del listener confirmó una conexión procedente de `10.0.2.6`.
Tras la transferencia, el archivo debe validarse en Kali:

``` bash
file reset_root
ls -lh reset_root
readelf -h reset_root
```

La transferencia solo se considera verificada si el archivo tiene tamaño
no nulo y las herramientas reconocen su formato ELF.

![Transferencia de reset_root a
Kali](./evidencias/12-transferencia-reset-root.png)

### 4.2. Análisis con `ltrace`

Se ejecutó el análisis de llamadas a bibliotecas:

``` bash
ltrace ./reset_root
```

La salida observada fue:

``` text
access("/dev/shm/kHgTFI5G", 0) = -1
access("/dev/shm/Zw7bV9U5", 0) = -1
access("/tmp/kcM0Wewe", 0) = -1
```

El resultado muestra que la copia analizada comprobó tres rutas y que
las tres llamadas a `access()` fallaron durante esa ejecución.

![Rutas comprobadas por ltrace](./evidencias/13-ltrace-rutas.png)

### 4.3. Interpretación de las rutas

Las rutas que aparecen en la salida deben copiarse literalmente:

  N.º   Ruta observada
  ----- ---------------------
  1     `/dev/shm/kHgTFI5G`
  2     `/dev/shm/Zw7bV9U5`
  3     `/tmp/kcM0Wewe`

En una comprobación anterior se había utilizado `/tmp/kcM0Wew`, que no
coincide con la ruta completa mostrada por `ltrace`. Aun así, crear
archivos con esos nombres no basta para afirmar que el programa
funcionará: también debe comprobarse que el binario de Earth usa las
mismas rutas y que el proceso puede ver los archivos.

**Nota:** `ltrace` se ejecutó sobre una copia en Kali. La salida
describe esa ejecución; debe contrastarse con el comportamiento del
binario original en Earth.

![Verificación de las rutas en
Earth](./evidencias/14-verificacion-rutas.png)

------------------------------------------------------------------------

## 5. Estado de la escalada de privilegios

La escalada final a `root` todavía no está confirmada por las evidencias
disponibles. Para cerrar esta fase, el informe debe incluir una
comprobación ejecutada en el objetivo que muestre el contexto
privilegiado, por ejemplo, la salida de `id` con `uid=0(root)`, y la
evidencia de lectura de la bandera final.

Hasta que esas pruebas existan, el resultado debe registrarse como
**pendiente de verificación**, no como compromiso total confirmado.

![Confirmación de root](./evidencias/15-root-confirmacion.png)

![Bandera final](./evidencias/16-root-flag.png)

------------------------------------------------------------------------

## 6. Resumen de resultados

  ------------------------------------------------------------------------------
  Elemento                Ubicación / evidencia          Estado
  ----------------------- ------------------------------ -----------------------
  Servicios SSH, HTTP y   Escaneo Nmap                   Confirmado
  HTTPS                                                  

  Panel administrativo    `https://earth.local/admin/`   Confirmado

  Pistas de XOR           `robots.txt`,                  Confirmadas
                          `testingnotes.txt`,            
                          `testdata.txt`                 

  Acceso como `terra`     Panel administrativo           Confirmado

  Ejecución como `apache` `whoami` en la CLI web         Confirmado

  Binario SUID            `/usr/bin/reset_root`          Confirmado
  investigado                                            

  Rutas comprobadas por   Copia de `reset_root`          Confirmadas para esa
  `ltrace`                analizada en Kali              copia

  Escalada a `root`       Evidencia final pendiente      No confirmada

  Bandera final           Evidencia pendiente            No confirmada
  ------------------------------------------------------------------------------

------------------------------------------------------------------------

## 7. Causa raíz y recomendaciones

  -----------------------------------------------------------------------
  Hallazgo                Riesgo / causa          Recomendación
  ----------------------- ----------------------- -----------------------
  Panel web con capacidad Una interfaz web que    Eliminar la ejecución
  de ejecutar comandos    permite ejecutar        arbitraria de comandos;
                          comandos puede ampliar  si una función
                          el impacto de fallos de administrativa la
                          autenticación o         requiere, usar una
                          autorización            lista permitida y
                                                  privilegios mínimos

  Recursos de pruebas     Las notas y los datos   Retirar archivos de
  expuestos               de prueba revelan       desarrollo y pruebas de
                          detalles de             los entornos publicados
                          implementación y pueden 
                          facilitar el análisis   
                          de la aplicación        

  Uso de XOR para         XOR sin un diseño       Utilizar criptografía
  proteger mensajes       criptográfico robusto   autenticada moderna y
                          no proporciona          una gestión segura de
                          protección adecuada por claves
                          sí solo                 

  Binario personalizado   Un fallo en un programa Eliminar SUID si no es
  con SUID                privilegiado puede      imprescindible, auditar
                          permitir escalada de    el código y reducir las
                          privilegios             operaciones
                                                  privilegiadas

  Servicios que requieren Versiones antiguas      Mantener el sistema y
  mantenimiento           pueden contener         los servicios
                          vulnerabilidades        actualizados y revisar
                          conocidas               periódicamente su
                                                  configuración
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 8. Lecciones aprendidas

-   La enumeración de dominios, rutas y archivos auxiliares puede
    revelar información que no aparece en la página principal.
-   Los mensajes hexadecimales requieren identificar correctamente el
    formato, la clave y el funcionamiento real del algoritmo antes de
    dar por válido un descifrado.
-   `whoami` permite confirmar el usuario efectivo con el que se
    ejecutan los comandos del panel.
-   La presencia de un binario SUID requiere investigación, pero no
    demuestra por sí sola que exista una vulnerabilidad explotable.
-   Los resultados de `ltrace` obtenidos en Kali deben contrastarse con
    el comportamiento del binario en el objetivo.
-   Un informe de auditoría debe distinguir entre hallazgos confirmados,
    hipótesis y fases pendientes de validación.
