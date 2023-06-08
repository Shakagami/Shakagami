Bienvenidos nuevamente a otra sala de tryhackme, en esta ocasion escanearemos puertos de nuestra maquina, y nos abusaremos utilizando #xss para obtener nuestra flag.

# Recon
Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`
```ruby
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-26 18:34 EDT
NSE: Loaded 45 scripts for scanning.
Initiating SYN Stealth Scan at 18:34
Scanning 10.10.159.162 [65535 ports]
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp   open  rtsp    syn-ack ttl 62
5000/tcp open  rtsp    syn-ack ttl 62

```

Como podremos observar, tendremos 3 puertos abiertos y nos centraremos en el puerto 5000 que comparte el mismo servicio 80(http), en este caso esta como "rtsp". Al dirigirnos a la pagina web con la direccion IP, nos mostrara un convertidor pdf. Al dirigirnos a IPTARGET:5000 obtendremos la misma respuesta pero con diferente visualizacion.
Antes que nada, primero enumeraremos si hay algun directorio oculto con nuestra herramienta gobuster.

# Enumeration

Utilizaremos nuestra herramienta **gobuster** para listar posibles directorios ocultos dentro de la maquina victima, ejectuaremos el siguiente comando para realizar nuestra busqueda.
`gobuster dir -u http://IPTARGET -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100`

```ruby
[+] Url:                     http://IPTARGET
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/05/26 18:35:08 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 403) [Size: 166]
Progress: 3374 / 220561 (1.53%)^C
[!] Keyboard interrupt detected, terminating.

```

Tendremos nuestro directorio **/admin** pero con el status 403, que significa **Forbidden**.
Al navegar dentro de este directorio, nos dira como mensaje, ademas de **Forbidden**, que solamente se puede ver internamente en http://localhost:5000, que al intentar navegar alli, no tenremos conexion ni podremos acceder internamente para ver nuestra flag.

# Exploit

Para aprovecharnos de esto, navegaremos donde anteriormente habiamos visto el convertidor a archivos PDF. Una vez alli, haremos usi de XSS para ver si podremos abusarnos de esta vulnerabilidad.

Al escribir algo simple como `<script>alert(hello)</script>` y selccionamos convertir, no nos dara respuesta alguna. Por lo que tendremos que utilizar otro metodo, para acceder a el contenido.  Utilizaremos una etiqueta de iframe en HTML con un atributo src que tiene un valor de http://localhost:5000. Nos quedaria de la siguiente manera:

`<iframe src=http://localhost:5000></iframe>`

Al convertirlo, podremos ver una respuesta positiva pero sin el contenido oculto, anteriormente realizamos una enumeracion a directorios ocultos, por lo que tendremos que modificar un poco mas esta etiqueta para que nos muestre nuestra flag, y como resultado final nos quedaria de la siguiente forma:

`iframe src=http://localhost:5000/admin></iframe>`

De este modo, nos dara la misma respuesta pero con la flag incluida.