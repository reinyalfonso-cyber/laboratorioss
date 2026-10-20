# Earth Secure Messaging --- Informe técnico de auditoría

> **Estado:** reconocimiento web, acceso al panel como `terra`,
> ejecución de comandos como `apache` y análisis de `reset_root`
> documentados. La escalada final a `root` y la bandera final siguen
> pendientes de verificación.

## Ficha técnica

  Campo                              Detalle
  ---------------------------------- ----------------------------------------------
  Máquina                            Earth Secure Messaging (VulnHub)
  Objetivo                           `10.0.2.6`
  Kali                               `10.0.2.15`
  Dominios                           `earth.local`, `terratest.earth.local`
  Puertos                            22/SSH, 80/HTTP, 443/HTTPS
  Servidor web                       Apache 2.4.51 (Fedora), mod_wsgi, Python 3.9
  Usuario del panel                  `terra`
  Usuario de ejecución de comandos   `apache`
  Binario SUID investigado           `/usr/bin/reset_root`

## Resumen de la cadena

  -----------------------------------------------------------------------
  Fase                    Técnica                 Resultado
  ----------------------- ----------------------- -----------------------
  Reconocimiento          Nmap                    Identificados SSH, HTTP
                                                  y HTTPS

  Enumeración web         Gobuster, `robots.txt`  Encontrados `/admin/` y
                                                  recursos de prueba

  Análisis de pistas      `testingnotes.txt`,     Las notas mencionan XOR
                          `testdata.txt`          y el uso de
                                                  `testdata.txt` en
                                                  pruebas

  Acceso web              Panel `/admin/`         Acceso como `terra`

  Ejecución de comandos   CLI administrativa      `whoami` devuelve
                                                  `apache`

  Enumeración local       Búsqueda SUID           Encontrado
                                                  `/usr/bin/reset_root`

  Análisis                Copia a Kali, `ltrace`  Se observan tres rutas
  estático/dinámico                               comprobadas por
                                                  `access()`

  Escalada final          Verificación de         Pendiente; no se ha
                          desencadenantes         confirmado `root`
  -----------------------------------------------------------------------

## 1. Reconocimiento

### 1.1 Nmap

Comando utilizado:

``` bash
nmap -sCV 10.0.2.6
```

Servicios observados: - `22/tcp`: OpenSSH 8.6. - `80/tcp`: Apache httpd
2.4.51 (Fedora), mod_wsgi/4.7.1, Python 3.9. - `443/tcp`: Apache httpd
2.4.51 (Fedora), mod_wsgi/4.7.1, Python 3.9.

El certificado TLS incluye `earth.local` y `terratest.earth.local`.

**Captura:** `01-nmap-servicios.png`

### 1.2 Enumeración de directorios

``` bash
gobuster dir -u https://earth.local -w /usr/share/wordlists/dirb/common.txt -k
gobuster dir -u https://terratest.earth.local -w /usr/share/wordlists/dirb/common.txt -k
```

Se encontró `/admin/` en `earth.local`; también se identificaron
recursos como `robots.txt` e `index.html` en el segundo dominio.

**Captura:** `02-gobuster-enumeracion.png`

### 1.3 Archivos de pistas

Se consultaron:

``` bash
curl -k https://terratest.earth.local/robots.txt
curl -k https://terratest.earth.local/testingnotes.txt
curl -k https://terratest.earth.local/testdata.txt
```

`testingnotes.txt` menciona XOR, indica que `testdata.txt` se usó para
probar el cifrado y señala que `terra` es el usuario del panel. La
página principal contiene mensajes representados como cadenas
hexadecimales.

**Capturas:** `03-robots-txt.png`, `04-testingnotes.png`,
`05-testdata.png`, `06-mensajes-cifrados.png`.

### 1.4 Análisis XOR

Para documentar correctamente el descifrado deben conservarse la cadena
hexadecimal de entrada, la clave exacta, la codificación y el resultado
legible. Las capturas disponibles no bastan para afirmar una contraseña
concreta sin verificar el resultado. No se debe publicar una credencial
en claro en la versión pública del informe.

**Captura prevista:** `07-descifrado-xor.png` (añadir cuando el
descifrado sea reproducible).

