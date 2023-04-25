Bienvenidos a otra sala de TryHackMe.. En este capitulo explotaremos una pagina web con servicio PHP,  y utilizaremos el comando "__find__" para leer nuestra flag sin necesidad de escalar privilegios.

# Recon
Para inciar nuestro reconocimiento, utilizaremos __nmap__ para escanear puertos abiertos y averiguar versiones del sistema. Ejecutaremos el siguiente comando : `nmap -sS -p- --open --min-rate=5000 -Pn -sV -n -vvv -oN nmap IPTARGET`

```ruby
Nmap scan report for TARGETIP
Host is up, received user-set (0.22s latency).
Scanned at 2023-03-30 18:58:52 EDT for 26s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 62 PHP cli server 5.5 or later (PHP 8.1.0-dev)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

# Research

Nuestro scan de nmap, nos muestra el puerto abierto 80 (http). Al navegar por internet, nos muestra un dashboard de administrador, siempre es positivo realizar una busqueda en toda la pagina web antes de proceder, cualquier tipo de enumeracion es bien recibida a la hora de buscar posibles vulnerabilidades. En este caso, utilizaremos nuestro scaneo de nmap para poder explotarlo de forma correcta.
 Nmap nos muestra la version del servicio, __PHP 8.1.0-dev__. Buscaremos un exploit que se adapte a la version, podremos utilizar "google" o en mi caso, utilizar __searchsploit__, un motor de busqueda de vulnerabildades que podremos obtener informacion y descargarlo.. otra buena opcion es utilizar __exploit-db__. Para buscar dentro de searchsploit, ejecutaremos el siguiente comando : `searchsploit PHP 8.1.0-dev` 

```ruby
Micro Focus Secure Messaging Gateway (SMG) < 471 - Remote Code Execution (Metasploit)                                                                                           | php/webapps/45083.rb
NPDS < 08.06 - Multiple Input Validation Vulnerabilities                                                                                                                        | php/webapps/32689.txt
OPNsense < 19.1.1 - Cross-Site Scripting                                                                                                                                        | php/webapps/46351.txt
PHP 8.1.0-dev - 'User-Agentt' Remote Code Execution                                                                                                                             | php/webapps/49933.py  "ESTE EXPLOIT ES EL QUE TENDREMOS QUE UTILIZAR"
Plesk < 9.5.4 - Remote Command Execution                                                                                                                                        | php/remote/25986.txt
REDCap < 9.1.2 - Cross-Site Scripting                                                                                                                                           | php/webapps/47146.txt
Responsive FileManager < 9.13.4 - Directory Traversal                                                                                                                           | php/webapps/45271.txt
Responsive Filemanger <= 9.11.0 - Arbitrary File Disclosure                                                                                                                     | php/webapps/41272.txt
ShoreTel Connect ONSITE < 19.49.1500.0 - Multiple Vulnerabilities                                                                                                               | php/webapps/46666.txt
Western Digital Arkeia < 10.0.10 - Remote Code Execution (Metasploit)                                                                                                           | php/remote/28407.rb
WordPress Plugin DZS Videogallery < 8.60 - Multiple Vulnerabilities                                                                                                             | php/webapps/39553.txt
Zoho ManageEngine ADSelfService Plus 5.7 < 5702 build - Cross-Site Scripting                                                                                                    | php/webapps/46815.txt
```

## 
Tendremos un exploit #RCE(__Remote Code Execution__). Esta vulnerabilidad ocurre cuando un atacante puede ejecutar codigo en un sistema remoto de forma no autorizada.

Para descargar este exploit, deberemos ejecutar : `searchsploit -m php/webapps/49933.py`
Obtendremos nuestro exploit, y podremos ver que es un script en lenguaje python que al ejecutarlo nos pedira la direccion URL.

# Exploit

Para ejecutar el script de lenguaje python, tendremos que solicitarle a python que ejecute el script utilizando : `python3 49933.py`. Nos pedira que introduzcamos el sitio a explotar
http://IPTARGET, y obtendremos acceso al servicio asi de simple. Una vez dentro, nos daremos cuenta que no podremos movernos de directorio, pero podremos leer archivos y enumerar directorios utilizando `cat, ls` . 

Para reclamar nuestra flag, primero tendremos que bucarla dentro del sistema. Para esta accion ejecutaremos el comando __find__ y le pediremos que busque el archivo en todo el sistema con el nombre "flag.txt" `find / -type f -name flag.txt`

Nuestra flag se encuentra en el direcorio del sistema,  le echaremos un vistazo con __cat__ para leer la flag y completar la sala. `cat /flag.txt`

# Respuestas

#FLAG = flag{4127d0530abf16d6d23973e3df8dbecb}