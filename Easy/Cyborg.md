Bienvenidos a otra sala en TryHackMe. En esta maquina, enumeraremos directorios utilizando **gobuster** , crackearemos una contraseña con la herramienta **hashcat** para luego utilizar #borg que es un software con la intencion de realizar copias de seguridad. Nos aprovecharemos de esto, y haremos una copia en nuestra maquina atacante.  Para finalizar, escalaremos privilegios utilizando el argumento `-c` 

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`
```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-04-28 04:49:25 EDT for 24s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
```

Como podemos ver en nuestro escaneo, nos muestra los puertos ssh y http abiertos. Servicio con version Apache, un ttl de 63 que significa que estamos atacando una maquina Linux, con distribucion Ubuntu. Siempre es recomendable navegar al sitio web e inspeccionar el Page-Source para encontrar alguna enumeracion que nos beneficie, en este caso como no encontramos nada enumeraremos con ayuda de **gobuster**

#  Enumeration

Utiizaremos **gobuster** ,es una  herramienta para enumerar directorios ocultos dentro de una pagina web, es una herramienta muy util porque nos permite encontrar posibles vulnerabilidades en archivos o directorios que no estan a simple vista. Ejecutamos gobuster : `gobuster dir -u http://IPTARGET -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
```ruby
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://IPTARGET
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/04/28 04:54:28 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 312] [--> http://10.10.66.182/admin/]
/etc                  (Status: 301) [Size: 310] [--> http://10.10.66.182/etc/]
```

Encontramos los directorios **/admin & /etc** . Al navegar en el primer directorio, nos encontramos un blog personal de **alex**. Descubriremos que es un profesor de musica, pero lo importante es que tendremos un nombre de usuario. Al utilizar **hydra** no obtuvimos exito porque es una password poco comun. Luego, al navegar al segundo directorio, podremos una carpeta llamada **passwd**, en la que se almacena un hash con un nombre de usuario. 

# 