## 2. Panel administrativo

Se accedió a `https://earth.local/admin/` como `terra`. En la
herramienta CLI se ejecutó:

``` bash
whoami
```

Resultado:

``` text
apache
```

Esto confirma que la ejecución de comandos desde el panel se realiza
como el usuario de servicio `apache`, no como `root`.

**Capturas:** `08-login-admin.png`, `09-cli-whoami-apache.png`.

## 3. Enumeración de SUID

Comando utilizado:

``` bash
find / -type f -perm -4000 -ls 2>/dev/null
```

La lista incluía `/usr/bin/sudo`, `/usr/bin/pkexec`, `/usr/bin/passwd`,
`/usr/bin/su` y `/usr/bin/reset_root`. El binario investigado fue
`/usr/bin/reset_root`, propiedad de `root` y con el bit SUID.

Al ejecutarlo, mostró:

``` text
CHECKING IF RESET TRIGGERS PRESENT...
RESET FAILED, ALL TRIGGERS ARE NOT PRESENT.
```

**Capturas:** `10-suid-enumeracion.png`, `11-reset-root-fallo.png`.

## 4. Transferencia y análisis de `reset_root`

Se abrió un listener en Kali para recibir el archivo:

``` bash
nc -lvnp 3333 > reset_root
```

La conexión observada procedía de `10.0.2.6`. Se verificó el archivo
recibido con `file`, `ls -lh` y `readelf -h`.

Después se analizó la copia con:

``` bash
ltrace ./reset_root
```

La salida mostró estas comprobaciones:

``` text
access("/dev/shm/kHgTFI5G", 0) = -1
access("/dev/shm/Zw7bV9U5", 0) = -1
access("/tmp/kcM0Wewe", 0) = -1
```

Las rutas deben respetarse exactamente, incluidos mayúsculas y
minúsculas. En una prueba anterior se había utilizado `/tmp/kcM0Wew`,
que no coincide con `/tmp/kcM0Wewe`.

**Importante:** `ltrace` se ejecutó en la copia de Kali. Para confirmar
el comportamiento en Earth hay que comprobar las rutas en el sistema
objetivo y validar el resultado allí. La salida actual no demuestra que
se haya obtenido `root`.

**Capturas:** `12-transferencia-reset-root.png`, `13-ltrace-rutas.png`,
`14-verificacion-rutas.png`.

## 5. Estado de la escalada

La escalada final no debe darse por completada hasta capturar: - `id`
mostrando `uid=0(root)`. - La lectura de la bandera final en Earth. -
Evidencia que permita distinguir la sesión del objetivo de la máquina
Kali.

**Capturas pendientes:** `15-root-confirmacion.png`, `16-root-flag.png`.

## 6. Hallazgos y recomendaciones

  -----------------------------------------------------------------------
  Hallazgo                            Recomendación
  ----------------------------------- -----------------------------------
  Panel web con capacidad de ejecutar Eliminar la ejecución arbitraria de
  comandos                            comandos y aplicar privilegios
                                      mínimos

  Recursos de pruebas y notas         Retirar archivos de desarrollo del
  accesibles públicamente             entorno publicado

  Uso de XOR para proteger mensajes   Usar criptografía autenticada
                                      moderna y gestión segura de claves

  Binario personalizado SUID          Retirar SUID si no es
                                      imprescindible y auditar su
                                      implementación

  Servicios y sistema operativo       Aplicar parches y mantener
                                      inventario de versiones
  -----------------------------------------------------------------------

## 7. Lecciones aprendidas

-   La enumeración de subdominios y recursos puede revelar pistas no
    enlazadas desde la página principal.
-   Los datos hexadecimales requieren identificar correctamente formato,
    clave y algoritmo.
-   `whoami` permite confirmar el contexto de ejecución de la CLI web.
-   Un binario SUID debe investigarse por su comportamiento real; su
    nombre no demuestra que sea vulnerable.
-   La salida de `ltrace` de una copia en Kali debe contrastarse con el
    sistema objetivo.
-   Un informe debe separar hechos confirmados, hipótesis y pasos
    pendientes.

